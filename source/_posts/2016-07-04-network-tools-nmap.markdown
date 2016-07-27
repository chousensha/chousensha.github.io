---
layout: post
title: "Network tools - Nmap"
date: 2016-07-04 08:50:02 -0400
comments: true
categories: [pentesting, networking]
keywords: pentesting, nmap, hacking, penetration testing, nmap tutorial, nmap guide, port scan, scanner
description: Nmap 101
published: false
---

I decided to some more in-depth tutorials about the tools that are frequently used in pentesting, and learn a new thing or two in the process :-)

So it's time for our favorite scanner to get the spotlight. Enter Nmap!

Before beginning, I want to recommend learning about Nmap from its author himself:

[Nmap Network Scanning](https://www.amazon.com/Nmap-Network-Scanning-Official-Discovery/dp/0979958717)

Half of the book is also available for free on https://nmap.org/book/toc.html

All right, let's get scanning!

<!-- more -->


# Commands cheatsheet

## Host discovery

**List scan - sL** - enumerates the addresses in the specified range and does reverse DNS lookup on them. Good for getting a feel about the targets by checking their DNS names, and maintaining a low profile on the network (because no ping requests are being sent yet, so it's stealthier)

**-p-**: Shortcut for *-p1-65535* 

**-Pn**: Treat all hosts as online -- skip host discovery (disable ping)

**--exclude <host1[,host2][,host3],...>**: Exclude hosts/networks

**-n**: Never do DNS resolution

**-sP**: perform a ping sweep

**-PS/PA/PU**: TCP SYN ping/TCP ACK ping/UDP ping - various host discovery options for bypassing firewalls

## Performance

**-T<0-5>**: Set timing template (higher is faster). -T4 (aggressive) is recommended for most cases, as it improves performance and optimization for a fast and reliable network. For evasion purposes you can use T0 (paranoid) or T1 (sneaky), but speed will be very slow. To conserve bandwidth and resources you may use T2 (polite), at another significant expense of speed. T3 is the normal mode, and T5 is an insane level of speed (but you might sacrifice some accuracy in the process)

**-F**: Fast scan, only scan most common 100 ports

## Evasion

**-D <decoy1,decoy2[,ME],...>**: Cloak a scan with decoys

**--scanflags <flags>**: Customize TCP scan flags. You can couple it with specifying a TCP scan type to give Nmap a base of interpreting the results

**-S <IP_Address>**: Spoof source address

**-sI <zombie host[:probeport]>**: Idle scan, a complex scan that is stealthy but slow. It uses zombies to perform the scan, without sending packets directly from your IP. Use in tandem with the -PN option to disable ping and not disclose your real address. Avoid -sV, as version detection would also make your IP visible to the target

**-f; --mtu <val>**: fragment packets (optionally w/given MTU), can aid in bypassing firewalls that don't handle fragmentation well

**-g/--source-port <portnum>**: Spoof source port, bypass security measures that rely only on checking the source port

**--spoof-mac <mac address/prefix/vendor name>**: Spoof your MAC address

**--badsum**: Send packets with a bogus TCP/UDP/SCTP checksum. Normal hosts discard packets with bad checksums, but firewalls may omit these checks for performance reasons

**--dns-servers <serv1[,serv2],...>**: Specify custom DNS servers when doing reverse DNS lookup to avoid your DNS server showing up in the target's logs

## Additional info

**-v**: increase verbosity level, you can use it 2 or 3 times to further increase verbosity

**-d**: Increase debugging level (use -dd or more for greater effect), can increase to a max of 9

**--packet-trace**: print summary of every packet sent and received

## Output

**-oN/-oX/-oS/-oG <file>**: Output scan in normal, XML, sCrIpt kIddi3, and Grepable format, respectively, to the given filename

**-oA <basename>**: Output in the three major formats at once

**--resume <filename>**: Resume an aborted scan

If you don't want Nmap to display its interactive output, you can pass a *-* to the format type(s), like so: <code>-oS -</code>

## Useful commands

* <code>nmap -sL -n target</code> - list IPs to be scanned before actually scanning them

* <code>-pT:22,80,U:53</code> - scan separate TCP and UDP ports (you have to specify a UDP scan in addition to a TCP scan)

* <code>--version-intensity <level></code>: Set from 0 (light) to 9 (try all probes), setting it to 0 will limit the version detection to the most likely to succeed probes, and improve the speed of the scan


## CIDR scanning

- you can scan a network range by adding the subnet mask after an IP like so: *ip/numbits*. An example would be: 192.168.80.0/24, where the hosts between 192.168.80.0 and 192.168.80.255 would be scanned

## Scan types

**-sS**: TCP SYN scan - the default scan, most popular due to its speed and stealth, as it doesn't complete a TCP connection

**-sT**: TCP Connect scan, it uses the connect() system call, used by unprivileged users who can't send raw packets or when scanning IPv6 targets

**-sV**: Version detection, omit for performance increase and/or stealth

**-sA**: TCP ACK scan, can't tell if the ports are open or closed, but it can determine whether firewall rules are stateful or stateless

**-sW**: TCP Window scan, same as ACK, but it can determine open and closed ports on certain systems

**-sM**: Maimon scan, yet another special scan type for bypassing firewalls

**-sU**: UDP scan. You can improve it by adding a version scan alongside it (but it will be slower)

**-sF**: FIN scan, sets the FIN bit, useful in bypassing some firewalls and determining the target's OS between Windows and Unix types, as it doesn't work against Windows

**-sX**: Xmas scan, another option for firewall evasion, sets the FIN, PSH, and URG flags, creating the impression of a lit Christmas tree 

**-sN**: Null scan, doesn't set any flags, an additional option to evade pesky firewalls 

**-sO**: IP protocol scan, check which IP protocols are supported on the target

**-b <FTP relay host>**: FTP bounce scan, also available to unprivileged users

**-A**: Enable OS detection, version detection, script scanning, and traceroute


## Script scanning

**-sC**: enable some common scripts, may be intrusive to the target

**--script**=<Lua scripts>: <Lua scripts> is a comma separated list of directories, script-files or script-categories

[Currently defined categories](https://nmap.org/book/nse-usage.html) are:

- *auth* - operations requiring authentication credentials

- *broadcast* - local network broadcasts

- *brute* - brute force attacks

- *default* - scripts that are run by default with -sC or -A options

- *discovery* - additional network information and enumeration

- *dos* - denial of service scripts

- *exploit* - exploitation of some specific vulnerability

- *external* - information may be sent to a third-party database or service

- *fuzzer* - fuzzing scripts

- *intrusive* - may crash the target system, consume excessive resources, or be perceived as malicious

- *malware* - test if the target is infected by malware or backdoors

- *safe* - general network discovery scripts that re unlikely to be resource intensive, crash a system, or lit the target logs on fire

- *version* - scripts that run only with -sV enabled

- *vuln* - check for specific vulnerabilities

- version detection is noisy

- you can specify port 0 for a scan, to catch malicious software that might listen on it

