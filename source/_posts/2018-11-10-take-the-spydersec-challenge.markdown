---
layout: post
title: "Take the SpyderSec Challenge"
date: 2018-11-10 07:43:09 -0500
comments: true
categories: [penetration testing, writeups]
keywords: spydersec, pentest lab, vulnhub, ctf, boot2root, spydersec challenge
description: Spydersec writeup
---

The challenge:

> You are looking for two flags. Using discovered pointers in various elements of the running web 
> application you can deduce the first flag (a downloadable file) which is required to find the second
> flag (a text file). Look, read and maybe even listen. You will need to use basic web application 
> recon skills as well as some forensics to find both flags. 

<!-- more -->

``` 
PORT   STATE  SERVICE VERSION
22/tcp closed ssh
80/tcp open   http    Apache httpd
|_http-server-header: Apache
|_http-title: SpyderSec | Challenge
```

There is just one page for the web server:

{% img center /images/pentest/spydersec/spydersec.png 'spydersec' 'spydersec' %} 

There is some peculiar Javascript in the source:

``` 
<script>
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('7:0:1:2:8:6:3:5:4:0:a:1:2:d:c:b:f:3:9:e',16,16,'6c|65|72|27|75|6d|28|61|74|29|64|62|66|2e|3b|69'.split('|'),0,{}))
</script>
```

I used an [online JS beautifier](https://beautifier.io/) to deobfuscate this to a hex string, which I then decoded to <code>alert('mulder.fbi');</code>. This name is familiar to me from the X-Files. Anyway, not sure about its relevance for now. Moving on with the information gathering, since nothing came out of Nikto or Gobuster, I browsed the page in Burp, and noticed a strange cookie being set:

``` 
Cookie: URI=%2Fv%2F81JHPbvyEQ8729161jd6aKQ0N4%2F
```

URL decoding this gave me the value <code>/v/81JHPbvyEQ8729161jd6aKQ0N4/</code> Tried browsing to this and got a 403 Forbidden error. There is some piece of the puzzle missing..so I took the next step and downloaded all the images and ran them through exiftool..and finally got some luck with the Challenge.png image, which had a hex comment:

``` 
Comment                         : 35:31:3a:35:33:3a:34:36:3a:35:37:3a:36:34:3a:35:38:3a:33:35:3a:37:31:3a:36:34:3a:34:35:3a:36:37:3a:36:61:3a:34:65:3a:37:61:3a:34:39:3a:33:35:3a:36:33:3a:33:30:3a:37:38:3a:34:32:3a:34:66:3a:33:32:3a:36:37:3a:33:30:3a:34:61:3a:35:31:3a:33:64:3a:33:64
```

Had to hex-decode this 2 times to get a base64 string: *QSFWdX5qdEgjNzI5c0xBO2g0JQ==*, which I decoded to <code>A!Vu~jtH#729sLA;h4%</code>. This looks like a password, but where to use it?

With the hints from the description, knowing that a file will need to be downloaded, I treated mulder.fbi as a file and tried appending it to the URL path that gave me a forbidden error: http://192.168.145.141/v/81JHPbvyEQ8729161jd6aKQ0N4/mulder.fbi

That actually worked and I downloaded the file, which is apparently an MP4:

``` 
file mulder.fbi 
mulder.fbi: ISO Media, MP4 v2 [ISO 14496-14]
```

This is a song by The Platters. Since I couldn't use steghide on a video, I googled for MP4 steganography and found this Lifehacker article about [embedding a TrueCrypt volume in a video](https://lifehacker.com/5771142/embed-a-truecrypt-volume-in-a-playable-video-file). I installed Veracrypt and used it to mount the file as a volume, in TrueCrypt mode. At the password prompt, I used the string that I've decoded earlier, and it served me the volume, with a file called Flag.txt inside!

``` 
cat Flag.txt 
Congratulations! 

You are a winner. 

Please leave some feedback on your thoughts regarding this challenge� Was it fun? Was it hard enough or too easy? What did you like or dislike, what could be done better?

https://www.spydersec.com/feedback
```

The main takeaways from this challenge for me were the steganography possibilities for video files. Until next time!

``` 
 ________________________________________
/ Q: How many hardware engineers does it \
| take to change a light bulb? A: None.  |
| We'll fix it in software.              |
|                                        |
| Q: How many system programmers does it |
| take to change a light bulb? A: None.  |
| The application can work around it.    |
|                                        |
| Q: How many software engineers does it |
| take to change a light bulb? A: None.  |
| We'll document it in the manual.       |
|                                        |
| Q: How many tech writers does it take  |
| to change a light bulb? A: None. The   |
\ user can figure it out.                /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
``` 



