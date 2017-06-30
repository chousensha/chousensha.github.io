---
layout: post
title: "wafw00f - Kali Linux tools"
date: 2017-06-30 04:46:55 -0400
comments: true
categories: [penetration testing, tools]
keywords: wafw00f, wafw00f kali linux, wafw00f kali, wafw00f tutorial
description: wafw00f tutorial
---

**Objective**: you need to establish if a web application firewall is in place, and ideally, what type of WAF it is. You can go bark at it with wafw00f!

Homepage: https://github.com/EnableSecurity/wafw00f

<!-- more -->

## wafw00f description

> WAFW00F identifies and fingerprints Web Application Firewall (WAF) products.

> How does it work?

> To do its magic, WAFW00F does the following:

>  Sends a normal HTTP request and analyses the response; this identifies a number of WAF solutions

>  If that is not successful, it sends a number of (potentially malicious) HTTP requests and uses simple logic to 
> deduce which WAF it is

>  If that is also not successful, it analyses the responses previously returned and uses another simple algorithm 
> to guess if a WAF or security solution is actively responding to our attacks

## wafw00f options

``` 
wafw00f -h

                                 ^     ^
        _   __  _   ____ _   __  _    _   ____
       ///7/ /.' \ / __////7/ /,' \ ,' \ / __/
      | V V // o // _/ | V V // 0 // 0 // _/
      |_n_,'/_n_//_/   |_n_,' \_,' \_,'/_/
                                <
                                 ...'

    WAFW00F - Web Application Firewall Detection Tool

    By Sandro Gauci && Wendel G. Henrique

Usage: wafw00f url1 [url2 [url3 ... ]]
example: wafw00f http://www.victim.org/

Options:
  -h, --help            show this help message and exit
  -v, --verbose         enable verbosity - multiple -v options increase
                        verbosity
  -a, --findall         Find all WAFs, do not stop testing on the first one
  -r, --disableredirect
                        Do not follow redirections given by 3xx responses
  -t TEST, --test=TEST  Test for one specific WAF
  -l, --list            List all WAFs that we are able to detect
  --xmlrpc              Switch on the XML-RPC interface instead of CUI
  --xmlrpcport=XMLRPCPORT
                        Specify an alternative port to listen on, default 8001
  -V, --version         Print out the version
```

## wafw00f usage

* see what WAFs it can detect at the moment

``` 
wafw00f -l

                                 ^     ^
        _   __  _   ____ _   __  _    _   ____
       ///7/ /.' \ / __////7/ /,' \ ,' \ / __/
      | V V // o // _/ | V V // 0 // 0 // _/
      |_n_,'/_n_//_/   |_n_,' \_,' \_,'/_/
                                <
                                 ...'

    WAFW00F - Web Application Firewall Detection Tool

    By Sandro Gauci && Wendel G. Henrique

Can test for these WAFs:

Profense
NetContinuum
Incapsula WAF
CloudFlare
USP Secure Entry Server
Cisco ACE XML Gateway
Barracuda Application Firewall
Art of Defence HyperGuard
BinarySec
Teros WAF
F5 BIG-IP LTM
F5 BIG-IP APM
F5 BIG-IP ASM
F5 FirePass
F5 Trafficshield
InfoGuard Airlock
Citrix NetScaler
Trustwave ModSecurity
IBM Web Application Security
IBM DataPower
DenyALL WAF
Applicure dotDefender
Juniper WebApp Secure
Microsoft URLScan
Aqtronix WebKnight
eEye Digital Security SecureIIS
Imperva SecureSphere
Microsoft ISA Server
```

* detect WAF

``` 
wafw00f https://bugcrowd.com

                                 ^     ^
        _   __  _   ____ _   __  _    _   ____
       ///7/ /.' \ / __////7/ /,' \ ,' \ / __/
      | V V // o // _/ | V V // 0 // 0 // _/
      |_n_,'/_n_//_/   |_n_,' \_,' \_,'/_/
                                <
                                 ...'

    WAFW00F - Web Application Firewall Detection Tool

    By Sandro Gauci && Wendel G. Henrique

Checking https://bugcrowd.com
The site https://bugcrowd.com is behind a CloudFlare
Number of requests: 1
```

* not detecting WAF

``` 
wafw00f cisco.com

                                 ^     ^
        _   __  _   ____ _   __  _    _   ____
       ///7/ /.' \ / __////7/ /,' \ ,' \ / __/
      | V V // o // _/ | V V // 0 // 0 // _/
      |_n_,'/_n_//_/   |_n_,' \_,' \_,'/_/
                                <
                                 ...'

    WAFW00F - Web Application Firewall Detection Tool

    By Sandro Gauci && Wendel G. Henrique

Checking http://cisco.com
Generic Detection results:
No WAF detected by the generic detection
Number of requests: 13
```

* establish there is a WAF but without detecting its type

``` 
wafw00f -a microsoft.com

                                 ^     ^
        _   __  _   ____ _   __  _    _   ____
       ///7/ /.' \ / __////7/ /,' \ ,' \ / __/
      | V V // o // _/ | V V // 0 // 0 // _/
      |_n_,'/_n_//_/   |_n_,' \_,' \_,'/_/
                                <
                                 ...'

    WAFW00F - Web Application Firewall Detection Tool

    By Sandro Gauci && Wendel G. Henrique

Checking http://microsoft.com
Generic Detection results:
The site http://microsoft.com seems to be behind a WAF or some sort of security solution
Reason: The server returned a different response code when a string trigged the blacklist.
Normal response code is "400", while the response code to an attack is "301"
Number of requests: 13
```

``` 
 _______________________________________
/ You'll wish that you had done some of \
| the hard things when they were easier |
\ to do.                                /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```



