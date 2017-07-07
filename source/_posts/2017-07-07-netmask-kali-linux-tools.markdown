---
layout: post
title: "netmask - Kali Linux tools"
date: 2017-07-07 05:55:22 -0400
comments: true
categories: [penetration testing, tools]
keywords: netmask, netmask calculator, netmask kali, netmask converter, kali linux netmask
description: netmask tutorial
---

**Objective**: you want to convert between different types of netmasks and network addresses, or generate optimized netmasks for firewall rules. netmask can take care of your netmasks!

Homepage: https://github.com/tlby/netmask

<!-- more -->

## netmask description

> This  program  accepts and produces a variety of common network address
> and netmask formats.  Not only can it convert address and netmask notations,  but it will optimize the masks to 
> generate the smallest list of rules.  This is very handy if you've  ever  configured  a  firewall  or
> router  and  some  nasty  network administrator before you decided that
> base 10 numbers were good places to start and end groups of machines.

Manpage: https://linux.die.net/man/1/netmask

## netmask options

``` 
This is netmask, an address netmask generation utility
Usage: netmask spec [spec ...]
  -h, --help			Print a summary of the options
  -v, --version			Print the version number
  -d, --debug			Print status/progress information
  -s, --standard		Output address/netmask pairs
  -c, --cidr			Output CIDR format address lists
  -i, --cisco			Output Cisco style address lists
  -r, --range			Output ip address ranges
  -x, --hex			Output address/netmask pairs in hex
  -o, --octal			Output address/netmask pairs in octal
  -b, --binary			Output address/netmask pairs in binary
  -n, --nodns			Disable DNS lookups for addresses
  -f, --files			Treat arguments as input files
Definitions:
  a spec can be any of:
    address
    address:address
    address:+address
    address/mask
  an address can be any of:
    N		decimal number
    0N		octal number
    0xN		hex number
    N.N.N.N	dotted quad
    hostname	dns domain name
  a mask is the number of bits set to one from the left
```

## netmask usage

* output standard address / netmask pairs

``` 
root@kali:~# netmask -s 192.168.217.0/24
  192.168.217.0/255.255.255.0  
```

* CIDR notation

``` 
root@kali:~# netmask -c google.com
  216.58.210.14/32
2a00:1450:4001:81d::200e/128
```

* Cisco-style notation

``` 
root@kali:~# netmask -i google.com
  216.58.210.14 0.0.0.0        
2a00:1450:4001:81d::200e ::   
```

* IP ranges

``` 
root@kali:~# netmask -r 192.168.217.0/24
  192.168.217.0-192.168.217.255 (256)
```

``` 
 ________________________________________
< Your ignorance cramps my conversation. >
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
