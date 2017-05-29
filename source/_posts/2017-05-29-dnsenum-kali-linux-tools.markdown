---
layout: post
title: "dnsenum - Kali Linux tools"
date: 2017-05-29 05:32:26 -0400
comments: true
categories: [penetration testing, tools]
keywords: dnsenum, dnsenum tutorial, dnsenum kali, dnsenum kali linux, dns, dnsenum usage, kali dnsenum
description: dnsenum tutorial
---


Today we'll look at dnsenum, one of the tools that come preinstalled on Kali for DNS information gathering.

<!-- more -->

Homepage: https://github.com/fwaeytens/dnsenum

### dnsenum description

> multithreaded perl script to enumerate DNS information of a domain and to discover non-contiguous ip blocks.

> OPERATIONS:

> 1) Get the host's addresse (A record).

> 2) Get the namservers (threaded).

> 3) Get the MX record (threaded).

> 4) Perform axfr queries on nameservers and get BIND VERSION (threaded).

> 5) Get extra names and subdomains via google scraping (google query = "allinurl: -www site:domain").

> 6) Brute force subdomains from file, can also perform recursion on subdomain that have NS records (all threaded).

> 7) Calculate C class domain network ranges and perform whois queries on them (threaded).

> 8) Perform reverse lookups on netranges ( C class or/and whois netranges) (threaded).

> 9) Write to domain_ips.txt file ip-blocks.

### dnsenum options

``` plain
dnsenum -h
dnsenum.pl VERSION:1.2.3
Usage: dnsenum.pl [Options] <domain>
[Options]:
Note: the brute force -f switch is obligatory.
GENERAL OPTIONS:
  --dnsserver 	<server>
			Use this DNS server for A, NS and MX queries.
  --enum		Shortcut option equivalent to --threads 5 -s 15 -w.
  -h, --help		Print this help message.
  --noreverse		Skip the reverse lookup operations.
  --nocolor		Disable ANSIColor output.
  --private		Show and save private ips at the end of the file domain_ips.txt.
  --subfile <file>	Write all valid subdomains to this file.
  -t, --timeout <value>	The tcp and udp timeout values in seconds (default: 10s).
  --threads <value>	The number of threads that will perform different queries.
  -v, --verbose		Be verbose: show all the progress and all the error messages.
GOOGLE SCRAPING OPTIONS:
  -p, --pages <value>	The number of google search pages to process when scraping names,
			the default is 5 pages, the -s switch must be specified.
  -s, --scrap <value>	The maximum number of subdomains that will be scraped from Google (default 15).
BRUTE FORCE OPTIONS:
  -f, --file <file>	Read subdomains from this file to perform brute force.
  -u, --update	<a|g|r|z>
			Update the file specified with the -f switch with valid subdomains.
	a (all)		Update using all results.
	g		Update using only google scraping results.
	r		Update using only reverse lookup results.
	z		Update using only zonetransfer results.
  -r, --recursion	Recursion on subdomains, brute force all discovred subdomains that have an NS record.
WHOIS NETRANGE OPTIONS:
  -d, --delay <value>	The maximum value of seconds to wait between whois queries, the value is defined randomly, default: 3s.
  -w, --whois		Perform the whois queries on c class network ranges.
			 **Warning**: this can generate very large netranges and it will take lot of time to performe reverse lookups.
REVERSE LOOKUP OPTIONS:
  -e, --exclude	<regexp>
			Exclude PTR records that match the regexp expression from reverse lookup results, useful on invalid hostnames.
OUTPUT OPTIONS:
  -o --output <file>	Output in XML format. Can be imported in MagicTree (www.gremwell.com)
```

### Fix dnsenum whois and autoloader errors

First, some preliminary troubleshooting information: dnsenum relies on some Perl modules that may not be already on your system. When running it with certain flags, you might see some warnings like this one: Warning: can't load Net::Whois::IP module, whois queries disabled. To fix it, install the module by running <code>perl -MCPAN -e shell</code> and then at the prompt: <code>install Net::Whois::IP</code>. If you try again, you might get a different error, however: Can't locate package AutoLoader for @net::Whois::IP::ISA at /usr/bin/dnsenum line 536. I scoured the Internet for a bit before finding a workaround: you can make it go away by adding <code>require AutoLoader;</code> to the module source code, or by removing the Autuloader reference: changing <code>@ISA = qw(Exporter AutoLoader);</code> to <code>@ISA = qw(Exporter);</code>. To find out where the module is located, use the CPAN tool:

``` plain
cpan -D Net::Whois::IP
Loading internal null logger. Install Log::Log4perl for logging messages
Reading '/root/.cpan/Metadata'
  Database was generated on Wed, 17 May 2017 06:54:04 GMT
Net::Whois::IP
-------------------------------------------------------------------------
	(no description)
	B/BS/BSCHMITZ/Net-Whois-IP-1.19.tar.gz
	/usr/local/share/perl/5.24.1/Net/Whois/IP.pm
	Installed: 1.19
	CPAN:      1.19  up to date
	Ben Schmitz (BSCHMITZ)
	ben@foink.com
```

