---
layout: post
title: "Masscan - Kali Linux tools"
date: 2017-06-10 04:17:42 -0400
comments: true
categories: [penetration testing, tools]
keywords: masscan, masscan tutorial, masscan port scanner, masscan kali, masscan kali linux
description: Tutorial for the Masscan port scanner
---

Nmap is our favorite port scanner, but if you want to scan billions of hosts, and still be relatively young when you get the results, there is a solid alternative focused entirely on speed: Masscan - the Mass IP port scanner

<!-- more -->

Homepage: https://github.com/robertdavidgraham/masscan

## masscan description

> This is the fastest Internet port scanner. It can scan the entire Internet in under 6 minutes, transmitting 10 
> million packets per second.

> It produces results similar to nmap, the most famous port scanner. Internally, it operates more like scanrand, 
> unicornscan, and ZMap, using asynchronous transmission. The major difference is that it's faster than these other 
> scanners. In addition, it's more flexible, allowing arbitrary address ranges and port ranges.

> NOTE: masscan uses a custom TCP/IP stack. Anything other than simple port scans will cause conflict with the local 
> TCP/IP stack. This means you need to either use the -S option to use a separate IP address, or configure your 
> operating system to firewall the ports that masscan uses.

> This program spews out packets very fast. On Windows, or from VMs, it can do 300,000 packets/second. On Linux (no 
> virtualization) it'll do 1.6 million packets-per-second. That's fast enough to melt most networks.

> Note that it'll only melt your own network. It randomizes the target IP addresses so that it shouldn't overwhelm any
> distant network.

> By default, the rate is set to 100 packets/second.

## masscan options

``` 
root@kali:~# masscan                                                         
usage:
masscan -p80,8000-8100 10.0.0.0/8 --rate=10000
 scan some web ports on 10.x.x.x at 10kpps
masscan --nmap
 list those options that are compatible with nmap
masscan -p80 10.0.0.0/8 --banners -oB <filename>
 save results of scan in binary format to <filename>
masscan --open --banners --readscan <filename> -oX <savefile>
 read binary scan results in <filename> and save them as xml in <savefile>
```

More information:

``` 
root@kali:~# masscan --help
MASSCAN is a fast port scanner. The primary input parameters are the
IP addresses/ranges you want to scan, and the port numbers. An example
is the following, which scans the 10.x.x.x network for web servers:
 masscan 10.0.0.0/8 -p80
The program auto-detects network interface/adapter settings. If this
fails, you'll have to set these manually. The following is an
example of all the parameters that are needed:
 --adapter-ip 192.168.10.123
 --adapter-mac 00-11-22-33-44-55
 --router-mac 66-55-44-33-22-11
Parameters can be set either via the command-line or config-file. The
names are the same for both. Thus, the above adapter settings would
appear as follows in a configuration file:
 adapter-ip = 192.168.10.123
 adapter-mac = 00-11-22-33-44-55
 router-mac = 66-55-44-33-22-11
All single-dash parameters have a spelled out double-dash equivalent,
so '-p80' is the same as '--ports 80' (or 'ports = 80' in config file).
To use the config file, type:
 masscan -c <filename>
To generate a config-file from the current settings, use the --echo
option. This stops the program from actually running, and just echoes
the current configuration instead. This is a useful way to generate
your first config file, or see a list of parameters you didn't know
about. I suggest you try it now:
 masscan -p1234 --echo
```

Config file:

> By default, the program will read default configuration from  the  file
> /etc/masscan/masscan.conf. This is useful for system-specific settings,
> such as the --adapter-xxx options. This is also useful for excluded  IP
> addresses,  so  that  you  can scan the entire Internet, while skipping
> dangerous addresses, like those owned by the DoD, and not make an accidental mistake.

Nmap compatible options:

