---
layout: post
title: "dnsrecon - Kali Linux tools"
date: 2017-07-12 14:41:27 -0400
comments: true
categories: [penetration testing, tools]
keywords: dnsrecon, dnsrecon tutorial, dnsrecon kali, dnsrecon zone transfer, dns, dnsrecon usage, kali dnsrecon
description: dnsrecon tutorial
---

**Objective**: perform thorough DNS enumeration and subdomain bruteforcing on a target. dnsrecon is true to its name, it's written in Python, and judging from the number of stars on its Github repository, it's a much more popular choice than the other candidates in Kali's DNS section.

Homepage: https://github.com/darkoperator/dnsrecon

<!-- more -->

## dnsrecon description

> This script provides the ability to perform:

> Check all NS Records for Zone Transfers.

> Enumerate General DNS Records for a given Domain (MX, SOA, NS, A, AAAA, SPF and TXT).

> Perform common SRV Record Enumeration.

> Top Level Domain (TLD) Expansion.

> Check for Wildcard Resolution.

> Brute Force subdomain and host A and AAAA records given a domain and a wordlist.

> Perform a PTR Record lookup for a given IP Range or CIDR.

> Check a DNS Server Cached records for A, AAAA and CNAME Records provided a list of host records in a text file to 
> check.

> Enumerate Common mDNS records in the Local Network

> Enumerate Hosts and Subdomains using Google

Manpage: http://manpages.ubuntu.com/manpages/zesty/man1/dnsrecon.1.html

## dnsrecon options

``` 
Version: 0.8.10
Usage: dnsrecon.py <options>

Options:
   -h, --help                   Show this help message and exit.
   -d, --domain      <domain>   Target domain.
   -r, --range       <range>    IP range for reverse lookup brute force in formats (first-last) or in (range/bitmask).
   -n, --name_server <name>     Domain server to use. If none is given, the SOA of the target will be used.
   -D, --dictionary  <file>     Dictionary file of subdomain and hostnames to use for brute force.
   -f                           Filter out of brute force domain lookup, records that resolve to the wildcard defined
                                IP address when saving records.
   -t, --type        <types>    Type of enumeration to perform:
                                std       SOA, NS, A, AAAA, MX and SRV if AXRF on the NS servers fail.
                                rvl       Reverse lookup of a given CIDR or IP range.
                                brt       Brute force domains and hosts using a given dictionary.
                                srv       SRV records.
                                axfr      Test all NS servers for a zone transfer.
                                goo       Perform Google search for subdomains and hosts.
                                snoop     Perform cache snooping against all NS servers for a given domain, testing
                                          all with file containing the domains, file given with -D option.
                                tld       Remove the TLD of given domain and test against all TLDs registered in IANA.
                                zonewalk  Perform a DNSSEC zone walk using NSEC records.
   -a                           Perform AXFR with standard enumeration.
   -s                           Perform a reverse lookup of IPv4 ranges in the SPF record with standard enumeration.
   -g                           Perform Google enumeration with standard enumeration.
   -w                           Perform deep whois record analysis and reverse lookup of IP ranges found through
                                Whois when doing a standard enumeration.
   -z                           Performs a DNSSEC zone walk with standard enumeration.
   --threads         <number>   Number of threads to use in reverse lookups, forward lookups, brute force and SRV
                                record enumeration.
   --lifetime        <number>   Time to wait for a server to response to a query.
   --db              <file>     SQLite 3 file to save found records.
   --xml             <file>     XML file to save found records.
   --iw                         Continue brute forcing a domain even if a wildcard records are discovered.
   -c, --csv         <file>     Comma separated value file.
   -j, --json        <file>     JSON file.
   -v                           Show attempts in the brute force modes.
```

## dnsrecon usage

* general enumeration

``` 
dnsrecon -d asterisk.org
[*] Performing General Enumeration of Domain: asterisk.org
[-] DNSSEC is not configured for asterisk.org
[*] 	 SOA ns.digium.com 216.207.245.18
[*] 	 NS nsx3.digium.com 166.78.177.30
[*] 	 Bind Version for 166.78.177.30 9.8.4-rpz2+rl005.12-P1
[*] 	 NS nsx2.digium.com 216.207.245.19
[*] 	 Bind Version for 216.207.245.19 9.8.1-P1
[*] 	 NS nsx1.digium.com 216.207.245.18
[*] 	 Bind Version for 216.207.245.18 9.8.1-P1
[*] 	 MX mail.digium.com 216.207.245.2
[*] 	 A asterisk.org 216.207.245.25
[*] Enumerating SRV Records
[*] 	 SRV _sip._udp.asterisk.org sip.asterisk.org 204.91.156.60 5060 0
[*] 1 Records Found
```

