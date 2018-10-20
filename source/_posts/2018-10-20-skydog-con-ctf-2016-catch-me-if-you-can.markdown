---
layout: post
title: "SkyDog Con CTF 2016 - Catch Me If You Can"
date: 2018-10-20 11:53:46 -0400
comments: true
categories: [penetration testing, writeups]
keywords: skydog 2016, pentest lab, vulnhub, ctf, boot2root, skydog con 2016, skydog vulnhub, volatility
description: Writeup for SkyDog Con 2016
---

For this box, you have to find 8 flags, each containing an MD5 hash.

<!-- more -->

``` 
nmap -T4 -sC -sV -p- 192.168.145.136

PORT      STATE  SERVICE  VERSION
22/tcp    closed ssh
80/tcp    open   http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: SkyDog Con CTF 2016 - Catch Me If You Can
443/tcp   open   ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: SkyDog Con CTF 2016 - Catch Me If You Can
| ssl-cert: Subject: commonName=Network Solutions EV Server CA 2/organizationName=Network Solutions L.L.C./stateOrProvinceName=VA/countryName=US
| Not valid before: 2016-09-21T14:51:57
|_Not valid after:  2017-09-21T14:51:57
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
22222/tcp open   ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:64:7c:d1:55:46:4e:50:e3:ba:cf:4c:1e:81:f9:db (RSA)
|   256 ef:17:df:cc:db:2e:c5:24:e3:9e:25:16:3d:25:68:35 (ECDSA)
|_  256 0e:1b:3f:c3:4a:56:a0:ef:4d:2a:af:a1:7e:94:d2:06 (ED25519)
```

## Flag #1 Don’t go Home Frank! There’s a Hex on Your House.

Starting with the web server, the web page is the homepage of the SkyDog con. Poking through the source, I found a suspicious comment:

``` 
<!--[If IE4]><script src="/oldIE/html5.js"></script><![Make sure to remove this before going to PROD]--> 
```

Going to the above mentioned JS file, on the first line we find a hex string:

``` 
/* 666c61677b37633031333230373061306566373164353432363633653964633166356465657d */
```

Decoded, it gives the first flag: <code>flag{7c0132070a0ef71d542663e9dc1f5dee}</code>. And decoding the MD5 hash gives a hint: <code>nmap</code>

## Flag #2 Obscurity or Security?

I already ran Nmap and saw that SSH is listening on port 22222. Trying to SSH as frank gives the next flag:

``` 
ssh frank@192.168.145.136 -p 22222
The authenticity of host '[192.168.145.136]:22222 ([192.168.145.136]:22222)' can't be established.
ECDSA key fingerprint is SHA256:DeCMZ74o5wesBHFLyaVY7UTCA7mW+bx6WroHm6AgMqU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.145.136]:22222' (ECDSA) to the list of known hosts.
###############################################################
#                         WARNING                             #
#		FBI - Authorized access only!                 # 
# Disconnect IMMEDIATELY if you are not an authorized user!!! #
#         All actions Will be monitored and recorded          #
#	Flag{53c82eba31f6d416f331de9162ebe997}		      #
###############################################################
frank@192.168.145.136's password: 
``` 

The decoded hint is <code>encrypt</code>

## Flag #3 Be Careful Agent, Frank Has Been Known to Intercept Traffic Our Traffic.

Remember that Nmap found also port 443 open, so I switched to https to check it out and was greeted by a self-signed certificate. Viewing the certificate gives the third flag: <code>flag3{f82366a9ddc064585d54e3f78bde3221}</code>

{% img center /images/pentest/skydogctf/flag3.jpg 'flag3' 'flag3' %}

And the next hint is <code>personnel</code>

## Flag #4 A Good Agent is Hard to Find.

I tried navigating to /personnel on the web server and it gave me the following message:

``` 
ACCESS DENIED!!! You Do Not Appear To Be Coming From An FBI Workstation. Preparing Interrogation Room 1. Car Batteries Charging....
```

So we need a specific user agent. Remembering the included JS file of the first flag, that hinted towards an old Internet Explorer version, I revisited it and searched for the string *fbi* inside it and found the lines:

``` 
/* maindev -  6/7/02 Adding temporary support for IE4 FBI Workstations */
/* newmaindev -  5/22/16 Last maindev was and idoit and IE4 is still Gold image -@Support doug.perterson@fbi.gov */
```

So FBI agents use IE4 on their workstations. Cool, nothing catastrophically wrong there. I grabbed an IE4 user agent from http://www.useragentstring.com/pages/useragentstring.php?name=Internet+Explorer, changed my User Agent to <code>User-Agent: Mozilla/4.0 WebTV/2.6 (compatible; MSIE 4.0)</code>, and went straight to the FBI portal!

{% img center /images/pentest/skydogctf/fbiportal.png 'fbi portal' 'fbi portal' %}

The flag is on the page: <code> flag{14e10d570047667f904261e6d08f520f}</code>, and the decoded value is <code>evidence</code>

