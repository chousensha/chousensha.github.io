---
layout: post
title: "Infosec Institute n00bs CTF Labs"
date: 2015-06-05 20:39:07 +0300
comments: true
categories: [penetration testing, ctf, writeups]
keywords: infosec, ctf, hacking labs, infosec institute ctf, infosec institute ctf solutions
description: Walkthrough for the Infosec Institute n00bs CTF Labs
---

It's been a while since I've last polished my web hacking skills, and I recently found out about these CTF challenges. Too late for the bounty though..

You can access the labs at http://ctf.infosecinstitute.com/index.php

<!-- more -->

### Level 1

This is straightforward, just listen to Yoda:

{% img center /images/infosec_institute_ctf/level1.png 'level1' 'level1' %}

You can find the flag in the source: <code>infosec_flagis_welcome</code>

### Level 2

{% img center /images/infosec_institute_ctf/level2.png 'level2' 'level2' %}

Found the image in the source:

``` html
<img src="img/leveltwo.jpeg" />
```
When clicking on it, there is this error:

<code>The image “view-source:http://ctf.infosecinstitute.com/img/leveltwo.jpeg” cannot be displayed because it contains errors.</code>

Downloaded it and ran *strings* on it:

``` plain
root@kali:~/Desktop# strings leveltwo.jpeg 
aW5mb3NlY19mbGFnaXNfd2VhcmVqdXN0c3RhcnRpbmc=
```

Well, well. This isn't suspicious at all! :D After you Base64 decode it, you get the flag: <code>infosec_flagis_wearejuststarting</code>

### Level 3

{% img center /images/infosec_institute_ctf/level3.png 'level3' 'level3' %}

