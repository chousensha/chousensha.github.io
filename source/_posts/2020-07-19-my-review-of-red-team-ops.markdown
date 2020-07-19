---
layout: post
title: "My review of Red Team Ops"
date: 2020-07-19 19:54:09 +0300
comments: true
categories: [red team, training, hacking, reviews]
keywords: red team ops, rto, rastamouse, zeropoint security
description: Red Team Ops review
---

Red Team Ops is a red team course launched by none other than RastaMouse. It was notoriously hard to get a free spot as the labs were full almost of the time since launch. I resorted to checking every day until I found a free seat to pounce on. Without a doubt, I can say it's been the best training experience I've undertaken, and I wanted to leave a more in-depth review. You can find an overview of the course and its syllabus on the official page: https://www.zeropointsecurity.co.uk/red-team-ops

<!-- more -->

RTO consists of the course materials, which are hosted on an online learning platform, and the labs, with users, servers, and a total of 3 domains. The materials are structured in such a way that you go more or less in order to complete the objectives. The explanations and examples are clear and concise, giving you enough for the task at hand. I liked the no fluff approach, and I also recognized that you can get a lot more out of the course if you research deeper into some topics to gain a better understanding of what's going on. 

The course mirrors a red team engagement where the objective is not to pwn all the things but to compromise a target organization and exfiltrate important data. You start from the outside with minimal information and you have to go through recon, OSINT and phishing to get a foothold on the internal network. Then you get to use the taught techniques for privilege escalation, lateral movement, domain compromise and evading defenses, as well as maintaining persistence in the environment. Don't expect to use exploits on this course. Rather, you have to exploit typical Windows and Active Directory misconfigurations to accomplish your objectives. A lot of AD attacks are covered, as well as different tradecraft for various scenarios.

At the beginning the course used only Covenant as the C2 framework of choice, but with the continous updates now it also shows how to accomplish the objectives with Cobalt Strike. I learned a lot from using Covenant, even though at times its bugs drove me crazy. The course was initially created for Covenant 0.4, but since the 0.5 release, it got updates to reflect that. Covenant is really powerful for such a new framework, but at the same time is not as feature rich as older frameworks, so the course shows you ways to use other tools to augment the Covenant usage when necessary. In particular, I really learned a ton for using Metasploit for SOCKS proxies and reverse port forwards. 

There is heavy emphasis on .NET tradecraft in the course, which is really nice considering that it's the current MO of choice for offensive operations. However, there is plenty of Powershell usage as well, so expect a well-rounded tradecraft experience. You don't just use tools all the time, sometimes you get to slightly modify code and compile tools and implants (called grunts in Covenant) to achieve the desired end state. All in all, the labs are spot on in terms of current tradecraft usage. There is a nice gamification effect as well. Throughout the labs you have to complete certain objectives by using specific attacks, so you can't skip content.

The exam is a beast on its own. You have 48 hours to collect 4 flags, and the passing score is 3 flags. You don't have to write a report. In the exam you will have to combine everything you learned and be really comfortable with the techniques and bypasses, ideally also how to achieve the same thing in different ways. It's a fun and difficult exam where you will switch from despair to joy and so on.

There are a couple other things that make an excellent course even greater, and that I haven't really seen with other courses, which I will list below.

* **Continuous updates**. RastaMouse was true to his word that this course would receive regular updates. A number of sections were added, or more explanations and examples were added for specific sections, as well as rewriting the Covenant usage to reflect the changes from 0.4 to 0.5 and adding the Cobalt Strike examples to show how the same thing can be done with CS instead of Covenant. 

* **Lifetime access**. Combined with the above, it's really amazing. Once you buy the course, you get lifetime access to the materials, including the updates, so you can always go back and absorb the newly added material. You also have a different channel for buying lab extensions than the new purchases, with more custom lab time than the initial 1, 2, or 3 month duration.

* **Fantastic support and community**. The support offered in this course is exceptional. RastaMouse is super fast to help with any issues or to offer guidance on the dedicated Slack channel. And the channel is a perfect place to discuss with other students and learn from others, so it's definitely recommended to hang around!

* **Course format**. There are 2 ways to get the Certified Red Team Operator certification. The external one is by having the OSCP and passing the RTO exam. The internal way, which I strongly recommend, is to achieve all the course objectives (you are granted a badge for each completed objective) and pass the RTO exam. The badge format is a nice motivator, as well as keeping your progress visualized and ensuring you don't skip tasks.

As a conclusion, Red Team Ops was a fantasting learning experience for me and thanks to RastaMouse for creating it!

``` 
  _______
< Indeed. >
  -------
         \   ^__^ 
          \  (oo)\_______
             (__)\       )\/\
                 ||----w |
                 ||     ||
```