## Flag #5 The Devil is in the Details - Or is it Dialogue? Either Way, if it’s Simple, Guessable, or Personal it Goes Against Best Practices

Following up on the clue *new+flag*, I tried navigating to 192.168.145.136/newevidence and a basic authentication prompt asked me for credentials. On the FBI portal we have the message "Welcome Agent Hanratty". A quick Google reveals that this is a reference to agent Carl Hanratty from the movie Catch Me If You Can. I went through the dialogue (hint hint) on the [IMDB page](https://www.imdb.com/title/tt0264464/characters/nm0000158) to find out Carl's daughter is called Grace (simple? guessable? personal?). I tried some combinations in the pop up until I found the right one is *carl.hanratty/Grace* (remember the naming convention of doug.perterson)

{% img center /images/pentest/skydogctf/newevidence.png 'new evidence' 'new evidence' %}

The flag is in the Evidence Summary File: <code>flag{117c240d49f54096413dd64280399ea9}</code>. The hint is <code>panam</code>

## Flag #6 Where in the World is Frank?

For this one you have to work with the other 2 files from the page, the image and the PDF invoice. The invoice mentions an encryption consultation project:

{% img center /images/pentest/skydogctf/invoice.jpg 'invoice' 'invoice' %}

This might suggest the use of steganography in the image, and this is confirmed if you search for Stefan Hetzl, it turns out he's the author of steghide! And we have the previous hint of panam, which is the passphrase in this case:

``` 
steghide extract -sf image.jpg 
Enter passphrase: 
wrote extracted data to "flag.txt".
root@kali:~/Downloads# cat flag.txt 
flag{d1e5146b171928731385eb7ea38c37b8}
=ILoveFrance

clue=iheartbrenda
```

## Flag #7 Frank Was Caught on Camera Cashing Checks and Yelling - I’m The Fastest Man Alive!

This was a movie reference and I had to look it up in other solutions..."I’m The Fastest Man Alive!" is a reference to The Flash, whose real name is Barry Allen. For the next part, log in to SSH with the the username *barryallen* and the password *iheartbrenda*.

``` 
ssh barryallen@192.168.145.136 -p 22222
[...]
barryallen@skydogconctf2016:~$ cat flag.txt 
flag{bd2f6a1d5242c962a05619c56fa47ba6}
```

And the hint for the last flag is <code>theflash</code>.

## Flag #8 Franks Lost His Mind or Maybe it’s His Memory. He’s Locked Himself Inside the Building. Find the Code to Unlock the Door Before He Gets Himself Killed!

Inside barryallen's home there's a 72M ZIP file called security-system.data:

``` 
barryallen@skydogconctf2016:~$ file security-system.data 
security-system.data: Zip archive data, at least v2.0 to extract
```

I transferred it to my machine with scp:

``` 
scp -P 22222 barryallen@192.168.145.136:security-system.data .
###############################################################
#                         WARNING                             #
#		FBI - Authorized access only!                 # 
# Disconnect IMMEDIATELY if you are not an authorized user!!! #
#         All actions Will be monitored and recorded          #
#	Flag{53c82eba31f6d416f331de9162ebe997}		      #
###############################################################
barryallen@192.168.145.136's password: 
security-system.data                          100%   71MB   6.6MB/s   00:10    
```

I unzipped the archive, but the file command didn't identify it as something specific, it just said it's data. I ran strings on it and saw various references to memory that seem to be linked to a Windows system. Time for some memory forensics with Volatility!

First, we need to identify what type of image we're working with:

``` 
---------------------------------
Module ImageInfo
---------------------------------
Identify information for the image 


volatility imageinfo -f data 
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86 (Instantiated with WinXPSP2x86)
                     AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                     AS Layer2 : FileAddressSpace (/root/Downloads/data)
                      PAE type : PAE
                           DTB : 0x33e000L
                          KDBG : 0x80545b60L
          Number of Processors : 1
     Image Type (Service Pack) : 3
                KPCR for CPU 0 : 0xffdff000L
             KUSER_SHARED_DATA : 0xffdf0000L
           Image date and time : 2016-10-10 22:00:50 UTC+0000
     Image local date and time : 2016-10-10 18:00:50 -0400
```

It's an image of a Windows XP machine! Next, let's look at the processes:

``` 
---------------------------------
Module PSList
---------------------------------
Print all running processes by following the EPROCESS lists


volatility pslist -f data 
Volatility Foundation Volatility Framework 2.6
Offset(V)  Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                          Exit                          
---------- -------------------- ------ ------ ------ -------- ------ ------ ------------------------------ ------------------------------
0x867c6830 System                    4      0     57      171 ------      0                                                              
0x86262900 smss.exe                332      4      3       19 ------      0 2016-10-10 21:59:14 UTC+0000                                 
0x8623b978 csrss.exe               560    332     10      423      0      0 2016-10-10 21:59:14 UTC+0000                                 
0x865ed020 winlogon.exe            588    332     24      512      0      0 2016-10-10 21:59:14 UTC+0000                                 
0x8662d808 services.exe            664    588     15      263      0      0 2016-10-10 21:59:14 UTC+0000                                 
0x866a5670 lsass.exe               676    588     25      356      0      0 2016-10-10 21:59:14 UTC+0000                                 
0x86358a70 vmacthlp.exe            848    664      1       25      0      0 2016-10-10 21:59:14 UTC+0000                                 
0x86651da0 svchost.exe             860    664     21      202      0      0 2016-10-10 21:59:14 UTC+0000                                 
0x865c2790 svchost.exe             944    664     11      258      0      0 2016-10-10 21:59:14 UTC+0000                                 
0x86554020 svchost.exe            1040    664     82     1287      0      0 2016-10-10 21:59:14 UTC+0000                                 
0x866196b8 svchost.exe            1092    664      5       59      0      0 2016-10-10 21:59:14 UTC+0000                                 
0x8643ca18 svchost.exe            1144    664     17      213      0      0 2016-10-10 21:59:15 UTC+0000                                 
0x866fca88 explorer.exe           1540   1520     14      417      0      0 2016-10-10 21:59:16 UTC+0000                                 
0x8656b4d0 spoolsv.exe            1636    664     15      125      0      0 2016-10-10 21:59:16 UTC+0000                                 
0x86338640 VGAuthService.e        1900    664      2       60      0      0 2016-10-10 21:59:25 UTC+0000                                 
0x8667bda0 vmtoolsd.exe           2012    664      9      271      0      0 2016-10-10 21:59:28 UTC+0000                                 
0x864f6440 wmiprvse.exe            488    860     14      251      0      0 2016-10-10 21:59:28 UTC+0000                                 
0x864fbad0 wscntfy.exe             536   1040      1       31      0      0 2016-10-10 21:59:28 UTC+0000                                 
0x85e5dd48 alg.exe                 624    664      8      110      0      0 2016-10-10 21:59:28 UTC+0000                                 
0x866f98b0 vmtoolsd.exe           1352   1540      7      242      0      0 2016-10-10 21:59:29 UTC+0000                                 
0x86674410 ctfmon.exe             1356   1540      1       79      0      0 2016-10-10 21:59:29 UTC+0000                                 
0x865bea48 CCleaner.exe           1388   1540      5      108      0      0 2016-10-10 21:59:29 UTC+0000                                 
0x865c3d78 cmd.exe                1336   1540      1       30      0      0 2016-10-10 22:00:05 UTC+0000                                 
0x8634fbb8 wuauclt.exe            1884   1040      9      198      0      0 2016-10-10 22:00:13 UTC+0000                                 
0x86260a78 wuauclt.exe            1024   1040      6      172      0      0 2016-10-10 22:00:29 UTC+0000                                 
0x8667b488 notepad.exe             268   1540      1       55      0      0 2016-10-10 22:00:41 UTC+0000                                 
0x8640cc10 cmd.exe                1276   2012      0 --------      0      0 2016-10-10 22:00:49 UTC+0000   2016-10-10 22:00:50 UTC+0000 
```

I took note of that Notepad process and several cmd.exes and then dumped the files:

``` 
---------------------------------
Module FileScan
---------------------------------
Pool scanner for file objects


volatility filescan -f data > filelist.txt
Volatility Foundation Volatility Framework 2.6
```

I searched for the string code in the resulting list of files and got a hit:

``` 
grep "code" filelist.txt 
0x0000000005e612f8      1      0 -W-r-- \Device\HarddiskVolume1\Documents and Settings\test\Desktop\code.txt
0x00000000062e04b0      1      0 R--r-d \Device\HarddiskVolume1\Documents and Settings\test\Recent\code.txt.lnk
0x00000000064900a0      1      0 R--rwd \Device\HarddiskVolume1\WINDOWS\system32\unicode.nls
0x0000000006640bc8      1      0 R--rwd \Device\HarddiskVolume1\Documents and Settings\test\Desktop\code.txt
```

Maybe this code.txt file is connected to the running Notepad process. I used Volatility's notepad plugin to dump the text found into Notepad:

``` 
---------------------------------
Module Notepad
---------------------------------
List currently displayed notepad text


volatility notepad -f data 
Volatility Foundation Volatility Framework 2.6
Process: 268
Text:
?

Text:
d

Text:


Text:
?

Text:
66 6c 61 67 7b 38 34 31 64 64 33 64 62 32 39 62 30 66 62 62 64 38 39 63 37 62 35 62 65 37 36 38 63 64 63 38 31 7d
```

We have a hex string! Decoding it yields the final flag: <code>flag{841dd3db29b0fbbd89c7b5be768cdc81}</code>. And the decoded value for the last flag is <code>Two[space]little[space]mice</code>

This is it for the SkyDog Con CTF machine! I particularly liked the memory forensics part at the end!

``` 
 ____________________________________
/ Q: Do you know what the death rate \
\ around here is? A: One per person. /
 ------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

