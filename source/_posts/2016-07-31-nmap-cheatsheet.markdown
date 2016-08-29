---
layout: post
title: "Nmap cheatsheet"
date: 2016-07-31 06:40:43 -0400
comments: true
categories: [penetration testing, networking, tools]
keywords: pentesting, nmap, hacking, penetration testing, nmap tutorial, nmap guide, port scan, scanner, port scanner, nmap cheatsheet, port scanning, exploit, vulnerability, security audit
description: Nmap 101
---

Recently, I've been reading Fyodor's [Nmap Network Scanning](https://www.amazon.com/Nmap-Network-Scanning-Official-Discovery/dp/0979958717) book and decided to create an Nmap cheatsheet while at it.

Half of the book, including updated content, is also available for free on https://nmap.org/book/toc.html

Fyodor was also nice enough to set up a machine that you can test your scans against (within reasonable limits), at http://scanme.nmap.org/

All right, let's get scanning!

<!-- more -->

# Commands cheatsheet

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

### Host discovery

**List scan - sL** - enumerates the addresses in the specified range and does reverse DNS lookup on them. Good for getting a feel about the targets by checking their DNS names, and maintaining a low profile on the network (because no ping requests are being sent yet, so it's stealthier)

**-p-**: Shortcut for *-p1-65535* 

**-Pn**: Treat all hosts as online -- skip host discovery (disable ping)

**--exclude <host1[,host2][,host3],...>**: Exclude hosts/networks

**-n**: Never do DNS resolution

**-sP**: perform a ping sweep

**-PS/PA/PU**: TCP SYN ping/TCP ACK ping/UDP ping - various host discovery options for bypassing firewalls

### Performance

**-T<0-5>**: Set timing template (higher is faster). -T4 (aggressive) is recommended for most cases, as it improves performance and optimization for a fast and reliable network. For evasion purposes you can use T0 (paranoid) or T1 (sneaky), but speed will be very slow. To conserve bandwidth and resources you may use T2 (polite), at another significant expense of speed. T3 is the normal mode, and T5 is an insane level of speed (but you might sacrifice some accuracy in the process)

**-F**: Fast scan, only scan most common 100 ports

### Evasion

**-D <decoy1,decoy2[,ME],...>**: Cloak a scan with decoys

**--scanflags <flags>**: Customize TCP scan flags. You can couple it with specifying a TCP scan type to give Nmap a base of interpreting the results

**-S <IP_Address>**: Spoof source address

**-sI <zombie host[:probeport]>**: Idle scan, a complex scan that is stealthy but slow. It uses zombies to perform the scan, without sending packets directly from your IP. Use in tandem with the -PN option to disable ping and not disclose your real address. Avoid -sV, as version detection would also make your IP visible to the target

**-f; --mtu <val>**: fragment packets (optionally w/given MTU), can aid in bypassing firewalls that don't handle fragmentation well

**-g/--source-port <portnum>**: Spoof source port, bypass security measures that rely only on checking the source port

**--spoof-mac <mac address/prefix/vendor name>**: Spoof your MAC address

**--badsum**: Send packets with a bogus TCP/UDP/SCTP checksum. Normal hosts discard packets with bad checksums, but firewalls may omit these checks for performance reasons

**--dns-servers <serv1[,serv2],...>**: Specify custom DNS servers when doing reverse DNS lookup to avoid your DNS server showing up in the target's logs

### Additional info

**-v**: increase verbosity level, you can use it 2 or 3 times to further increase verbosity

**-d**: Increase debugging level (use -dd or more for greater effect), can increase to a max of 9

**--packet-trace**: print summary of every packet sent and received

### Output

**-oN/-oX/-oS/-oG <file>**: Output scan in normal, XML, sCrIpt kIddi3, and Grepable format, respectively, to the given filename

**-oA <basename>**: Output in the three major formats at once

**--resume <filename>**: Resume an aborted scan

If you don't want Nmap to display its interactive output, you can pass a *-* to the format type(s), like so: <code>-oS -</code>

## Useful tips

* <code>nmap -sL -n target</code> - list IPs to be scanned before actually scanning them

* <code>-pT:22,80,U:53</code> - scan separate TCP and UDP ports (you have to specify a UDP scan in addition to a TCP scan)

* <code>--version-intensity <level></code>: Set from 0 (light) to 9 (try all probes), setting it to 0 will limit the version detection to the most likely to succeed probes, and improve the speed of the scan

- you can scan a network range by adding the subnet mask after an IP like so: *ip/numbits*. An example would be: 192.168.80.0/24, where the hosts between 192.168.80.0 and 192.168.80.255 would be scanned

- version detection is noisy

- you can specify port 0 for a scan, to catch malicious software that might listen on it


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

A full list of scripts and their description and usage can be found at https://nmap.org/nsedoc/

Some notable scripts that you might use more often are listed below.

* dns-zone-transfer - requests a zone transfer from a DNS server

* ftp-anon - checks if anonymous login is enabled on an FTP server and lists the root directory if yes

* ftp-bounce - checks if FTP server allows port scanning via the bounce method

* http-auth-finder - spiders a site to find pages containing form or HTTP authentication

* http-backup-finder - spiders a site to find backups of discovered files

* http-config-backup - searches for backups of web server and CMS configuration files

* http-csrf - checks for forms vulnerable to CSRF

* http-default-accounts - checks for default credentials in web applications, routers, and VOIP and security devices

* http-devframework - checks for the underlying framework of a site

* http-dlink-backdoor - checks for a firmware backdoor that is found in some D-Link routers

* http-dombased-xss - searches for places susceptible to DOM-based XSS

* http-drupal-enum-users - enumerates Drupal users

* http-enum - enumerates common directories found in web apps

* http-fileupload-exploiter - exploits file upload vulnerabilities

* http-mobileversion-checker - checks if the site has a mobile version

* http-open-redirect - checks for open redirects

* http-passwd - checks for directory traversal

* http-phpself-xss - finds PHP_SELF XSS vulnerabilities

* http-rfi-spider - searches for RFI in forms and URL parameters

* http-shellshock - checks if web app is vulnerable to shellshock

* http-sitemap-generator - crawls a site and lists its directory structure and files

* http-sql-injection - looks for SQLi in forms and URL parameters of HTTP servers

* http-stored-xss - searches for unfiltered input that might lead to XSS

* http-tplink-dir-traversal - on certain TPLINK routers that are vulnerable to directory traversal, it can read remote files with no authentication required

* http-unsafe-output-escaping - crawls a site looking for where improperly escaped data is reflected back to the user 

* http-vuln-cve2012-1823 - attempts to exploit vulnerable PHP CGI apps that allow remote code execution

* http-vuln-cve2015-1635 - checks for the MS15-034 RCE vulnerability in Windows machines

* http-waf-detect - tries to detect IDS / IPS / WAF protection on the target

* http-waf-fingerprint - attempts to fingerpring the WAF

* http-webdav-scan - WebDAV detection

* ipidseq - tests for suitable zombies to be used in idle scan

* mongodb-databases - enumerates MongoDB tables

* ms-sql-empty-password - tries to login as the sa account with no password on MS-SQL

* mysql-empty-password - checks for MySQL servers with empty password for the root or anonymous account

* samba-vuln-cve-2012-1182 - checks if Samba server is vulnerable to CVE-2012-1182 heap overflow

* shodan-api - uses Shodan (API key needed) to scan targets in a way similar to the -sV scan

* sniffer-detect - checks if a NIC on the local network is in promiscuous mode

* ssl-heartbleed - checks for the Heartbleed vulnerability

* ssl-poodle - checks for the Poodle vulnerability

* tor-consensus-checker - checks if target host is a Tor node

* traceroute-geolocation - performs geolocation on traceroute hops

Many thanks to Fyodor for creating such an awesome tool and freely sharing with everyone! As you can see, Nmap is so much more than just a port scanner. You can easily perform some sort of vulnerability analysis and exploitation with it, and if there is no task that you would like Nmap to do and it can't, you can write the code yourself and contribute to this amazing project.

``` plain
/ You have the capacity to learn from \
\ mistakes. You'll learn a lot today. /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

