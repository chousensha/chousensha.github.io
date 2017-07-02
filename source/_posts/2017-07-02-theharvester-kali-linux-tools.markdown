---
layout: post
title: "TheHarvester - Kali Linux tools"
date: 2017-07-02 08:32:26 -0400
comments: true
categories: [penetration testing, tools]
keywords: theharvester, theharvester kali, theharvester tutorial, theharvester examples, theharvester github
description: theharvester tutorial
---

**Objective**: you want to perform OSINT recon on a target and aggregate information from different sources. TheHarvester is an e-mail, subdomain and people names harvester written in Python.

Homepage: https://github.com/laramies/theHarvester

<!-- more -->

## theharvester description

> theHarvester is a tool for gathering e-mail accounts, subdomain names, virtual
> hosts, open ports/ banners, and employee names from different public sources
> (search engines, pgp key servers).

> Is a really simple tool, but very effective for the early stages of a penetration
> test or just to know the visibility of your company in the Internet.

> The sources are:

> Passive:

> -google: google search engine  - www.google.com

> -googleCSE: google custom search engine

> -google-profiles: google search engine, specific search for Google profiles

> -bing: microsoft search engine  - www.bing.com

> -bingapi: microsoft search engine, through the API (you need to add your Key in the discovery/bingsearch.py file)

> -dogpile: Dogpile search engine - www.dogpile.com

> -pgp: pgp key server - mit.edu

> -linkedin: google search engine, specific search for Linkedin users

> -vhost: Bing virtual hosts search

> -twitter: twitter accounts related to an specific domain (uses google search)

> -googleplus: users that works in target company (uses google search)

> -yahoo: Yahoo search engine

> -baidu: Baidu search engine

