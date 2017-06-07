---
layout: post
title: "lbd - Kali Linux tools"
date: 2017-06-07 05:46:27 -0400
comments: true
categories: [penetration testing, tools]
keywords: lbd, lbd tutorial, lbd kali, lbd kali linux, load balancing detector, lbd usage, kali lbd, load balancing
description: Load balancing detector tutorial
---

Load balancing is the practice of distributing traffic across multiple servers, in order to increase performance and reliability. With multiple servers offering the same resources, single points of failure are eliminated, and availability is increased. Load balancers may be set up in a way that users from certain geographic locations are sent to specific servers, in order to increase the speed of access.

Load balancing introduces some issue in penetration tests, because it interferes with the accuracy of the testing. This is why it's important to establish whether load balancers are in place, and if there are, taking that into account when performing the tests and writing the reports.

<!-- more -->

lbd (load balancing detector) is a Kali tool that is useful for determining the presence of load balancing.

Author: Stefan Behte

## lbd description

> lbd (load balancing detector) detects if a given domain uses DNS and/or HTTP Load-Balancing (via Server: and Date: 
> header and diffs between server answers).

### DNS load balancing

In DNS load balancing, a system has a list of IPs that can respond to requests. When you request a resource, you hit on one of these IPs, and you need to test further to identify the exact target. If your target is *example.com*, and 3 IPs are serving that, when you find a vulnerability, you still have to determine which of these addresses is the vulnerable one (or if all are).

### HTTP load balancing

One of the ways HTTP load balancing can be achieved is through cookies. This comes in handy in online stores and other such web applications that need to identify a client and send it to the same specific resource

## lbd options

``` 
lbd - load balancing detector 0.4 - Checks if a given domain uses load-balancing.
                                    Written by Stefan Behte (http://ge.mine.nu)
                                    Proof-of-concept! Might give false positives.
usage: /usr/bin/lbd domain [port] {https}
```

## lbd usage

Let's now check a bunch of domains and see what load balancers we can find, if at all

* DNS and HTTP load balancing

``` 
lbd hackerone.com

lbd - load balancing detector 0.4 - Checks if a given domain uses load-balancing.
                                    Written by Stefan Behte (http://ge.mine.nu)
                                    Proof-of-concept! Might give false positives.

Checking for DNS-Loadbalancing: FOUND
hackerone.com has address 104.16.100.52
hackerone.com has address 104.16.99.52

Checking for HTTP-Loadbalancing [Server]: 
 cloudflare-nginx
 NOT FOUND

Checking for HTTP-Loadbalancing [Date]: 11:11:37, 11:11:37, 11:11:37, 11:11:37, 11:11:37, 11:11:37, 11:11:37, 11:11:37, 11:11:37, 11:11:37, 11:11:37, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:38, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:39, 11:11:40, 11:11:40, 11:11:40, 11:11:40, 11:11:40, 11:11:40, 11:11:40, 11:11:40, 11:11:40, NOT FOUND

Checking for HTTP-Loadbalancing [Diff]: FOUND
< CF-RAY: 36b32c07c6bf2950-OTP
> CF-RAY: 36b32c0835d7292c-OTP

hackerone.com does Load-balancing. Found via Methods: DNS HTTP[Diff]
```

* HTTP load balancing

``` 
lbd cisco.com

lbd - load balancing detector 0.4 - Checks if a given domain uses load-balancing.
                                    Written by Stefan Behte (http://ge.mine.nu)
                                    Proof-of-concept! Might give false positives.

Checking for DNS-Loadbalancing: NOT FOUND
Checking for HTTP-Loadbalancing [Server]: 
 Apache
 NOT FOUND

Checking for HTTP-Loadbalancing [Date]: 11:13:47, 11:13:47, 11:13:48, 11:13:49, 11:13:50, 11:13:51, 11:13:51, 11:13:52, 11:13:53, 11:13:54, 11:13:54, 11:13:55, 11:13:56, 11:13:57, 11:13:57, 11:13:58, 11:13:59, 11:14:00, 11:14:01, 11:14:01, 11:14:02, 11:14:03, 11:14:04, 11:14:04, 11:14:05, 11:14:06, 11:14:07, 11:14:07, 11:14:08, 11:14:09, 11:14:10, 11:14:11, 11:14:11, 11:14:12, 11:14:13, 11:14:14, 11:14:14, 11:14:15, 11:14:16, 11:14:17, 11:14:17, 11:14:18, 11:14:19, 11:14:20, 11:14:21, 11:14:21, 11:14:22, 11:14:23, 11:14:24, 11:14:24, NOT FOUND

Checking for HTTP-Loadbalancing [Diff]: FOUND
> Cache-Control: max-age=0
> Expires: Wed, 07 Jun 2017 11:14:27 GMT

cisco.com does Load-balancing. Found via Methods: HTTP[Diff]
```

* no load balancing

``` 
lbd nmap.org

lbd - load balancing detector 0.4 - Checks if a given domain uses load-balancing.
                                    Written by Stefan Behte (http://ge.mine.nu)
                                    Proof-of-concept! Might give false positives.

Checking for DNS-Loadbalancing: NOT FOUND
Checking for HTTP-Loadbalancing [Server]: 
 Apache/2.4.6 (CentOS)
 NOT FOUND

Checking for HTTP-Loadbalancing [Date]: 11:17:00, 11:17:00, 11:17:01, 11:17:01, 11:17:02, 11:17:02, 11:17:03, 11:17:03, 11:17:03, 11:17:04, 11:17:04, 11:17:05, 11:17:05, 11:17:06, 11:17:06, 11:17:07, 11:17:07, 11:17:07, 11:17:08, 11:17:08, 11:17:09, 11:17:09, 11:17:10, 11:17:10, 11:17:10, 11:17:11, 11:17:11, 11:17:12, 11:17:12, 11:17:13, 11:17:13, 11:17:14, 11:17:14, 11:17:14, 11:17:15, 11:17:15, 11:17:16, 11:17:16, 11:17:17, 11:17:17, 11:17:17, 11:17:18, 11:17:18, 11:17:19, 11:17:19, 11:17:20, 11:17:20, 11:17:20, 11:17:21, 11:17:21, NOT FOUND

Checking for HTTP-Loadbalancing [Diff]: NOT FOUND

nmap.org does NOT use Load-balancing.
```

### Key takeaways

* when testing load balanced systems, you can try accessing them by IP instead of name (be advised that firewalls might pick up on this as suspicious activity)

To learn more about load balancing and pentesting, check out this [SANS paper](https://www.sans.org/reading-room/whitepapers/testing/identifying-load-balancers-penetration-testing-33313)

``` 
/ Q: What is printed on the bottom of \
| beer bottles in Minnesota? A: Open  |
\ other end.                          /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