I used an [online barcode scanner](http://www.onlinebarcodereader.com/) that produced this output: <code>.. -. ..-. --- ... . -.-. ..-. .-.. .- --. .. ... -- --- .-. ... .. -. --.</code>. Well, that seems familiar..Morse code! Used [this site](http://www.onlineconversion.com/morse_code.htm) to decode it: <code>INFOSECFLAGISMORSING</code>

### Level 4

{% img center /images/infosec_institute_ctf/level4.png 'level4' 'level4' %}

If you hover over the image with your mouse, a popup appears with the message *Stop poking me!*. Looking in the source, the Javascript function responsible for that is called <code>poke()</code>, and you can find it in the <code>custom.js</code> file. All it does is alert the message. I spent some time here going in a wrong direction, disabling Javascript, and trying to see if something is hidden in the image. The hint states HTTP, and I did look at the headers and all, and it's not often that you see a cookie called *fusrodah*. But initially I didn't think it was pertaining to this specific level, because it was present in the requests to other levels as well. But when I hit a blank on everything else, I returned to it and ran it in some decoders:

``` plain
fusrodah=vasbfrp_syntvf_jrybirpbbxvrf
```

And I hit the jackpot with a Caesar shift of 13: <code>infosec_flagis_welovecookies</code>

### Level 5

When you click on Level 5, you immediatelly get a popup saying *Hacker!!!*. I disabled Javascript to see this image:

{% img center /images/infosec_institute_ctf/level5.png 'level5' 'level5' %}

Right, back to the thing I hate most, steganography. Stegdetect didn't find anything, and I didn't have any luck with StegExpose either. Surprisingly, an online tool came to help: https://futureboy.us/stegano/decinput.html

The output was this string: 

``` plain
01101001011011100110011001101111011100110110010101100011010111110110011001101100011000010110011101101001011100110101111101110011011101000110010101100111011000010110110001101001011001010110111001110011
``` 

which I then ran in a binary decoder: <code>infosec_flagis_stegaliens</code>

### Level 6

{% img center /images/infosec_institute_ctf/level6.png 'level6' 'level6' %}

Packet analysis is not really my thing, so the way I solved this was by just following streams until I hit on something potentially interesting: the string <code>696e666f7365635f666c616769735f736e6966666564</code>, contained in the first UDP packet, with a source and destination of 127.0.0.1. This was actually the hex encoded flag: <code>infosec_flagis_sniffed</code>

### Level 7

{% img center /images/infosec_institute_ctf/level7.png 'level7' 'level7' %}

I tried to manually navigate to levelseven.php and all I got was a blank page. But when I looked at the request with Live HTTP Headers, I noticed this: <code>HTTP/1.0 200 aW5mb3NlY19mbGFnaXNfeW91Zm91bmRpdA==</code>. A base64 encoded string...and decoding it gives the flag: <code>infosec_flagis_youfoundit</code>

### Level 8

{% img center /images/infosec_institute_ctf/level8.png 'level8' 'level8' %}

Well, didn't expect to solve it just by running *strings*, but that's all you need to do! :D The flag is in the *strings* output: <code>infosec_flagis_0x1a</code>

### Level 9

{% img center /images/infosec_institute_ctf/level9.png 'level9' 'level9' %}

No SQLi involved here..so I googled for Cisco IDS Web Login System password. Eventually I found the credentials that work on [this page](http://www.anameless.com/blog/default-passwords.html) that contains default passwords for a number of devices. The ones that worked were <code>root:attack</code>. A popup appeared with the flag <code>ssaptluafed_sigalf_cesofni</code>, which is the reverse for <code>infosec_flagis_defaultpass</code>

### Level 10

{% img center /images/infosec_institute_ctf/level10.png 'level10' 'level10' %}

Played the .wav file, it sounded like something being fast forwarded. So I launched Audacity, and under Effect->Change Speed, you can play with the Speed Multiplier (some 0.20 and lower) until you hear a voice telling you the flag: <code>infosec_flagis_sound</code>

### Level 11

{% img center /images/infosec_institute_ctf/level11.png 'level11' 'level11' %}

Right, that huge PHP image is suspicious. I downloaded it and ran *strings* on it to begin with, and: 

``` plain
infosec_flagis_aHR0cDovL3d3dy5yb2xsZXJza2kuY28udWsvaW1hZ2VzYi9wb3dlcnNsaWRlX2xvZ29fbGFyZ2UuZ2lm
```

Base64? Yes it is! Decoding it gives the address of another image: http://www.rollerski.co.uk/imagesb/powerslide_logo_large.gif

{% img center /images/infosec_institute_ctf/powerslide.png 'powerslide' 'powerslide' %}

Couldn't find anything hidden in this image, so I guess <code>infosec_flagis_powerslide</code>

### Level 12

{% img center /images/infosec_institute_ctf/level12.png 'level12' 'level12' %}

Spent some time scratching my head at this one, because I couldn't find anything. Since I was just jumping over the initial section of the page source, with the CSS files and all, I was missing the relevant information. So, if you look in the source and you compare it with other pages, you can see that there is a new CSS file, called *design.css*, with the following inside:

``` css
.thisloveis{
	color: #696e666f7365635f666c616769735f686579696d6e6f7461636f6c6f72;
}
```

And if you decode that hex string, you get the flag: <code>infosec_flagis_heyimnotacolor</code>

### Level 13

{% img center /images/infosec_institute_ctf/level13.png 'level13' 'level13' %}

I kept trying editing the URL and adding file extensions commonly associated with backup files from [this list](http://www.file-extensions.org/filetype/extension/name/backup-files). Eventually, stumbled upon one that works: the *.old* file extension. 

[What is an OLD file?](http://file.org/extension/old):

> Files that contain the .old file extension are most commonly used to indicate that a file on a user's hard drive is a backup copy of a newer 
> version of the file. The .old extension is given to the file name when the newer version of the file is saved within the associated computer 
> application.

> The initial file extension is often kept intact when the .old file extension is assigned to a file. For example, if the original version of a file
> is saved as mydocument.doc, then when a new version of the file is created that version will also be saved with the name of mydocument.doc. 
> However, in order to store a copy of the old version of the file, the original version will be saved with the name as mydocument.doc.old.

So when I added *.old* to the URL, I got a message if I want to open or download levelthirteen.php.old. Ok, now we're getting somewhere! Ran *strings* on it (again!) and noticed an extra something in the source:

``` html
/* <img src="img/clippy1.jpg" class="imahe" /> <br /> <br />
    <p>Do you want to download this mysterious file?</p>
    <a href="misc/imadecoy">
      <button class="btn">Yes</button>
    </a>
    <a href="index.php">
      <button class="btn">No</button>
    </a>
    */
```

I went to http://ctf.infosecinstitute.com/misc/imadecoy and you can download the mysterious file. Apparently it's another job for Wireshark:

``` plain
root@kali:~/Desktop# file imadecoy
imadecoy: tcpdump capture file (little-endian) - version 2.4 (Linux "cooked", capture length 65535)
```

Ok, more random following streams and trying to glean what's interesting. Eventually, I reached this part:

{% img center /images/infosec_institute_ctf/imadecoy.png 'imadecoy' 'imadecoy' %}

The GET request for that HoneyPY.PNG image occurs a few times after first spotting it. So the image might be interesting. And it's possible to reconstruct it from the packet file! Click on the relevant packet:

``` plain
633	46.399534	127.0.0.1	127.0.0.1	HTTP	1955	HTTP/1.1 200 OK  (PNG)
```

Then go to File->Export Objects->HTTP. Wireshark will then give you all the HTTP objects list:

{% img center /images/infosec_institute_ctf/http_objects.png 'http object list' 'http objects' %}

We're only interested in the image, which is the last one. I saved it, and...

{% img center /images/infosec_institute_ctf/flag13.png 'flag13' 'flag13' %}

### Level 14

{% img center /images/infosec_institute_ctf/level14.png 'level14' 'level14' %}

The file is a phpMyAdmin SQL Dump. If you scroll through it, towards the end there will be this table:

``` plain
-- Dumping data for table `friends`
--

INSERT INTO `friends` (`id`, `name`, `address`, `status`) VALUES
(102, 'Sasha Grey', 'Vatican City', 'Active'),
(101, 'Andres Bonifacio', 'Tondo, Manila', 'Active'),
(103, 'lol', 'what the???', 'Inactive'),
(104, '\\u0069\\u006e\\u0066\\u006f\\u0073\\u0065\\u0063\\u005f\\u0066\\u006c\\u0061\\u0067\\u0069\\u0073\\u005f\\u0077\\u0068\\u0061\\u0074\\u0073\\u006f\\u0072\\u0063\\u0065\\u0072\\u0079\\u0069\\u0073\\u0074\\u0068\\u0069\\u0073', 'annoying', '0x0a');
```

Used [this decoder](http://ddecode.com/hexdecoder/) to decode the not-at-all conspicuous string and: <code>infosec_flagis_whatsorceryisthis</code> ! :D

### Level 15

Something seems wrong with the server and I get a (real) 404 error for this level only. When it works again I will update this section

{% img center /images/infosec_institute_ctf/cookie.png 'cookie' 'cookie' %}
 
