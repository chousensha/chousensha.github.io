---
layout: post
title: "dnsmap - Kali Linux tools"
date: 2017-06-27 04:12:46 -0400
comments: true
categories: [penetration testing, tools]
keywords: dnsmap, dnsmap tutorial, dnsmap kali, dnsmap kali linux, dnsmap tool, dns, dnsmap usage, kali dnsmap, dns network mapper
description: dnsmap tutorial
---

Today we'll explore another tool for DNS enumeration: the DNS Network Mapper (dnsmap). Although at the time of this post, its last update was in 2010, this tool has survived the passage of time, and has been packaged with versions of both Backtrack and Kali. There is quite a wealth of information about the tool on its homepage, and it comes with a built-in wordlist for domain bruteforcing.

Homepage: https://github.com/makefu/dnsmap

<!-- more -->

## dnsmap description

> dnsmap was originally released back in 2006 and was inspired by the
> fictional story "The Thief No One Saw" by Paul Craig, which can be found
> in the book "Stealing the Network - How to 0wn the Box"

> dnsmap is mainly meant to be used by pentesters during the information
> gathering/enumeration phase of infrastructure security assessments. During the
> enumeration stage, the security consultant would typically discover the target
> company's IP netblocks, domain names, phone numbers, etc ...

> Subdomain brute-forcing is another technique that should be used in the
> enumeration stage, as it's especially useful when other domain enumeration
> techniques such as zone transfers don't work (I rarely see zone transfers
> being *publicly* allowed these days by the way).

> LIMITATIONS

> Lack of multi-threading. This speed issue will hopefully be resolved in future versions.


> FUN THINGS THAT CAN HAPPEN

> 1. Finding interesting remote access servers (e.g.: https://extranet.targetdomain.com)

> 2. Finding badly configured and/or unpatched servers (e.g.: test.targetdomain.com)

> 3. Finding new domain names which will allow you to map non-obvious/hard-to-find netblocks
>   of your target organization (registry lookups - aka whois is your friend)

> 4. Sometimes you find that some bruteforced subdomains resolve to internal IP addresses
>   (RFC 1918). This is great as sometimes they are real up-to-date "A" records which means
>   that it *is* possible to enumerate internal servers of a target organization from the
>   Internet by only using standard DNS resolving (as oppossed to zone transfers for instance).

> 5. Discover embedded devices configured using Dynamic DNS services (e.g.: linksys-cam.com).
>   This method is an alternative to finding devices via Google hacking techniques

> USAGE

> Bruteforcing can be done either with dnsmap's built-in wordlist or a user-supplied wordlist.
> Results can be saved in CSV and human-readable format for further processing. dnsmap does
> NOT require root privileges to be run, and should NOT be run with such privileges for security reasons.

## dnsmap options

``` 
dnsmap 0.30 - DNS Network Mapper by pagvac (gnucitizen.org)

usage: dnsmap <target-domain> [options]
options:
-w <wordlist-file>
-r <regular-results-file>
-c <csv-results-file>
-d <delay-millisecs>
-i <ips-to-ignore> (useful if you're obtaining false positives)

e.g.:
dnsmap target-domain.foo
dnsmap target-domain.foo -w yourwordlist.txt -r /tmp/domainbf_results.txt
dnsmap target-fomain.foo -r /tmp/ -d 3000
dnsmap target-fomain.foo -r ./domainbf_results.txt
```

## dnsmap usage

``` 
dnsmap slack.com
dnsmap 0.30 - DNS Network Mapper by pagvac (gnucitizen.org)

[+] warning: domain might use wildcards. 54.230.203.126 will be ignored from results
[+] searching (sub)domains for slack.com using built-in wordlist
[+] using maximum random delay of 10 millisecond(s) between requests

email.slack.com
IP address #1: 34.196.74.192
IP address #2: 54.88.163.82

files.slack.com
IP address #1: 52.85.178.60

qa.slack.com
IP address #1: 54.192.201.191

staging.slack.com
IP address #1: 52.87.251.252

upload.slack.com
IP address #1: 34.197.50.42
IP address #2: 52.22.100.1
IP address #3: 52.44.55.102
IP address #4: 52.54.239.215

[+] 5 (sub)domains and 9 IP address(es) found
[+] completion time: 122 second(s)
```

Here is how the output looks in the CSV format:

{% img center /images/tools/dnsmap.png 'dnsmap' 'dnsmap CSV report' %}

``` 
 _____________________________________
/ In the stairway of life, you'd best \
\ take the elevator.                  /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