``` 
root@kali:~# masscan --nmap                                                  
Masscan (https://github.com/robertdavidgraham/masscan)
Usage: masscan [Options] -p{Target-Ports} {Target-IP-Ranges}
TARGET SPECIFICATION:
  Can pass only IPv4 address, CIDR networks, or ranges (non-nmap style)
  Ex: 10.0.0.0/8, 192.168.0.1, 10.0.0.1-10.0.0.254
  -iL <inputfilename>: Input from list of hosts/networks
  --exclude <host1[,host2][,host3],...>: Exclude hosts/networks
  --excludefile <exclude_file>: Exclude list from file
  --randomize-hosts: Randomize order of hosts (default)
HOST DISCOVERY:
  -Pn: Treat all hosts as online (default)
  -n: Never do DNS resolution (default)
SCAN TECHNIQUES:
  -sS: TCP SYN (always on, default)
SERVICE/VERSION DETECTION:
  --banners: get the banners of the listening service if available. The
    default timeout for waiting to recieve data is 30 seconds.
PORT SPECIFICATION AND SCAN ORDER:
  -p <port ranges>: Only scan specified ports
    Ex: -p22; -p1-65535; -p 111,137,80,139,8080
TIMING AND PERFORMANCE:
  --max-rate <number>: Send packets no faster than <number> per second
  --connection-timeout <number>: time in seconds a TCP connection will
    timeout while waiting for banner data from a port.
FIREWALL/IDS EVASION AND SPOOFING:
  -S/--source-ip <IP_Address>: Spoof source address
  -e <iface>: Use specified interface
  -g/--source-port <portnum>: Use given port number
  --ttl <val>: Set IP time-to-live field
  --spoof-mac <mac address/prefix/vendor name>: Spoof your MAC address
OUTPUT:
  --output-format <format>: Sets output to binary/list/unicornscan/json/grepable/xml
  --output-file <file>: Write scan results to file. If --output-format is
     not given default is xml
  -oL/-oJ/-oG/-oB/-oX/-oU <file>: Output scan in List/JSON/Grepable/Binary/XML/Unicornscan format,
     respectively, to the given filename. Shortcut for
     --output-format <format> --output-file <file>
  -v: Increase verbosity level (use -vv or more for greater effect)
  -d: Increase debugging level (use -dd or more for greater effect)
  --open: Only show open (or possibly open) ports
  --packet-trace: Show all packets sent and received
  --iflist: Print host interfaces and routes (for debugging)
  --append-output: Append to rather than clobber specified output files
  --resume <filename>: Resume an aborted scan
MISC:
  --send-eth: Send using raw ethernet frames (default)
  -V: Print version number
  -h: Print this help summary page.
EXAMPLES:
  masscan -v -sS 192.168.0.0/16 10.0.0.0/8 -p 80
  masscan 23.0.0.0/0 -p80 --banners -output-format binary --output-filename internet.scan
  masscan --open --banners --readscan internet.scan -oG internet_scan.grepable
SEE (https://github.com/robertdavidgraham/masscan) FOR MORE HELP
```

Manpage: http://manpages.org/masscan/8

## masscan usage

* look at the current configuration

``` 
root@kali:~# masscan --echo
rate =     100.00
randomize-hosts = true
seed = 9262294816069822464
shard = 1/1
# ADAPTER SETTINGS
adapter = 
adapter-ip = 0.0.0.0
adapter-mac = 00:00:00:00:00:00
router-mac = 00:00:00:00:00:00
# OUTPUT/REPORTING SETTINGS
output-format = unknown(0)
show = open,,
output-filename = 
rotate = 0
rotate-dir = .
rotate-offset = 0
rotate-filesize = 0
pcap = 
# TARGET SELECTION (IP, PORTS, EXCLUDES)
retries = 0
ports = 

capture = cert
nocapture = html
nocapture = heartbleed

min-packet = 60
```

* check installation

``` 
masscan --regress
regression test: success!
```

* full port scan on local subnet

``` 
root@kali:~# masscan -p0-65535 192.168.217.0/24 --rate 100000

Starting masscan 1.0.3 (http://bit.ly/14GZzcT) at 2017-06-10 09:02:34 GMT
 -- forced options: -sS -Pn -n --randomize-hosts -v --send-eth
Initiating SYN Stealth Scan
Scanning 256 hosts [65536 ports/host]
Discovered open port 443/tcp on 192.168.217.131                                
Discovered open port 139/tcp on 192.168.217.133                                
Discovered open port 139/tcp on 192.168.217.134                                
Discovered open port 139/tcp on 192.168.217.131                                
Discovered open port 111/tcp on 192.168.217.131                                
Discovered open port 23/tcp on 192.168.217.133                                 
Discovered open port 21/tcp on 192.168.217.133                                 
Discovered open port 22/tcp on 192.168.217.131                                 
Discovered open port 22/tcp on 192.168.217.135                                 
Discovered open port 5432/tcp on 192.168.217.133                               
Discovered open port 135/tcp on 192.168.217.134                                
Discovered open port 22/tcp on 192.168.217.133                                 
Discovered open port 25/tcp on 192.168.217.133                                 
Discovered open port 53/tcp on 192.168.217.133                                 
Discovered open port 3632/tcp on 192.168.217.133                               
Discovered open port 80/tcp on 192.168.217.135                                 
Discovered open port 80/tcp on 192.168.217.131                                 
Discovered open port 80/tcp on 192.168.217.133                                 
Discovered open port 445/tcp on 192.168.217.131                                
Discovered open port 445/tcp on 192.168.217.133                                
Discovered open port 445/tcp on 192.168.217.134                                
Discovered open port 8180/tcp on 192.168.217.133                               
Discovered open port 8009/tcp on 192.168.217.133                               
Discovered open port 3306/tcp on 192.168.217.133   
```

