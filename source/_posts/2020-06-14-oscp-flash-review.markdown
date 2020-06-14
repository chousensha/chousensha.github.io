---
layout: post
title: "OSCP flash review"
date: 2020-06-14 20:07:43 +0300
comments: true
categories: [penetration testing, training, hacking, reviews]
keywords: oscp, offensive security, offsec, pwk
description: Quick OSCP review
---

I've finished my OSCP shortly before [PWKv2](https://www.offensive-security.com/pwk-oscp/) was released. Since my experience was pre-update, I didn't want to write a review that might no longer apply. Since then, I've purchased the updated course materials, but I haven't yet gone back to do the newly added machines as well. I decided to write a quick review nonetheless, because I noticed a misconception about what the OSCP is. So I'll just address those points and not add just another OSCP review for the internet to swallow.

<!-- more -->

A quick rundown of my experience, I did all the lab machines and all the exam machines within 3 months. I plan to go back and also do the ones that were added by the update, but right now I am too busy with other things. After I do that, maybe I'll update this post or write a new one more focused on the impressions of the update. In terms of the new course materials, which I did review, there's a lot compared to the previous version. In particular, the new Windows section is superb and there are lots of usable snippets for various scenarios in modern environments.

Back to the reason for this post, it's not that I want to add just another post with what I did to prepare and the kind of information that is widely available already through the experiences of so many other successful takers (and you should read those posts and learn the most out of them to help you with your journey). My reason is that I noticed many people consider the OSCP the end all, be all of pentesting and are under the impression that it automatically makes you a pentesting god, while there are others who think it's overrated and that you don't learn much out of it. I consider both these opinion categories wrong. Let's dive in.

### My 50 cents about the OSCP

After doing the OSCP and also moving beyond it, I consider it more like a necessary qualification to go through as a baseline for pentesting, much like the bootcamp in an army, training camp of a fight, or the qualifiers to a World Cup. It's hard and diverse enough, but it doesn't cover everything. It's mostly focused on network and web exploitation, along with enumeration and privilege escalation. Achieving it will demonstrate perseverance in front of obstacles, a working pentest methodology and skill, and a reasonable level of experience performing it. From that point, there's a lot on the horizon..going deeper in network and web attacks, exploit development, Active Directory..you realize when you're done that you just passed through a warpgate and now there's a whole lot of new challenges lining up. But the principles you learned through completing it will continue to be usable in your career and in future challenges.

### You get out what you put in

I've heard of several people who jumped into the exam after only going through a few machines in the labs. You are literally robbing yourself of the most valuable thing the OSCP provides. All the lab machines present a wide variety of operating systems (not just the usual flavors of Windows and Linux) and attack vectors. Some machines have different and unique vectors hidden in the attack surface that are way more rewarding than the obvious one. I don't think I used the same exploit more than 2 times in the lab. Each machine was a learning experience by itself. If you're running exploit suggester and throwing kernel exploits until one works, you're doing it wrong. Also, the hidden networks present a new challenge, in the way of pivoting, that needs to be solved before even reaching the targets. Not to mention the client side attacks and their difficulties.

Too many people are focusing just on the finality of earning a certification instead of squeezing the most out of all the labs have to offer. And even after you complete every box, don't imagine the exam won't be difficult. But you will be as prepared as you can get. Plus it's fun! If you don't get excited about the prospect of pwning over 50 boxes, why are you doing it in the first place? Your ROI will come from the effort you put into the labs, not from the letters you put on your resume.


### Outdated machines?

Another thing I've heard often is the complaint about lab machines being too outdated. Yes, the majority of boxes have older OS flavors..and that's a good thing! You can try your hand at exploits that will teach you things that you wouldn't otherwise experience in a completely modern environment. This is the foundation you are building, the training you're doing for match day. Plus, you learn to achieve objectives without your favorite tools. How do you transfer files to and from an older Windows machine without Powershell and all other native tools that you're used to? What can you do if you have RCE on a NIX system that's different than the ones you're familiar with? And many more wins that you arrive at from struggling with each challenge you're presented.

### Target audience

Even with the new course update, which gives you a better starting point than before, I don't believe OSCP should be pursued for beginners. People who want to start in pentesting should first make sure they have their basics covered. Things like networking, system administration on Windows and Linux, programming, they are not basics as in easy and simple things, but as in the necessary foundation to add to with security. Trying to skip them won't end well. Once that is covered, there are lots of resources to prepare you, which I won't touch on here. VulnHub, HackTheBox, Ippsec, they have enough content to keep you busy for years. At some point, you have to dive in, but properly preparing for OSCP is as important an effort as working through it.


### Exam

It's hard and it's fun. Don't think just because you finished all the lab machines it will be a walk in the park - it won't ;). But I do believe that doing all the machines is the best way to gauge your readiness for it. 

Keeping it short and to the point I intended, these are my thoughts about the OSCP. It's an awesome training experience and challenge. You constantly switch back and forth between frustration and elation. And once you're on the other side..you don't stop there, but you go forward exploring all that new fog of war.

```
 _____________________________________
/ You definitely intend to start living \
\ sometime soon.                        /
  -------------------------------------
         \   ^__^ 
          \  (oo)\_______
             (__)\       )\/\
                 ||----w |
                 ||     ||
```


 
