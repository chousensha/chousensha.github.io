---
layout: post
title: "Book review - Nmap Network Scanning"
date: 2016-08-24 06:02:00 -0400
comments: true
categories: [books, reviews]
keywords: pentesting, nmap, hacking, penetration testing, nmap tutorial, nmap guide, port scan, scanner, nmap network guide, nmap book, nmap manual
description: Nmap Network Scanning&#58; The Official Nmap Project Guide to Network Discovery and Security Scanning review
---

Recently I have read the so-called Nmap bible, the Nmap Network Scanning book written by the creator of Nmap himself. I've decided to make
a new section on the blog with the books I'm reading and that I find most useful. Needless to say, this was a great book, that has upped my
Nmap game considerably. Right from the start, I highly recommend this as a book that should be in the arsenal of every pentester, system admin, 
network engineer or plain old computer guy. The amount of information you get out of it is not constrained to a specific field.

<a href="https://www.amazon.com/Nmap-Network-Scanning-Official-Discovery/dp/0979958717/ref=as_li_ss_il?ie=UTF8&linkCode=li2&tag=sensha10-20&linkId=791bdedacea7ac3bd1caea9852082708" target="_blank"><img src="https://images-na.ssl-images-amazon.com/images/I/51BIsSMZqML._SX382_BO1,204,203,200_.jpg"/></a>

<!-- more -->

# Contents

**Ch. 1. Getting Started with Nmap** - begins with some usage stories, and some legal information

**Ch. 2. Obtaining, Compiling, Installing, and Removing Nmap** - continues with how to install Nmap on different operating systems

**Ch. 3. Host Discovery (Ping Scanning)** - covers various host discovery techniques

**Ch. 4. Port Scanning Overview** - all you want to know about port scanning

**Ch. 5. Port Scanning Techniques and Algorithms** - port scanning techniques explained in more detail, along with their under-the-hood
implementations

**Ch. 6. Optimizing Nmap Performance** - fine-tuning your scans for more speed

**Ch. 7. Service and Application Version Detection** - all about the service detection capabilities of Nmap

**Ch. 8. Remote OS Detection** - how Nmap performs OS detection

**Ch. 9. Nmap Scripting Engine** - script creation and API information

**Ch. 10. Detecting and Subverting Firewalls and Intrusion Detection Systems** - firewall evasion techniques

**Ch. 11. Defenses Against Nmap** - defenses against Nmap scans

**Ch. 12. Zenmap GUI Users' Guide** - presents the Nmap GUI interface

**Ch. 13. Nmap Output Formats** - directing Nmap output to different file formats

**Ch. 14. Understanding and Customizing Nmap Data Files** - working with some of the main internal Nmap files

**Ch. 15. Nmap Reference Guide** - reference guide to Nmap commands

If you have enjoyed the Stealing the Network series, you will love this one! The style is similar, after all, Fyodor is also a 
co-author of that amazing series. The book is sprinkled with fictional stories that illustrate the use of Nmap, along with backstories
of engagements the author has participated in, and case studies of users improving Nmap for certain tasks. All these stories contain great
advice not just on the technical side, but also on best practices. For example, Fyodor stresses the fact of always confirming that the IP range
you received for an engagement actually belongs to the target organization - or else you might end up  scanning the wrong target..like the army :-) 

There are also plenty of funny comments and geeky stuff, from the friendly tip that "using Nmap from your work or school computer to attack 
banks and military targets is a bad idea" and the technique and Nmap command that Trinity used to scan the Matrix, to life-changing
advice that says "In life's bleakest moments when all hope seems to be lost, what should you turn to? Nmap, of course !". By the way, if you are curious,
the above mentioned scan against the Matrix was performed with <code>nmap -v -sS -0 10.2.2.2</code>. Fyodor also made a list on his site of other
movies that showcased the use of Nmap, which you can find at https://nmap.org/movies/

But I'm sure that you want to hear about the actual content that will help turn you into a network scanning wizard. There are tons 
of resources presented, not just related to Nmap, that can facilitate the information gathering process, such as DNS tools. Every Nmap command
is described in detail, with actual examples of using it, what happens under the hood when you run it, and  the situation where it will come 
most in handy. All the command line switches are covered individually, but also in combinations, for comprehensive coverage of different scenarios.

The book also contains a good amount of TCP information and low-level details of both Nmap and network operations in general, along with
beautiful and detailed diagrams. There is information on scrips and how to create your own to enhance Nmap, and also description of the code itself.
At the end, there is a reference guide for all commands. While some of the content is a bit dated, you can get the whole up-to-date information
on the https://nmap.org/ site, for free!

Finally, this awesome book was created with open source tools. Hats off to Fyodor's support to the open source community.

### Flash review

* in-depth coverage of all Nmap commands, their internals and use cases

* low-level networking internals, comprehensive overview of how Nmap operates in the network environment itself

* introduction to Nmap code and scripting

* the use of the GUI is not neglected

* some dated content, but the updated information is available on the site

Overall, an excellent book that opened my eyes to the extent one can use Nmap for penetration testing..in fact, with all the scripts that are 
available, you can even conduct web app security checks just with Nmap! 

<a href="https://www.amazon.com/Nmap-Network-Scanning-Official-Discovery/dp/0979958717/ref=as_li_ss_il?ie=UTF8&linkCode=li2&tag=sensha10-20&linkId=791bdedacea7ac3bd1caea9852082708" target="_blank"><img src="https://images-na.ssl-images-amazon.com/images/I/51BIsSMZqML._SX382_BO1,204,203,200_.jpg"/></a>

