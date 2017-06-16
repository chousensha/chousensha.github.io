---
layout: post
title: "Netdiscover - Kali Linux tools"
date: 2017-06-16 13:05:44 -0400
comments: true
categories: [penetration testing, tools]
keywords: netdiscover, netdiscover kali, netdiscover tutorial, netdiscover command, netdiscover linux
description: netdiscover tutorial
---

**Objective**: a tool that can be used to perform ARP reconaissance and discover hosts on the local network. You could do that with Nmap too, but here netdiscover shines!

<!-- more -->

Homepage: https://github.com/alexxy/netdiscover

## Netdiscover description

> netdiscover is an  active/passive  arp  reconnaissance  tool,  initialy
> developed  to  gain  information  about  wireless networks without dhcp
> servers in wardriving scenarios.  It  can  also  be  used  on  switched
> networks.  Built  on top of libnet and libpcap, it can passively detect
> online hosts or search for them by sending arp requests.

> Furthermore, it can be used to inspect your network's arp  traffic,  or
> find network addresses using auto scan mode, which will scan for common
> local networks.

Manpage: http://manpages.ubuntu.com/manpages/precise/man8/netdiscover.8.html

## Netdiscover options

``` 
Netdiscover 0.3-pre-beta7 [Active/passive arp reconnaissance tool]
Written by: Jaime Penalba <jpenalbae@gmail.com>

Usage: netdiscover [-i device] [-r range | -l file | -p] [-m file] [-s time] [-n node] [-c count] [-f] [-d] [-S] [-P] [-c]
  -i device: your network device
  -r range: scan a given range instead of auto scan. 192.168.6.0/24,/16,/8
  -l file: scan the list of ranges contained into the given file
  -p passive mode: do not send anything, only sniff
  -m file: scan the list of known MACs and host names
  -F filter: Customize pcap filter expression (default: "arp")
  -s time: time to sleep between each arp request (milliseconds)
  -n node: last ip octet used for scanning (from 2 to 253)
  -c count: number of times to send each arp reques (for nets with packet loss)
  -f enable fastmode scan, saves a lot of time, recommended for auto
  -d ignore home config files for autoscan and fast mode
  -S enable sleep time supression between each request (hardcore mode)
  -P print results in a format suitable for parsing by another program
  -N Do not print header. Only valid when -P is enabled.
  -L in parsable output mode (-P), continue listening after the active scan is completed

If -r, -l or -p are not enabled, netdiscover will scan for common lan addresses.
```

## Netdiscover usage


Simply typing netdiscover at the terminal will launch its autoscan. The screen is interactive, and you can see new hosts as they appear.

* scan range

``` 
netdiscover -r 192.168.217.0/24
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                      
                                                                                                                                                    
 3 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 180                                                                                    
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.217.1   00:50:56:c0:00:08      1      60  Unknown vendor                                                                                   
 192.168.217.2   00:50:56:fc:f6:8b      1      60  Unknown vendor                                                                                   
 192.168.217.254 00:50:56:f2:e2:e6      1      60  Unknown vendor     
```

* passive scan (don't send anything, just sniff)

``` 
netdiscover -p
 Currently scanning: (passive)   |   Screen View: Unique Hosts                                                                                      
                                                                                                                                                    
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 120                                                                                    
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.217.2   00:50:56:fc:f6:8b      1      60  Unknown vendor                                                                                   
 192.168.217.138 00:0c:29:43:d4:3e      1      60  Unknown vendor      
```

This takes longer, because netdiscover is waiting to see the ARP requests and replies between other hosts

* fast scan

> -f     Enable  fast  mode  scan. This will only scan for .1, .100 and .254 on each network. This mode is useful while
> searching for ranges being used. After you found such range you can make a specific range scan to find online boxes.

* produce parseable output and stop after scanning

``` 
root@kali:~# netdiscover -r 192.168.217.0/24 -P
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.217.1   00:50:56:c0:00:08      1      60  Unknown vendor
 192.168.217.2   00:50:56:fc:f6:8b      1      60  Unknown vendor
 192.168.217.138 00:0c:29:43:d4:3e      1      60  Unknown vendor
 192.168.217.254 00:50:56:f2:e2:e6      1      60  Unknown vendor

-- Active scan completed, 4 Hosts found.
```

Cookie:

``` 
_____________________________________
/ F.S. Fitzgerald to Hemingway:        \
|                                      |
| "Ernest, the rich are different from |
| us." Hemingway:                      |
|                                      |
\ "Yes. They have more money."         /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