You can see the path here, now go apply the fix in the IP.pm file and the error should go away. 

### dnsenum usage

* default

``` plain
dnsenum yahoo.com
dnsenum.pl VERSION:1.2.3

-----   yahoo.com   -----


Host's addresses:
__________________

yahoo.com.                               5        IN    A        206.190.36.45
yahoo.com.                               5        IN    A        98.139.183.24
yahoo.com.                               5        IN    A        98.138.253.109


Name Servers:
______________

ns1.yahoo.com.                           5        IN    A        68.180.131.16
ns3.yahoo.com.                           5        IN    A        203.84.221.53
ns4.yahoo.com.                           5        IN    A        98.138.11.157
ns2.yahoo.com.                           5        IN    A        68.142.255.16
ns5.yahoo.com.                           5        IN    A        119.160.247.124


Mail (MX) Servers:
___________________

mta7.am0.yahoodns.net.                   5        IN    A        98.138.112.32
mta7.am0.yahoodns.net.                   5        IN    A        98.138.112.37
mta7.am0.yahoodns.net.                   5        IN    A        63.250.192.45
mta7.am0.yahoodns.net.                   5        IN    A        66.196.118.34
mta7.am0.yahoodns.net.                   5        IN    A        66.196.118.37
mta7.am0.yahoodns.net.                   5        IN    A        66.196.118.240
mta7.am0.yahoodns.net.                   5        IN    A        98.136.216.26
mta7.am0.yahoodns.net.                   5        IN    A        98.136.217.202
mta5.am0.yahoodns.net.                   5        IN    A        98.138.112.33
mta5.am0.yahoodns.net.                   5        IN    A        98.138.112.34
mta5.am0.yahoodns.net.                   5        IN    A        66.196.118.33
mta5.am0.yahoodns.net.                   5        IN    A        66.196.118.34
mta5.am0.yahoodns.net.                   5        IN    A        66.196.118.35
mta5.am0.yahoodns.net.                   5        IN    A        66.196.118.36
mta5.am0.yahoodns.net.                   5        IN    A        98.136.216.25
mta5.am0.yahoodns.net.                   5        IN    A        98.136.217.202
mta6.am0.yahoodns.net.                   5        IN    A        98.136.216.25
mta6.am0.yahoodns.net.                   5        IN    A        98.138.112.38
mta6.am0.yahoodns.net.                   5        IN    A        98.138.112.37
mta6.am0.yahoodns.net.                   5        IN    A        98.136.217.203
mta6.am0.yahoodns.net.                   5        IN    A        98.138.112.35
mta6.am0.yahoodns.net.                   5        IN    A        66.196.118.36
mta6.am0.yahoodns.net.                   5        IN    A        63.250.192.46
mta6.am0.yahoodns.net.                   5        IN    A        66.196.118.37


Trying Zone Transfers and getting Bind Versions:
_________________________________________________


Trying Zone Transfer for yahoo.com on ns4.yahoo.com ...
AXFR record query failed: REFUSED

Trying Zone Transfer for yahoo.com on ns5.yahoo.com ...
AXFR record query failed: REFUSED

Trying Zone Transfer for yahoo.com on ns2.yahoo.com ...
AXFR record query failed: REFUSED

Trying Zone Transfer for yahoo.com on ns3.yahoo.com ...
AXFR record query failed: REFUSED

Trying Zone Transfer for yahoo.com on ns1.yahoo.com ...
AXFR record query failed: REFUSED

brute force file not specified, bay.
```

From this output, you can see that the script performed DNS queries of the yahoo.com domain, enumerated the DNS and mail servers, and attempted zone transfers through the AXFR record type. Successful zone transfers are a misconfiguration that can have serious security impacts, because the DNS server sends its zone records to whoever requested them, thus revealing potentially sensitive information about the internal network topology, etc.

* with the --enum shortcut, which includes the flags: --threads 5 (5 threads), -s 15 (15 maximum subdomains to be scraped from Google), and -w (perform whois queries on class C ranges)