For this example I selected a site that would also have SRV records. These records help with identifying certain services, in this case SIP for VoIP. Here you can see SIP being used on port 5060 on the host with the address 204.91.156.60.

* perform Google search enumeration

``` 
dnsrecon -d uber.com -g
[*] Performing General Enumeration of Domain: uber.com
[-] DNSSEC is not configured for uber.com
[*] 	 SOA pdns80.ultradns.com 156.154.64.80
[*] 	 NS pdns80.ultradns.net 156.154.65.80
[*] 	 Bind Version for 156.154.65.80 UltraDNS Resolver
[*] 	 NS pdns80.ultradns.net 2610:a1:1014::74
[*] 	 NS pdns80.ultradns.biz 156.154.66.80
[*] 	 Bind Version for 156.154.66.80 UltraDNS Resolver
[*] 	 NS pdns80.ultradns.biz 2610:a1:1015::74
[*] 	 NS pdns80.ultradns.org 156.154.67.80
[*] 	 Bind Version for 156.154.67.80 UltraDNS Resolver
[*] 	 NS pdns80.ultradns.org 2001:502:4612::74
[*] 	 NS pdns80.ultradns.com 156.154.64.80
[*] 	 Bind Version for 156.154.64.80 UltraDNS Resolver
[*] 	 NS pdns80.ultradns.com 2001:502:f3ff::74
[*] 	 MX alt4.aspmx.l.google.com 74.125.28.27
[*] 	 MX alt2.aspmx.l.google.com 74.125.200.26
[*] 	 MX alt3.aspmx.l.google.com 74.125.23.26
[*] 	 MX alt1.aspmx.l.google.com 64.233.161.27
[*] 	 MX aspmx.l.google.com 64.233.167.26
[*] 	 MX alt4.aspmx.l.google.com 2607:f8b0:400e:c04::1a
[*] 	 MX alt2.aspmx.l.google.com 2404:6800:4003:c00::1b
[*] 	 MX alt3.aspmx.l.google.com 2404:6800:4008:c02::1a
[*] 	 MX alt1.aspmx.l.google.com 2a00:1450:4010:c01::1b
[*] 	 MX aspmx.l.google.com 2a00:1450:400c:c0c::1b
[*] 	 A uber.com 104.36.192.133
[*] 	 A uber.com 104.36.192.221
[*] 	 A uber.com 104.36.192.183
[*] 	 A uber.com 104.36.192.132
[*] 	 A uber.com 104.36.192.135
[*] 	 A uber.com 104.36.192.220
[*] 	 A uber.com 104.36.192.182
[*] 	 TXT uber.com v=spf1 include:uber.com._nspf.vali.email include:%{i}._ip.%{h}._ehlo.%{d}._spf.vali.email ~all
[*] 	 TXT uber.com google-site-verification=pO8Bhyhw0N0939yKp6cQICCpvY--sebhQKWGviAkuLM
[*] 	 TXT uber.com docusign=635f0402-4f58-42de-8e07-e1da6d8a971a
[*] Enumerating SRV Records
[*] 	 SRV _kerberos._udp.uber.com kerberos.uber.com 10.6.0.74 88 0
[*] 	 SRV _kpasswd._udp.uber.com kerberos.uber.com 10.6.0.74 464 0
[*] 2 Records Found
[*] Performing Google Search Enumeration
[*] 	 CNAME www.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 	 CNAME help.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 CNAME ride.uber.com d3b8kpte4epqc9.cloudfront.net
[*] 	 A d3b8kpte4epqc9.cloudfront.net 13.32.22.6
[*] 	 A d3b8kpte4epqc9.cloudfront.net 13.32.22.61
[*] 	 A d3b8kpte4epqc9.cloudfront.net 13.32.22.99
[*] 	 A d3b8kpte4epqc9.cloudfront.net 13.32.22.205
[*] 	 A d3b8kpte4epqc9.cloudfront.net 13.32.22.188
[*] 	 A d3b8kpte4epqc9.cloudfront.net 13.32.22.243
[*] 	 A d3b8kpte4epqc9.cloudfront.net 13.32.22.77
[*] 	 A d3b8kpte4epqc9.cloudfront.net 13.32.22.181
[*] 	 CNAME bonjour.uber.com frontends-geo.uber.com
[*] 	 CNAME frontends-geo.uber.com geo-frontends-dca1.uber.com
[*] 	 CNAME geo-frontends-dca1.uber.com frontends-dca1.uber.com
[*] 	 A frontends-dca1.uber.com 104.36.194.234
[*] 	 A frontends-dca1.uber.com 104.36.194.160
[*] 	 A frontends-dca1.uber.com 104.36.194.133
[*] 	 A frontends-dca1.uber.com 104.36.194.131
[*] 	 CNAME accessibility.uber.com uberpolicy.wpengine.com
[*] 	 A uberpolicy.wpengine.com 35.184.61.224
[*] 	 CNAME drive.uber.com uberdrive.wpengine.com
[*] 	 A uberdrive.wpengine.com 35.184.61.224
[*] 	 CNAME central.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 CNAME dial.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 CNAME developer.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 CNAME t.uber.com ghs.google.com
[*] 	 CNAME ghs.google.com ghs.l.google.com
[*] 	 A ghs.l.google.com 172.217.16.179
[*] 	 CNAME t.uber.com ghs.google.com
[*] 	 CNAME ghs.google.com ghs.l.google.com
[*] 	 AAAA ghs.l.google.com 2a00:1450:4001:817::2013
[*] 	 CNAME newsroom.uber.com ubernewblog.wpengine.com
[*] 	 A ubernewblog.wpengine.com 35.184.61.224
[*] 	 CNAME rush.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 	 CNAME join.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A pages.et.uber.com 198.245.92.62
[*] 	 CNAME get.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 	 CNAME freight.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 CNAME eng.uber.com ubereng.wpengine.com
[*] 	 A ubereng.wpengine.com 35.184.61.224
[*] 	 CNAME movement.uber.com frontends-geo.uber.com
[*] 	 CNAME frontends-geo.uber.com geo-frontends-dca1.uber.com
[*] 	 CNAME geo-frontends-dca1.uber.com frontends-dca1.uber.com
[*] 	 A frontends-dca1.uber.com 104.36.194.234
[*] 	 A frontends-dca1.uber.com 104.36.194.160
[*] 	 A frontends-dca1.uber.com 104.36.194.131
[*] 	 A frontends-dca1.uber.com 104.36.194.133
[*] 	 CNAME m.uber.com frontends.uber.com
[*] 	 CNAME frontends.uber.com frontends-sjc1.uber.com
[*] 	 A frontends-sjc1.uber.com 104.36.192.183
[*] 	 A frontends-sjc1.uber.com 104.36.192.221
[*] 	 A frontends-sjc1.uber.com 104.36.192.202
[*] 	 A frontends-sjc1.uber.com 104.36.192.135
[*] 98 Records Found
```