The scan was done in a couple of minutes. You might be wondering, how does this tool scan the entire internet in 3 minutes if it took the same length of time for a puny subnet? Well, there are some limitations. To get the most of its speed, you need the proper adapter and driver. Also notice that I rate-limited it to 100k packets per second, because I didn't want my local network to blow up! If you want to benchmark masscan's performance, look on its homepage, under the Performance testing section.

* grab banners of open ports

``` 
root@kali:~# masscan 192.168.217.0/24 -p22,80,139,445,3306 --banners --source-ip 192.168.217.150 --rate 100000

Starting masscan 1.0.3 (http://bit.ly/14GZzcT) at 2017-06-10 09:18:27 GMT
 -- forced options: -sS -Pn -n --randomize-hosts -v --send-eth
Initiating SYN Stealth Scan
Scanning 256 hosts [5 ports/host]
Discovered open port 80/tcp on 192.168.217.133                                 
Discovered open port 139/tcp on 192.168.217.134                                
Discovered open port 22/tcp on 192.168.217.133                                 
Discovered open port 139/tcp on 192.168.217.133                                
Discovered open port 22/tcp on 192.168.217.131                                 
Discovered open port 80/tcp on 192.168.217.131                                 
Discovered open port 445/tcp on 192.168.217.131                                
Discovered open port 139/tcp on 192.168.217.131                                
Discovered open port 22/tcp on 192.168.217.135                                 
Discovered open port 80/tcp on 192.168.217.135                                 
Banner on port 22/tcp on 192.168.217.135: [ssh] SSH-2.0-OpenSSH_5.5p1 Debian-6+squeeze2
Banner on port 22/tcp on 192.168.217.133: [ssh] SSH-2.0-OpenSSH_4.7p1 Debian-8ubuntu1
Banner on port 22/tcp on 192.168.217.131: [ssh] SSH-2.0-OpenSSH_6.6.1          
Discovered open port 445/tcp on 192.168.217.134                                
Discovered open port 445/tcp on 192.168.217.133                                
Discovered open port 3306/tcp on 192.168.217.133                               
Banner on port 3306/tcp on 192.168.217.133: [unknown] \x3e\x00\x00\x00\x0a5.0.51a-3ubuntu5\x00\x07\x00\x00\x00M`yb]-d3\x00,\xaa\x08\x02\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00q^e`0o`r\x3crLb\x00
Banner on port 80/tcp on 192.168.217.131: [http] HTTP/1.1 200 OK\x0d\x0aDate: Sat, 10 Jun 2017 09:18:35 GMT\x0d\x0aServer: Apache/2.4.6 (CentOS) OpenSSL/1.0.1e-fips\x0d\x0aLast-Modified: Tue, 02 Aug 2016 13:07:09 GMT\x0d\x0aETag: \x2211-5391664f1e697\x22\x0d\x0aAccept-Ranges: bytes\x0d\x0aContent-Length: 17\x0d\x0aConnection: close\x0d\x0aContent-Type: text/html; charset=UTF-8\x0d\x0a\x0d
Banner on port 80/tcp on 192.168.217.133: [http] HTTP/1.1 200 OK\x0d\x0aDate: Sat, 10 Jun 2017 09:18:36 GMT\x0d\x0aServer: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.10 with Suhosin-Patch\x0d\x0aLast-Modified: Wed, 17 Mar 2010 14:08:25 GMT\x0d\x0aETag: \x22107f7-2d-481ffa5ca8840\x22\x0d\x0aAccept-Ranges: bytes\x0d\x0aContent-Length: 45\x0d\x0aConnection: close\x0d\x0aContent-Type: text/html\x0d\x0a\x0d
```

For this option to work, you have to give masscan its own IP address on the local subnet, something unused by another device. In fact, the recommendation from its homepage is to use it with its separate IP address whenever possible.

And here is how the XML output of the previous scan would look in a spreadsheet:

{% img center /images/tools/masscan-xml.png 'masscan xml' 'masscan xml output' %}

Some other features are:

* output formats in xml, binary, grepable, list, or JSON

* resume scans

* exclude targets

* runs in Linux, Windows, and Mac OS X

### More resources

- [Robert Graham post on masscan](http://blog.erratasec.com/2013/09/masscan-entire-internet-in-3-minutes.html)

- [select common ports with Nmap and feed them to Masscan](https://josephpierini.blogspot.com/2016/06/using-masscan-with-top-ports.html)

``` 
/ Your own qualities will help prevent \
\ your advancement in the world.       /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