``` plain
dnsenum --enum kali.org
dnsenum.pl VERSION:1.2.3

-----   kali.org   -----


Host's addresses:
__________________

kali.org.                                5        IN    A        192.124.249.10


Name Servers:
______________

ns5.no-ip.com.                           5        IN    A        204.16.255.155
ns4.no-ip.com.                           5        IN    A        204.16.254.44
ns1.no-ip.com.                           5        IN    A        204.16.255.55
ns3.no-ip.com.                           5        IN    A        207.34.6.1
ns2.no-ip.com.                           5        IN    A        204.16.254.6


Mail (MX) Servers:
___________________

aspmx.l.google.com.                      5        IN    A        108.177.15.27
alt3.aspmx.l.google.com.                 5        IN    A        74.125.23.26
alt4.aspmx.l.google.com.                 5        IN    A        74.125.28.26
alt1.aspmx.l.google.com.                 5        IN    A        108.177.14.27
alt2.aspmx.l.google.com.                 5        IN    A        74.125.200.27


Trying Zone Transfers and getting Bind Versions:
_________________________________________________


Trying Zone Transfer for kali.org on ns2.no-ip.com ... 
AXFR record query failed: REFUSED

Trying Zone Transfer for kali.org on ns4.no-ip.com ... 
AXFR record query failed: REFUSED

Trying Zone Transfer for kali.org on ns3.no-ip.com ... 
AXFR record query failed: REFUSED

Trying Zone Transfer for kali.org on ns1.no-ip.com ... 
AXFR record query failed: NOTAUTH

Trying Zone Transfer for kali.org on ns5.no-ip.com ... 
AXFR record query failed: NOTAUTH


Scraping kali.org subdomains from Google:
__________________________________________


 ----   Google search page: 1   ---- 


 ----   Google search page: 2   ---- 

  docs

 ----   Google search page: 3   ---- 

  de.docs
  archive

 ----   Google search page: 4   ---- 

  archive-4
  ja.docs

 ----   Google search page: 5   ---- 



Google Results:
________________

de.docs.kali.org.                        5        IN    A        192.124.249.10
ja.docs.kali.org.                        5        IN    A        192.124.249.10
docs.kali.org.                           5        IN    A        192.124.249.10
archive.kali.org.                        5        IN    CNAME    hera.kali.org.
hera.kali.org.                           5        IN    A        192.99.45.140
archive-4.kali.org.                      5        IN    CNAME    hecate.kali.org.
hecate.kali.org.                         5        IN    A        149.202.201.51

brute force file not specified, bay.
```

One thing that you should keep in mind is that the Google scraping feature might not always work. In that case, you can check manually by using the same Google operator that dnsenum uses, with "allinurl:-www site:target.com"



* bruteforce subdomains and perform whois queries

dnsenum has a domain bruteforce file located at <code>/usr/share/dnsenum/dns.txt</code>:

``` plain
wc -l /usr/share/dnsenum/dns.txt 
1480 /usr/share/dnsenum/dns.txt
```

Let's see some domain bruteforcing in action! Again, looking at kali.org:

``` plain
dnsenum -f /usr/share/dnsenum/dns.txt -w --noreverse kali.org
[...]
Brute forcing with /usr/share/dnsenum/dns.txt:
_______________________________________________

archive.kali.org.                        5        IN    CNAME    hera.kali.org.
hera.kali.org.                           5        IN    A        192.99.45.140
bugs.kali.org.                           5        IN    A        198.50.203.236
bugs.kali.org.                           5        IN    A        198.50.203.235
forums.kali.org.                         5        IN    A        192.124.249.12
hermes.kali.org.                         5        IN    A        198.27.70.128
http.kali.org.                           5        IN    CNAME    hebe.kali.org.
hebe.kali.org.                           5        IN    A        192.99.200.113
mail.kali.org.                           5        IN    CNAME    apollo.kali.org.
apollo.kali.org.                         5        IN    A        23.239.31.82
old.kali.org.                            5        IN    CNAME    hermes.kali.org.
hermes.kali.org.                         5        IN    A        198.27.70.128
pan.kali.org.                            5        IN    A        167.114.101.148
shop.kali.org.                           5        IN    A        45.79.147.167
www.kali.org.                            5        IN    A        192.124.249.10


Launching Whois Queries:
_________________________

 whois ip result:   23.239.31.0        ->      23.239.0.0/19
 whois ip result:   45.79.147.0        ->      45.79.0.0/16
 whois ip result:   167.114.101.0      ->      167.114.0.0/16
 whois ip result:   192.99.45.0        ->      192.99.0.0/16
 whois ip result:   192.124.249.0      ->      192.124.249.0/24
 whois ip result:   198.27.70.0        ->      198.27.64.0/18
 whois ip result:   198.50.203.0       ->      198.50.203.0/27


kali.org________

 192.99.0.0/16
 23.239.0.0/19
 45.79.0.0/16
 198.50.203.0/27
 192.124.249.0/24
 167.114.0.0/16
 198.27.64.0/18


kali.org ip blocks:
____________________

 23.239.31.82/32
 45.79.147.167/32
 167.114.101.148/32
 192.99.45.140/32
 192.99.200.113/32
 192.124.249.10/32
 192.124.249.12/32
 198.27.70.128/32
 198.50.203.235/32
 198.50.203.236/32

done.
```

And today's cow saying is:

``` plain
/ Tomorrow, this will be part of the    \
| unchangeable past but fortunately, it |
\ can still be changed today.           /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