* pull SRV records of domain

``` 
dnsrecon -d cisco.com -t srv
[*] Enumerating Common SRV Records against cisco.com
[*] 	 SRV _sip._tcp.cisco.com vcsgw.cisco.com 173.36.224.91 5060 10
[*] 	 SRV _sips._tcp.cisco.com vcsgw.cisco.com 173.36.224.91 5061 10
[*] 	 SRV _h323ls._udp.cisco.com vcsgw.cisco.com 173.36.224.91 1719 10
[*] 	 SRV _sip._udp.cisco.com vcsgw.cisco.com 173.36.224.91 5060 10
[*] 	 SRV _h323cs._tcp.cisco.com vcsgw.cisco.com 173.36.224.91 1720 10
[*] 	 SRV _xmpp-client._tcp.cisco.com isj3cmx.webexconnect.com 66.163.36.181 5222 1
[*] 	 SRV _xmpp-server._tcp.cisco.com isj3jxf.webexconnect.com 66.163.36.186 5269 1
[*] 7 Records Found
```

* domain bruteforcing

``` 
dnsrecon -D /usr/share/wordlists/dnsmap.txt -t brt -d line.me
[*] Performing host and subdomain brute force against line.me
[*] 	 A ads.line.me 203.104.153.62
[*] 	 A agp.line.me 203.104.153.74
[*] 	 CNAME api.line.me api.line.me.akadns.net
[*] 	 CNAME api.line.me.akadns.net im.api.line.me.edgekey.net
[*] 	 CNAME im.api.line.me.edgekey.net e1102.a1.akamaiedge.net
[*] 	 A e1102.a1.akamaiedge.net 2.17.116.42
[*] 	 A biz.line.me 125.6.149.168
[...]
```

* reverse lookup

