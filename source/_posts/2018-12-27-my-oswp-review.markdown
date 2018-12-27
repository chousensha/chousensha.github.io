---
layout: post
title: "My OSWP review"
date: 2018-12-27 18:13:52 +0200
comments: true
categories: [reviews]
keywords: oswp, offensive security wireless professional, wifu, offensive security, wireless hacking, wifi, wireless security, owsp exam, oswp review
description: My 2018 review of the OSWP certification
---


I wanted to finish 2018 on a strong note, so right before Christmas I've completed the exam for the [Offensive Security Wireless Professional](https://www.offensive-security.com/information-security-certifications/oswp-offensive-security-wireless-professional/) (OSWP) certification. This is my review for the [Offensive Security Wireless Attacks (WiFu)](https://www.offensive-security.com/information-security-training/offensive-security-wireless-attacks/) course. But as a short summay, the course and exam were great, very practical and applicable, and I thoroughly enjoyed them!

<!-- more -->

{% img center /images/study/oswp/oswp.png 'oswp' 'oswp' %}

## The course

The course consists of PDF material, videos, a BackTrack ISO and packet captures. The labs are hardware dependent, so you are supposed to procure your own hardware in order to practice the attacks. For my wireless card, I've used Alfa AWUS036NHA and have been very happy with its performance. In the course, BackTrack is being used as the operating system, but I've done the labs on Kali with minimal to none difference in commands.

As a testimony to the applicability of what you learn in the course, the author is none other than Thomas d'Otreppe de Bouvette, the creator of the Aircrack-NG suite! The material is very well structured and explained, you can practice what you're taught and see immediate results. Most chapters also contain a troubleshooting section with tips on what to try if you run into problems.

Almost half of the course written material revolves around the description of Wifi internals, how it works, how packets look like, hardware and drivers consideration. There are PCAP examples for what is being discussed, and it is really useful if you want to understand what is happening under the hood.

After that, it's all attacks and the stuff we're interested in! The focus of the course materials is the Aircrack-NG suite of tools, as expected, since it's the de facto choice for wireless security auditing. You learn how to place your wifi card on monitor mode, how to sniff traffic, how to do packet injection and ultimately crack the keys for different network scenarios. You also learn how to find hidden SSIDs and bypass MAC filtering.

The attacks you perform are targeted at different network types. There are multiple scenarios for WEP networks that use open authentication, shared key authentication, or have no associated clients. The labs go over multiple attacks: interactive packet replay attacks, fragmentation attacks, KoreK ChopChop attacks.

The most applicable part is the WPA/WPA2 cracking section, as these are the networks you'll run into most often nowadays. Dictionary and rainbow table attacks are being demonstrated with multiple tools. I've really enjoyed the addition in the course material of Cowpatty and Pyrit. Also, you're shown how to leverage John's rules to enhance an already existing wordlist that's used with Aircrack-NG.

At the end of the course, you also get to see Kismet and Karmetasploit in action, along with rogue AP and man-in-the-middle attacks.

## The exam

{% img center /images/study/oswp/skills.png 'oswp skills' 'oswp skills' %}

The exam was really enjoyable, I didn't find it too hard or too easy. If you practice the labs in the course, you will be well prepared for the exam. I really liked that the exam takes almost 4 hours, as that allowed me to also work on the report as I was going through each attack. I've experienced some laggy bits and a disconnection here and there, but overall, the exam experience was great.

### Concluding remarks

I could think of no better course for wireless hacking, but as I was reading reviews of other people's experiences, I saw some criticism concerning this particular course. I will share my thoughts on the critiques below.

* *The course is outdated*. It looks like the course was last updated in 2014, and the videos and labs make use of BackTrack. But these don't detract from its value at all. I used Kali for the course, and I think only 2 times I had to slightly alter the commands for use on Kali. It would make no sense to re-record everything just to be on Kali, since there are almost no differences between them. Also, the wifi landscape hasn't changed that much, so the material is highly relevant and you can put into practice what you learn right away. But I do agree that the course would be even better with a section of WPA enterprise attacks and WPS cracking.

* *The material is too WEP-centric*. It's true that today you won't really see WEP that much. But that's not to say that there are no WEP networks left in the world. Seeing how vulnerable they are in reality and being able to crack different types of WEP networks is a skill that you should have, regardless of how rare you'll get to see them. So for me, this wasn't a weak point at all, I found it very interesting.

* *The exam is too easy*. I've also seen several opinions claiming that the exam was too easy. But this only attests to the quality of the material, if you can perform the attacks you're being taught, you will succeed. Maybe as an enhancement to the exam, besides cracking the keys, there could be some objectives like decrypting the traffic of a certain client to discover something, or performing MitM. But I was happy with the quality of the exam, in fact, I tried to make the most of it by going through the attacks with more than one tool.

All in all, OSWP was awesome, and stay tuned for more!

``` 
 ________________________________________
/ Living your life is a task so          \
| difficult, it has never been attempted |
\ before.                                /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