> -shodan: Shodan Computer search engine, will search for ports and banner of the
> discovered hosts  (http://www.shodanhq.com/)


> Active:

> -DNS brute force: this plugin will run a dictionary brute force enumeration

> -DNS reverse lookup: reverse lookup of ip´s discovered in order to find hostnames

> -DNS TDL expansion: TLD dictionary brute force enumeration


> Modules that need API keys to work:

> -googleCSE: You need to create a Google Custom Search engine(CSE), and add your
> Google API key and CSE ID in the plugin (discovery/googleCSE.py)

> -shodan: You need to provide your API key in discovery/shodansearch.py

## theharvester options

``` 
*******************************************************************
*                                                                 *
* | |_| |__   ___    /\  /\__ _ _ ____   _____  ___| |_ ___ _ __  *
* | __| '_ \ / _ \  / /_/ / _` | '__\ \ / / _ \/ __| __/ _ \ '__| *
* | |_| | | |  __/ / __  / (_| | |   \ V /  __/\__ \ ||  __/ |    *
*  \__|_| |_|\___| \/ /_/ \__,_|_|    \_/ \___||___/\__\___|_|    *
*                                                                 *
* TheHarvester Ver. 2.7                                           *
* Coded by Christian Martorella                                   *
* Edge-Security Research                                          *
* cmartorella@edge-security.com                                   *
*******************************************************************


Usage: theharvester options 

       -d: Domain to search or company name
       -b: data source: google, googleCSE, bing, bingapi, pgp, linkedin,
                        google-profiles, jigsaw, twitter, googleplus, all

       -s: Start in result number X (default: 0)
       -v: Verify host name via dns resolution and search for virtual hosts
       -f: Save the results into an HTML and XML file (both)
       -n: Perform a DNS reverse query on all ranges discovered
       -c: Perform a DNS brute force for the domain name
       -t: Perform a DNS TLD expansion discovery
       -e: Use this DNS server
       -l: Limit the number of results to work with(bing goes from 50 to 50 results,
            google 100 to 100, and pgp doesn't use this option)
       -h: use SHODAN database to query discovered hosts

Examples:
        theharvester -d microsoft.com -l 500 -b google -h myresults.html
        theharvester -d microsoft.com -b pgp
        theharvester -d microsoft -l 200 -b linkedin
        theharvester -d apple.com -b googleCSE -l 500 -s 300
```

## theharvester usage

* search with all options for a given domain

``` 
theharvester -d hackerone.com -l 200 -b all -f harvested

*******************************************************************
*                                                                 *
* | |_| |__   ___    /\  /\__ _ _ ____   _____  ___| |_ ___ _ __  *
* | __| '_ \ / _ \  / /_/ / _` | '__\ \ / / _ \/ __| __/ _ \ '__| *
* | |_| | | |  __/ / __  / (_| | |   \ V /  __/\__ \ ||  __/ |    *
*  \__|_| |_|\___| \/ /_/ \__,_|_|    \_/ \___||___/\__\___|_|    *
*                                                                 *
* TheHarvester Ver. 2.7                                           *
* Coded by Christian Martorella                                   *
* Edge-Security Research                                          *
* cmartorella@edge-security.com                                   *
*******************************************************************


Full harvest..
[-] Searching in Google..
	Searching 0 results...
	Searching 100 results...
	Searching 200 results...
[-] Searching in PGP Key server..
[-] Searching in Bing..
	Searching 50 results...
	Searching 100 results...
	Searching 150 results...
	Searching 200 results...
[-] Searching in Exalead..
	Searching 50 results...
	Searching 100 results...
	Searching 150 results...
	Searching 200 results...
	Searching 250 results...


[+] Emails found:
------------------
feedback@hackerone.com
support@hackerone.com
[REDACTED}

[+] Hosts found in search engines:
------------------------------------
[-] Resolving hostnames IPs... 
162.159.0.31:a.ns.hackerone.com
104.16.13.26:support.hackerone.com
104.16.99.52:www.hackerone.com
[+] Virtual hosts:
==================
104.16.13.26	support.hackerone
104.16.13.26	support.hackerone.com
104.16.99.52	hackerone
[+] Saving files...
Files saved!
```

Keep in mind that I'm editing some results from the output for privacy reasons. I'm also saving all results in HTML

* Linkedin enumeration

``` 
theharvester -d cisco.com -b linkedin -l 100 -f cisco

*******************************************************************
*                                                                 *
* | |_| |__   ___    /\  /\__ _ _ ____   _____  ___| |_ ___ _ __  *
* | __| '_ \ / _ \  / /_/ / _` | '__\ \ / / _ \/ __| __/ _ \ '__| *
* | |_| | | |  __/ / __  / (_| | |   \ V /  __/\__ \ ||  __/ |    *
*  \__|_| |_|\___| \/ /_/ \__,_|_|    \_/ \___||___/\__\___|_|    *
*                                                                 *
* TheHarvester Ver. 2.7                                           *
* Coded by Christian Martorella                                   *
* Edge-Security Research                                          *
* cmartorella@edge-security.com                                   *
*******************************************************************


[-] Searching in Linkedin..
	Searching 100 results..
Users from Linkedin:
====================
Name
Another name
Names 
Names everywhere
```

* Twitter enumeration

``` 
theharvester -d chousensha.github.io -l 10 -b twitter

*******************************************************************
*                                                                 *
* | |_| |__   ___    /\  /\__ _ _ ____   _____  ___| |_ ___ _ __  *
* | __| '_ \ / _ \  / /_/ / _` | '__\ \ / / _ \/ __| __/ _ \ '__| *
* | |_| | | |  __/ / __  / (_| | |   \ V /  __/\__ \ ||  __/ |    *
*  \__|_| |_|\___| \/ /_/ \__,_|_|    \_/ \___||___/\__\___|_|    *
*                                                                 *
* TheHarvester Ver. 2.7                                           *
* Coded by Christian Martorella                                   *
* Edge-Security Research                                          *
* cmartorella@edge-security.com                                   *
*******************************************************************


[-] Searching in Twitter ..
	Searching 100 results..
Users from Twitter:
====================
@-moz-keyframes gb__a
@keyframes gb__a
@-moz-keyframes gb__nb
@keyframes gb__nb
@media 
@keyframes qli-container-rotate 
@keyframes qli-fill-unfill-rotate 
@keyframes qli-blue-fade-in-out 
@keyframes qli-red-fade-in-out 
@keyframes qli-yellow-fade-in-out 
@keyframes qli-green-fade-in-out 
@keyframes qli-left-spin 
@keyframes qli-right-spin 
@kalilinux load balancer detector ...
@chous3nsha
@
```

* sample HTML output

{% img center /images/tools/harvester.png 'theharvester' 'theharvester HTML report' %}

Also note that doing a reverse query on the discovered ranges takes long and the output might be overwhelming.

``` 
 _________________________________________
/ In India, "cold weather" is merely a    \
| conventional phrase and has come into   |
| use through the necessity of having     |
| some way to distinguish between weather |
| which will melt a brass door-knob and   |
| weather which will only make it mushy.  |
|                                         |
\ -- Mark Twain                           /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