``` 
dnsrecon -d nmap.org -w
[...]
[*] Performing Whois lookup against records found.
[*] The following IP Ranges where found:
[*] 	 0) 162.158.0.0-162.159.255.255 Cloudflare, Inc.
[*] 	 1) 74.125.0.0-74.125.255.255 Google Inc.
[*] 	 2) 64.233.160.0-64.233.191.255 Google Inc.
[*] 	 3) 45.33.0.0-45.33.127.255 Linode
[*] What Range do you wish to do a Revers Lookup for?
[*] number, comma separated list, a for all or n for none
3
[*] Linode
[*] Performing Reverse Lookup of range 45.33.0.0-45.33.127.255
[*] Performing Reverse Lookup from 45.33.0.0 to 45.33.127.255
[*] 	 PTR li954-9.members.linode.com 45.33.0.9
[*] 	 PTR li954-4.members.linode.com 45.33.0.4
[*] 	 PTR gw-li954.linode.com 45.33.0.1
[*] 	 PTR li954-7.members.linode.com 45.33.0.7
[*] 	 PTR cloud521.configrapp.com 45.33.0.8
[*] 	 PTR mr2.linode.rbkmoney.net 45.33.0.10
[*] 	 PTR li954-5.members.linode.com 45.33.0.5
[*] 	 PTR rqdq.net 45.33.0.6
[*] 	 PTR li954-12.members.linode.com 45.33.0.12
[...]
```

* working zone transfer

[Robin Wood](https://digi.ninja/projects/zonetransferme.php) has been nice enough to register a domain that allows zone transfers for testing purposes. It's called zonetransfer.me, and here we'll look at partial output from one of the name servers:

```
dnsrecon -d zonetransfer.me -a
[*] Performing General Enumeration of Domain: zonetransfer.me
[*] Checking for Zone Transfer for zonetransfer.me name servers
[*] Resolving SOA Record
[*] 	 SOA nsztm1.digi.ninja 81.4.108.41
[*] Resolving NS Records
[*] NS Servers found:
[*] 	NS nsztm1.digi.ninja 81.4.108.41
[*] 	NS nsztm2.digi.ninja 167.88.42.94
[*] Removing any duplicate NS server IP Addresses...
[*]  
[*] Trying NS server 167.88.42.94
[-] Zone Transfer Failed for 167.88.42.94!
[-] Port 53 TCP is being filtered
[*]  
[*] Trying NS server 81.4.108.41
[*] 81.4.108.41 Has port 53 TCP Open
[*] Zone Transfer was successful!!
[*] 	 SOA nsztm1.digi.ninja 81.4.108.41
[*] 	 NS nsztm1.digi.ninja 81.4.108.41
[*] 	 NS nsztm2.digi.ninja 167.88.42.94
[*] 	 NS intns1.zonetransfer.me 167.88.42.94
[*] 	 NS intns2.zonetransfer.me 167.88.42.94
[*] 	 TXT google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA
[*] 	 TXT Remember to call or email Pippa on +44 123 4567890 or pippa@zonetransfer.me when making DNS changes
[*] 	 TXT '><script>alert('Boo')</script>
[*] 	 TXT AbCdEfG
[*] 	 TXT ZoneTransfer.me service provided by Robin Wood - robin@digi.ninja. See http://digi.ninja/projects/zonetransferme.php for more information.
[*] 	 TXT ; ls
[*] 	 TXT () { :]}; echo ShellShocked
[*] 	 TXT ' or 1=1 --
[*] 	 TXT Robin Wood
[*] 	 PTR www.zonetransfer.me 217.147.177.157
[*] 	 MX @.zonetransfer.me ASPMX.L.GOOGLE.COM 64.233.166.26
[*] 	 MX @.zonetransfer.me ASPMX.L.GOOGLE.COM 2a00:1450:400c:c07::1a
[*] 	 MX @.zonetransfer.me ALT1.ASPMX.L.GOOGLE.COM 173.194.221.27
[*] 	 MX @.zonetransfer.me ALT1.ASPMX.L.GOOGLE.COM 2a00:1450:4010:c0a::1b
[*] 	 MX @.zonetransfer.me ALT2.ASPMX.L.GOOGLE.COM 74.125.130.27
[*] 	 MX @.zonetransfer.me ALT2.ASPMX.L.GOOGLE.COM 2404:6800:4003:c01::1b
[*] 	 MX @.zonetransfer.me ASPMX2.GOOGLEMAIL.COM 173.194.221.26
[*] 	 MX @.zonetransfer.me ASPMX2.GOOGLEMAIL.COM 2a00:1450:4010:c0b::1b
[*] 	 MX @.zonetransfer.me ASPMX3.GOOGLEMAIL.COM 74.125.130.26
[*] 	 MX @.zonetransfer.me ASPMX3.GOOGLEMAIL.COM 2404:6800:4003:c01::1b
[*] 	 MX @.zonetransfer.me ASPMX4.GOOGLEMAIL.COM 108.177.97.27
[*] 	 MX @.zonetransfer.me ASPMX4.GOOGLEMAIL.COM 2404:6800:4008:c00::1b
[*] 	 MX @.zonetransfer.me ASPMX5.GOOGLEMAIL.COM 74.125.28.27
[*] 	 MX @.zonetransfer.me ASPMX5.GOOGLEMAIL.COM 2607:f8b0:400e:c04::1b
[*] 	 AAAA deadbeef.zonetransfer.me dead:beaf::
[*] 	 AAAA ipv6actnow.org.zonetransfer.me 2001:67c:2e8:11::c100:1332
[*] 	 A @.zonetransfer.me 217.147.177.157
[*] 	 A dc-office.zonetransfer.me 143.228.181.132
[*] 	 A owa.zonetransfer.me 207.46.197.32
[*] 	 A alltcpportsopen.firewall.test.zonetransfer.me 127.0.0.1
[*] 	 A vpn.zonetransfer.me 174.36.59.154
[*] 	 A email.zonetransfer.me 74.125.206.26
[*] 	 A asfdbbox.zonetransfer.me 127.0.0.1
[*] 	 A www.zonetransfer.me 217.147.177.157
[*] 	 A canberra-office.zonetransfer.me 202.14.81.230
[*] 	 A intns1.zonetransfer.me 167.88.42.94
[*] 	 A intns2.zonetransfer.me 167.88.42.94
[*] 	 A office.zonetransfer.me 4.23.39.254
[*] 	 CNAME staging.zonetransfer.me www.sydneyoperahouse.com. 52.64.62.190
[*] 	 CNAME staging.zonetransfer.me www.sydneyoperahouse.com. 13.54.224.164
[*] 	 SRV _sip._tcp.zonetransfer.me www 5060 0 no_ip
[*] 	 HINFO Casio fx-700G Windows XP
[*] 	 RP robin robinwood
[*] 	 AFSDB 1 asfdbbox
[*] 	 AFSDB 1 asfdbbox
[*] 	 LOC 53 20 56.558 N 1 38 33.526 W 0.00m
[*] 	 NAPTR P 2 3 !^.*$!sip:customer-service@zonetransfer.me! . E2U+sip
[*] 	 NAPTR P 1 1  email.zonetransfer.me E2U+email
[*] 	 DNSKEY RSASHA256 256 03010001d5deaa18c99ca1dccd489f0b df0dcf6c60db133a0d77b424413774d7 a718dca807252bc46da6428672efc976 3ec86373f80ac1c4e36b01ca788f3f68 ed7ac4ff671b8a602b2c9b2cf24df57b c9455e6d5c5a24e3c8360cf6debb4fd3 3cda156002eb31eaeeab1146bff6bf46 28f32b1add13712a95ed7dd3ace09d77 f8de190b 3
[*] 	 DNSKEY RSASHA256 256 03010001d779d1f1daf7054a21ff7f82 93a1171dcd06adf6586abf4b2911354c 123273d4dce3a44735132842fecb1768 011bcbfe96d1fb9a4add600e573b0b29 dcb7fb178ba9f20b7606353f49ac8d55 d734dda898157e1ecd8b8a9eea4b8464 6091e0f8eabc4568b1a5e092009d7f0e 448b136d076188aadbb6b4ee9344a1cf 8e7cd79f 3
[*] 	 DNSKEY RSASHA256 257 03010001e14b3915ec8b1107d9fb66e6 8a3855023ae83eb3301ade8a7cac4ddc 798ce560740b4e6b723a2f7c9eee55d9 1496dd9cf670d30d1d7f636f59b2070f 32024316c98d74d4916b8dd30b979bd7 fa0453fc55c35af273e6d78176fbab9e f9c331c8c397656ff8bedd8357986c1f e94d590c4655e4c574289f52562db0f9 68c6ca581134f9be1aae2a3fd535aae2 45e717d334f3c8cebb17e82b091f9c52 8ef90a765a45b5591bb808050efcded5 4ee6d150094263ab0df33b57fa790a7f c64e2b368ac22bb043aa2b6051ba5d3e e1610aa99c29562c16b11588d6da23f3 64f652471fe9d0698052671f9c5ab4cb 3d9b64369dda86a63780bc00d115207d 86425a75 3
[*] 	 NSEC _sip._tcp
[*] 	 NSEC sqli
[*] 	 NSEC vpn
[*] 	 NSEC asfdbbox
[*] 	 NSEC deadbeef
[*] 	 NSEC intns1
[*] 	 NSEC 157.177.147.217.IN-ADDR.ARPA
[*] 	 NSEC asfdbauthdns
[*] 	 NSEC dc-office
[*] 	 NSEC alltcpportsopen.firewall.test
[*] 	 NSEC canberra-office
[*] 	 NSEC robinwood
[*] 	 NSEC testing
[*] 	 NSEC www
[*] 	 NSEC Info
[*] 	 NSEC dr
[*] 	 NSEC @
[*] 	 NSEC asfdbvolume
[*] 	 NSEC xss
[*] 	 NSEC email
[*] 	 NSEC internal
[*] 	 NSEC cmdexec
[*] 	 NSEC intns2
[*] 	 NSEC office
[*] 	 NSEC contact
[*] 	 NSEC owa
[*] 	 NSEC staging
[*] 	 NSEC sshock
[*] 	 NSEC sip
[*] 	 NSEC rp
[*] 	 NSEC DZC
[*] 	 NSEC ipv6actnow.org
[*] Checking for Zone Transfer for zonetransfer.me name servers
[*] Resolving SOA Record
[*] 	 SOA nsztm1.digi.ninja 81.4.108.41
[*] Resolving NS Records
[*] NS Servers found:
[*] 	NS nsztm2.digi.ninja 167.88.42.94
[*] 	NS nsztm1.digi.ninja 81.4.108.41
[*] Removing any duplicate NS server IP Addresses...
[*]  
[*] Trying NS server 167.88.42.94
[-] Zone Transfer Failed for 167.88.42.94!
[-] Port 53 TCP is being filtered
[*]  
[*] Trying NS server 81.4.108.41
[*] 81.4.108.41 Has port 53 TCP Open
[*] Zone Transfer was successful!!
[*] 	 SOA nsztm1.digi.ninja 81.4.108.41
[*] 	 NS nsztm1.digi.ninja 81.4.108.41
[*] 	 NS nsztm2.digi.ninja 167.88.42.94
[*] 	 NS intns1.zonetransfer.me 167.88.42.94
[*] 	 NS intns2.zonetransfer.me 167.88.42.94
[*] 	 TXT google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA
[*] 	 TXT Remember to call or email Pippa on +44 123 4567890 or pippa@zonetransfer.me when making DNS changes
[*] 	 TXT '><script>alert('Boo')</script>
[*] 	 TXT AbCdEfG
[*] 	 TXT ZoneTransfer.me service provided by Robin Wood - robin@digi.ninja. See http://digi.ninja/projects/zonetransferme.php for more information.
[*] 	 TXT ; ls
[*] 	 TXT () { :]}; echo ShellShocked
[*] 	 TXT ' or 1=1 --
[*] 	 TXT Robin Wood
[*] 	 PTR www.zonetransfer.me 217.147.177.157
[*] 	 MX @.zonetransfer.me ASPMX.L.GOOGLE.COM 64.233.166.26
[*] 	 MX @.zonetransfer.me ASPMX.L.GOOGLE.COM 2a00:1450:400c:c07::1a
[*] 	 MX @.zonetransfer.me ALT1.ASPMX.L.GOOGLE.COM 173.194.221.27
[*] 	 MX @.zonetransfer.me ALT1.ASPMX.L.GOOGLE.COM 2a00:1450:4010:c0a::1b
[*] 	 MX @.zonetransfer.me ALT2.ASPMX.L.GOOGLE.COM 74.125.130.27
[*] 	 MX @.zonetransfer.me ALT2.ASPMX.L.GOOGLE.COM 2404:6800:4003:c01::1b
[*] 	 MX @.zonetransfer.me ASPMX2.GOOGLEMAIL.COM 173.194.221.26
[*] 	 MX @.zonetransfer.me ASPMX2.GOOGLEMAIL.COM 2a00:1450:4010:c0b::1b
[*] 	 MX @.zonetransfer.me ASPMX3.GOOGLEMAIL.COM 74.125.130.26
[*] 	 MX @.zonetransfer.me ASPMX3.GOOGLEMAIL.COM 2404:6800:4003:c01::1b
[*] 	 MX @.zonetransfer.me ASPMX4.GOOGLEMAIL.COM 108.177.97.27
[*] 	 MX @.zonetransfer.me ASPMX4.GOOGLEMAIL.COM 2404:6800:4008:c00::1b
[*] 	 MX @.zonetransfer.me ASPMX5.GOOGLEMAIL.COM 74.125.28.27
[*] 	 MX @.zonetransfer.me ASPMX5.GOOGLEMAIL.COM 2607:f8b0:400e:c04::1b
[*] 	 AAAA deadbeef.zonetransfer.me dead:beaf::
[*] 	 AAAA ipv6actnow.org.zonetransfer.me 2001:67c:2e8:11::c100:1332
[*] 	 A @.zonetransfer.me 217.147.177.157
[*] 	 A dc-office.zonetransfer.me 143.228.181.132
[*] 	 A owa.zonetransfer.me 207.46.197.32
[*] 	 A alltcpportsopen.firewall.test.zonetransfer.me 127.0.0.1
[*] 	 A vpn.zonetransfer.me 174.36.59.154
[*] 	 A email.zonetransfer.me 74.125.206.26
[*] 	 A asfdbbox.zonetransfer.me 127.0.0.1
[*] 	 A www.zonetransfer.me 217.147.177.157
[*] 	 A canberra-office.zonetransfer.me 202.14.81.230
[*] 	 A intns1.zonetransfer.me 167.88.42.94
[*] 	 A intns2.zonetransfer.me 167.88.42.94
[*] 	 A office.zonetransfer.me 4.23.39.254
[*] 	 CNAME staging.zonetransfer.me www.sydneyoperahouse.com. 52.64.62.190
[*] 	 CNAME staging.zonetransfer.me www.sydneyoperahouse.com. 13.54.224.164
[*] 	 SRV _sip._tcp.zonetransfer.me www 5060 0 no_ip
[*] 	 HINFO Casio fx-700G Windows XP
[*] 	 RP robin robinwood
[*] 	 AFSDB 1 asfdbbox
[*] 	 AFSDB 1 asfdbbox
[*] 	 LOC 53 20 56.558 N 1 38 33.526 W 0.00m
[*] 	 NAPTR P 2 3 !^.*$!sip:customer-service@zonetransfer.me! . E2U+sip
[*] 	 NAPTR P 1 1  email.zonetransfer.me E2U+email
[*] 	 DNSKEY RSASHA256 256 03010001d5deaa18c99ca1dccd489f0b df0dcf6c60db133a0d77b424413774d7 a718dca807252bc46da6428672efc976 3ec86373f80ac1c4e36b01ca788f3f68 ed7ac4ff671b8a602b2c9b2cf24df57b c9455e6d5c5a24e3c8360cf6debb4fd3 3cda156002eb31eaeeab1146bff6bf46 28f32b1add13712a95ed7dd3ace09d77 f8de190b 3
[*] 	 DNSKEY RSASHA256 256 03010001d779d1f1daf7054a21ff7f82 93a1171dcd06adf6586abf4b2911354c 123273d4dce3a44735132842fecb1768 011bcbfe96d1fb9a4add600e573b0b29 dcb7fb178ba9f20b7606353f49ac8d55 d734dda898157e1ecd8b8a9eea4b8464 6091e0f8eabc4568b1a5e092009d7f0e 448b136d076188aadbb6b4ee9344a1cf 8e7cd79f 3
[*] 	 DNSKEY RSASHA256 257 03010001e14b3915ec8b1107d9fb66e6 8a3855023ae83eb3301ade8a7cac4ddc 798ce560740b4e6b723a2f7c9eee55d9 1496dd9cf670d30d1d7f636f59b2070f 32024316c98d74d4916b8dd30b979bd7 fa0453fc55c35af273e6d78176fbab9e f9c331c8c397656ff8bedd8357986c1f e94d590c4655e4c574289f52562db0f9 68c6ca581134f9be1aae2a3fd535aae2 45e717d334f3c8cebb17e82b091f9c52 8ef90a765a45b5591bb808050efcded5 4ee6d150094263ab0df33b57fa790a7f c64e2b368ac22bb043aa2b6051ba5d3e e1610aa99c29562c16b11588d6da23f3 64f652471fe9d0698052671f9c5ab4cb 3d9b64369dda86a63780bc00d115207d 86425a75 3
[*] 	 NSEC _sip._tcp
[*] 	 NSEC sqli
[*] 	 NSEC vpn
[*] 	 NSEC asfdbbox
[*] 	 NSEC deadbeef
[*] 	 NSEC intns1
[*] 	 NSEC 157.177.147.217.IN-ADDR.ARPA
[*] 	 NSEC asfdbauthdns
[*] 	 NSEC dc-office
[*] 	 NSEC alltcpportsopen.firewall.test
[*] 	 NSEC canberra-office
[*] 	 NSEC robinwood
[*] 	 NSEC testing
[*] 	 NSEC www
[*] 	 NSEC Info
[*] 	 NSEC dr
[*] 	 NSEC @
[*] 	 NSEC asfdbvolume
[*] 	 NSEC xss
[*] 	 NSEC email
[*] 	 NSEC internal
[*] 	 NSEC cmdexec
[*] 	 NSEC intns2
[*] 	 NSEC office
[*] 	 NSEC contact
[*] 	 NSEC owa
[*] 	 NSEC staging
[*] 	 NSEC sshock
[*] 	 NSEC sip
[*] 	 NSEC rp
[*] 	 NSEC DZC
[*] 	 NSEC ipv6actnow.org
[-] A timeout error occurred please make sure you can reach the target DNS Servers
[-] directly and requests are not being filtered. Increase the timeout from 3.0 second
[-] to a higher number with --lifetime <time> option.
```

Here we see some interesting records that might warrant an additional explanation:

**SOA** - the State of Authority record indicates which DNS server is the best source of information for the specified domain

**HINFO** specifies the host / server's type of CPU and operating system. We can see a Casio here, which is highly unlikely xD

**LOC** - geolocation information, here in latitude / longitude values

**TXT** - text records, with plenty of information like mail addresses, phone numbers, XSS and shellshock attempts

* map internal network with DNSSEC zone walk

``` 
dnsrecon -d zonetransfer.me -z
[...]
[*] Performing NSEC Zone Walk for zonetransfer.me
[*] Getting SOA record for zonetransfer.me
[*] Name Server 81.4.108.41 will be used
[*] 	 A zonetransfer.me 217.147.177.157
[*] 	 SRV _sip._tcp.zonetransfer.me www.zonetransfer.me 217.147.177.157 5060 0
[*] 	 A 157.177.147.217.IN-ADDR.ARPA.zonetransfer.me no_ip
[*] 	 A asfdbauthdns.zonetransfer.me no_ip
[*] 	 A asfdbbox.zonetransfer.me 127.0.0.1
[*] 	 A asfdbvolume.zonetransfer.me no_ip
[*] 	 A canberra-office.zonetransfer.me 202.14.81.230
[*] 	 A cmdexec.zonetransfer.me no_ip
[*] 	 A contact.zonetransfer.me no_ip
[*] 	 A dc-office.zonetransfer.me 143.228.181.132
[*] 	 AAAA deadbeef.zonetransfer.me dead:beaf::
[*] 	 A dr.zonetransfer.me no_ip
[*] 	 A DZC.zonetransfer.me no_ip
[*] 	 A email.zonetransfer.me 74.125.206.26
[*] 	 A Info.zonetransfer.me no_ip
[*] 	 A internal.zonetransfer.me no_ip
[*] 	 A intns1.zonetransfer.me 167.88.42.94
[*] 	 A intns2.zonetransfer.me 167.88.42.94
[*] 	 A office.zonetransfer.me 4.23.39.254
[*] 	 AAAA ipv6actnow.org.zonetransfer.me 2001:67c:2e8:11::c100:1332
[*] 	 A owa.zonetransfer.me 207.46.197.32
[*] 	 A robinwood.zonetransfer.me no_ip
[*] 	 A rp.zonetransfer.me no_ip
[*] 	 A sip.zonetransfer.me no_ip
[*] 	 A sqli.zonetransfer.me no_ip
[*] 	 A sshock.zonetransfer.me no_ip
[*] 	 A staging.zonetransfer.me 52.64.62.190
[*] 	 A staging.zonetransfer.me 13.54.224.164
[*] 	 A alltcpportsopen.firewall.test.zonetransfer.me 127.0.0.1
[*] 	 CNAME testing.zonetransfer.me www.zonetransfer.me
[*] 	 A www.zonetransfer.me 217.147.177.157
[*] 	 A vpn.zonetransfer.me 174.36.59.154
[*] 	 A www.zonetransfer.me 217.147.177.157
[*] 	 A xss.zonetransfer.me no_ip
[*] 34 records found
```

* sample CSV

{% img center /images/tools/dnsrecon.png 'dnsrecon' 'dnsrecon CSV report' %}

``` 
 _________________________________________
/ The only way to keep your health is to  \
| eat what you don't want, drink what you |
| don't like, and do what you'd rather    |
| not.                                    |
|                                         |
\ -- Mark Twain                           /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

