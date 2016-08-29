---
layout: post
title: "Kali tools catalog - Wireless Attacks"
date: 2015-04-20 16:31:23 +0300
comments: true
categories: [kali, tools, penetration testing]
keywords: kali, security, tools, wireless, wifi, kali wifi, kali wireless, wireless tools, wireless attacks, wireless hacking, wifi hacking, wireless security, wifi security
description: Overview of Kali Linux tools for the Wireless Attacks category
---


### 802.11 Wireless Tools

**aircrack-ng**

aircrack-ng is an 802.11 WEP and WPA/WPA2-PSK key cracking program.
It can recover the WEP key once enough encrypted packets have been captured with airodump-ng. This part of the aircrack-ng  suite  determines
the  WEP key using two fundamental methods. The first method is via the
PTW approach (Pyshkin, Tews, Weinmann). The main advantage of  the  PTW
approach  is  that  very few data packets are required to crack the WEP
key. The second method is the FMS/KoreK method.  The  FMS/KoreK  method
incorporates  various  statistical  attacks to discover the WEP key and
uses these in combination with brute forcing.

<!-- more -->

Additionally, the program offers a dictionary  method  for  determining
the WEP key. For cracking WPA/WPA2 pre-shared keys, a wordlist (file or
stdin) or an airolib-ng has to be used.

``` plain
  Aircrack-ng 1.2 rc1 - (C) 2006-2013 Thomas d'Otreppe
  http://www.aircrack-ng.org

  usage: aircrack-ng [options] <.cap / .ivs file(s)>

  Common options:

      -a <amode> : force attack mode (1/WEP, 2/WPA-PSK)
      -e <essid> : target selection: network identifier
      -b <bssid> : target selection: access point's MAC
      -p <nbcpu> : # of CPU to use  (default: all CPUs)
      -q         : enable quiet mode (no status output)
      -C <macs>  : merge the given APs to a virtual one
      -l <file>  : write key to file

  Static WEP cracking options:

      -c         : search alpha-numeric characters only
      -t         : search binary coded decimal chr only
      -h         : search the numeric key for Fritz!BOX
      -d <mask>  : use masking of the key (A1:XX:CF:YY)
      -m <maddr> : MAC address to filter usable packets
      -n <nbits> : WEP key length :  64/128/152/256/512
      -i <index> : WEP key index (1 to 4), default: any
      -f <fudge> : bruteforce fudge factor,  default: 2
      -k <korek> : disable one attack method  (1 to 17)
      -x or -x0  : disable bruteforce for last keybytes
      -x1        : last keybyte bruteforcing  (default)
      -x2        : enable last  2 keybytes bruteforcing
      -y         : experimental  single bruteforce mode
      -K         : use only old KoreK attacks (pre-PTW)
      -s         : show the key in ASCII while cracking
      -M <num>   : specify maximum number of IVs to use
      -D         : WEP decloak, skips broken keystreams
      -P <num>   : PTW debug:  1: disable Klein, 2: PTW
      -1         : run only 1 try to crack key with PTW

  WEP and WPA-PSK cracking options:

      -w <words> : path to wordlist(s) filename(s)

  WPA-PSK options:

      -E <file>  : create EWSA Project file v3
      -J <file>  : create Hashcat Capture file
      -S         : WPA cracking speed test
      -r <DB>    : path to airolib-ng database
                   (Cannot be used with -w)

  Other options:

      -u         : Displays # of CPUs & MMX/SSE support
      --help     : Displays this usage screen
```

**asleap**

Actively recover LEAP/PPTP passwords

{% img center /images/kali/wifi/asleap.png 'asleap' 'asleap' %}

**bully**

Bully is a new implementation of the WPS brute force attack, written in C. It is conceptually identical to other programs, in that it exploits the (now well known) design flaw in the WPS specification. It has several advantages over the original reaver code. These include fewer dependencies, improved memory and cpu performance, correct handling of endianness, and a more robust set of options. It runs on Linux, and was specifically developed to run on embedded Linux systems (OpenWrt, etc) regardless of architecture.

Bully provides several improvements in the detection and handling of anomalous scenarios. It has been tested against access points from numerous vendors, and with differing configurations, with much success.

``` plain
  usage: bully <options> interface

  Required arguments:

      interface      : Wireless interface in monitor mode (root required)

      -b, --bssid macaddr    : MAC address of the target access point
   Or
      -e, --essid string     : Extended SSID for the access point

  Optional arguments:

      -c, --channel N[,N...] : Channel number of AP, or list to hop [b/g]
      -i, --index N          : Starting pin index (7 or 8 digits)  [Auto]
      -l, --lockwait N       : Seconds to wait if the AP locks WPS   [43]
      -o, --outfile file     : Output file for messages          [stdout]
      -p, --pin N            : Starting pin number (7 or 8 digits) [Auto]
      -s, --source macaddr   : Source (hardware) MAC address      [Probe]
      -v, --verbosity N      : Verbosity level 1-3, 1 is quietest     [3]
      -w, --workdir path     : Location of pin/session files  [~/.bully/]
      -5, --5ghz             : Hop on 5GHz a/n default channel list  [No]
      -B, --bruteforce       : Bruteforce the WPS pin checksum digit [No]
      -F, --force            : Force continue in spite of warnings   [No]
      -S, --sequential       : Sequential pins (do not randomize)    [No]
      -T, --test             : Test mode (do not inject any packets) [No]

  Advanced arguments:

      -a, --acktime N        : Deprecated/ignored                  [Auto]
      -r, --retries N        : Resend packets N times when not acked  [2]
      -m, --m13time N        : Deprecated/ignored                  [Auto]
      -t, --timeout N        : Deprecated/ignored                  [Auto]
      -1, --pin1delay M,N    : Delay M seconds every Nth nack at M5 [0,1]
      -2, --pin2delay M,N    : Delay M seconds every Nth nack at M7 [5,1]
      -A, --noacks           : Disable ACK check for sent packets    [No]
      -C, --nocheck          : Skip CRC/FCS validation (performance) [No]
      -D, --detectlock       : Detect WPS lockouts unreported by AP  [No]
      -E, --eapfail          : EAP Failure terminate every exchange  [No]
      -L, --lockignore       : Ignore WPS locks reported by the AP   [No]
      -M, --m57nack          : M5/M7 timeouts treated as WSC_NACK's  [No]
      -N, --nofcs            : Packets don't contain the FCS field [Auto]
      -P, --probe            : Use probe request for nonbeaconing AP [No]
      -R, --radiotap         : Assume radiotap headers are present [Auto]
      -W, --windows7         : Masquerade as a Windows 7 registrar   [No]
      -Z, --suppress         : Suppress packet throttling algorithm  [No]
      -V, --version          : Print version info and exit
      -h, --help             : Display this help information
```

**cowpatty**

Implementation of an offline dictionary attack against WPA/WPA2 networks using PSK-based authentication (e.g. WPA-Personal). Many enterprise networks deploy PSK-based authentication mechanisms for WPA/WPA2 since it is much easier than establishing the necessary RADIUS, supplicant and certificate authority architecture needed for WPA-Enterprise authentication. Cowpatty can implement an accelerated attack if a precomputed PMK file is available for the SSID that is being assessed.

{% img center /images/kali/wifi/cowpatty.png 'cowpatty' 'cowpatty' %}

**eapmd5pass**

EAP-MD5 is a legacy authentication mechanism that does not provide sufficient protection for user authentication credentials. Users who authenticate using EAP-MD5 subject themselves to an offline dictionary attack vulnerability. This tool reads from a live network interface in monitor-mode, or from a stored libpcap capture file, and extracts the portions of the EAP-MD5 authentication exchange. Once the challenge and response portions have been collected from this exchange, eapmd5pass will mount an offline dictionary attack against the user’s password.

{% img center /images/kali/wifi/eapmd5pass.png 'eapmd5pass' 'eapmd5pass' %}

**fern-wifi-cracker**

Fern Wifi Cracker is a Wireless security auditing and attack software program written using the Python Programming Language and the Python Qt GUI library, the program is able to crack and recover WEP/WPA/WPS keys and also run other network based attacks on wireless or ethernet based networks.

Fern Wifi Cracker currently supports the following features:

* WEP Cracking with Fragmentation,Chop-Chop, Caffe-Latte, Hirte, ARP Request Replay or WPS attack

* WPA/WPA2 Cracking with Dictionary or WPS based attacks

* Automatic saving of key in database on successful crack

* Automatic Access Point Attack System

* Session Hijacking (Passive and Ethernet Modes)

* Access Point MAC Address Geo Location Tracking

* Internal MITM Engine

* Bruteforce Attacks (HTTP,HTTPS,TELNET,FTP)

* Update Support

{% img center /images/kali/wifi/fern-wifi-cracker.png 'fern-wifi-cracker' 'fern-wifi-cracker' %}

**genkeys**

Generates lookup file for asleap

``` plain
genkeys 2.2 - generates lookup file for asleap. <jwright@hasborg.com>
genkeys: Must supply -r -f and -n
Usage: genkeys [options]

	-r 	Input dictionary file, one word per line
	-f 	Output pass+hash filename
	-n 	Output index filename
	-h 	Last 2 hash bytes to filter with (optional)
```

**genpmk**

WPA-PSK precomputation attack

{% img center /images/kali/wifi/genpmk.png 'genpmk' 'genpmk' %}


**giskismet**

GISKismet is a wireless recon visualization tool to represent data gathered using Kismet in a flexible manner. GISKismet stores the information in a database so that the user can generate graphs using SQL. GISKismet currently uses SQLite for the database and GoogleEarth / KML files for graphing.

``` plain
Usage: giskismet [Options]

Input File:
       --csv <csv-file>             Parse the input from Kismet-devel CSV
   -x  --xml <xml-file>             Parse the input from Kismet-newcore NETXML

Input Filters: 
       --bssid file | list          Filter based on BSSID     
       --essid file | list          Filter based on ESSID 
       --encryption file | list     Filter based on Encryption 
       --channel file | list        Filter based on Channel

file | list (list = comma separated lists(needs quotes)

Kismet-newcore Options:
   -a  --ap                         Insert only the APs

Query
   -q  --query [sql]                SQL query
   -m  --manual [csv]               CSV output of manual SQL query

   -o  --output [file]              Output filename
   -n  --name [str]                 Name of the KML layer
       --desc [str]                 Description of the KML layer

General Options:                
       --ignore-gps                 Import data even when GPS fields are missing
       --database [file]            SQLite3 database name [default: wireless.dbl]
   -d  --debug [num]                Display debug information
   -s  --silent                     No output when adding APs
   -v  --version                    Display version
   -h  --help                       Display this information

Send Comments to Joshua "Jabra" Abraham ( jabra@spl0it.org )
```

**kismet**

Kismet is an 802.11 layer2 wireless network detector, sniffer, and intrusion detection system. Kismet will work with any wireless card which supports raw monitoring (rfmon) mode, and (with appropriate hardware) can sniff 802.11b, 802.11a, 802.11g, and 802.11n traffic. Kismet also supports plugins which allow sniffing other media such as DECT.

Kismet identifies networks by passively collecting packets and detecting standard named networks, detecting (and given time, decloaking) hidden networks, and infering the presence of nonbeaconing networks via data traffic. 

Kismet supports logging to the wtapfile packet format (readable by tcpdump and ethereal) and saves detected network information as plaintext,
CSV, and XML.  kismet is capable of using any GPS supported by gpsd and
logs and plots network data.

{% img center /images/kali/wifi/kismet.png 'kismet' 'kismet' %}

**mdk3**

MDK is a proof-of-concept tool to exploit common IEEE 802.11 protocol weaknesses.

``` plain
MDK 3.0 v6 - "Yeah, well, whatever"
by ASPj of k2wrlz, using the osdep library from aircrack-ng
And with lots of help from the great aircrack-ng community:
Antragon, moongray, Ace, Zero_Chaos, Hirte, thefkboss, ducttape,
telek0miker, Le_Vert, sorbo, Andy Green, bahathir and Dawid Gajownik
THANK YOU!

MDK is a proof-of-concept tool to exploit common IEEE 802.11 protocol weaknesses.
IMPORTANT: It is your responsibility to make sure you have permission from the
network owner before running MDK against it.

This code is licenced under the GPLv2

MDK USAGE:
mdk3 <interface> <test_mode> [test_options]

Try mdk3 --fullhelp for all test options
Try mdk3 --help <test_mode> for info about one test only

TEST MODES:
b   - Beacon Flood Mode
      Sends beacon frames to show fake APs at clients.
      This can sometimes crash network scanners and even drivers!
a   - Authentication DoS mode
      Sends authentication frames to all APs found in range.
      Too much clients freeze or reset some APs.
p   - Basic probing and ESSID Bruteforce mode
      Probes AP and check for answer, useful for checking if SSID has
      been correctly decloaked or if AP is in your adaptors sending range
      SSID Bruteforcing is also possible with this test mode.
d   - Deauthentication / Disassociation Amok Mode
      Kicks everybody found from AP
m   - Michael shutdown exploitation (TKIP)
      Cancels all traffic continuously
x   - 802.1X tests
w   - WIDS/WIPS Confusion
      Confuse/Abuse Intrusion Detection and Prevention Systems
f   - MAC filter bruteforce mode
      This test uses a list of known client MAC Adresses and tries to
      authenticate them to the given AP while dynamically changing
      its response timeout for best performance. It currently works only
      on APs who deny an open authentication request properly
g   - WPA Downgrade test
      deauthenticates Stations and APs sending WPA encrypted packets.
      With this test you can check if the sysadmin will try setting his
      network to WEP or disable encryption.
```

**wifiarp**

Wifi injection ARP answering tool based on Wifitap

**wifidns**

Wifi injection DNS answering tool based on Wifitap

``` plain
root@kali:~# wifidns -h
Psyco optimizer not installed, running anyway...
INFO: did not find python gnuplot wrapper . Won't be able to plot
INFO: Can't open /etc/ethertypes file
Usage: wifidns -b <BSSID> -a <IP> [-o <iface>] [-i <iface>]
                          [-s <SMAC>] [-t <TTL>] [-w <WEP key>]
                          [-k <key id>]] [-d [-v]] [-h]
     -b <BSSID>    specify BSSID for injection
     -a <IP>       specify IP address for DNS answers
     -t <TTL>      Set TTL (default: 64)
     -o <iface>    specify interface for injection (default: ath0)
     -i <iface>    specify interface for listening (default: ath0)
     -s <SMAC>     specify source MAC address for injected frames
     -w <key>      WEP mode and key
     -k <key id>   WEP key id (default: 0)
     -d            activate debug
     -v            verbose debugging
     -h            this so helpful output
```

**wifi-honey**

This script creates five monitor mode interfaces, four are used as APs and the fifth is used for airodump-ng. To make things easier, rather than having five windows all this is done in a screen session which allows you to switch between screens to see what is going on. All sessions are labelled so you know which is which.

``` plain
Usage: /usr/bin/wifi-honey <essid> <channel> <interface>

Default channel is 1
Default interface is wlan0

Robin Wood <robin@digininja.org>
See Security Tube Wifi Mega Primer episode 26 for more information
```

**wifiping**

Wifi injection based answering tool based on Wifitap

**wifitap**

Wifitap is a proof of concept for communication over WiFi networks using traffic injection.

Wifitap allows any application do send and receive IP packets using 802.11 traffic capture and injection over a WiFi network simply configuring wj0, which means :

* setting an IP address consistent with target network address range

* routing desired traffic through it

In particular, it’s a cheap method for arbitrary packets injection in 802.11 frames without specific library.

In addition, it will allow one to get rid of any limitation set at access point level, such as bypassing inter-client communications prevention systems (e.g. Cisco PSPF) or reaching multiple SSID handled by the same access point.

{% img center /images/kali/wifi/wifitap.png 'wifitap' 'wifitap' %}

**wifite**

An automated wireless attack tool. To attack multiple WEP, WPA, and WPS encrypted networks in a row. This tool is customizable to be automated with only a few arguments. Wifite aims to be the "set it and forget it" wireless auditing tool. 

Features

* sorts targets by signal strength (in dB); cracks closest access points first

* automatically de-authenticates clients of hidden networks to reveal SSIDs

* numerous filters to specify exactly what to attack (wep/wpa/both, above certain signal strengths, channels, etc)

* customizable settings (timeouts, packets/sec, etc)

* "anonymous" feature; changes MAC to a random address before attacking, then changes back when attacks are complete

* all captured WPA handshakes are backed up to wifite.py's current directory

* smart WPA de-authentication; cycles between all clients and broadcast deauths

* stop any attack with Ctrl+C, with options to continue, move onto next target, skip to cracking, or exit

* displays session summary at exit; shows any cracked keys

* all passwords saved to cracked.txt

* built-in updater: ./wifite.py -upgrade 

``` plain
 .;'                     `;,    
 .;'  ,;'             `;,  `;,   WiFite v2 (r85)
.;'  ,;'  ,;'     `;,  `;,  `;,  
::   ::   :   ( )   :   ::   ::  automated wireless auditor
':.  ':.  ':. /_\ ,:'  ,:'  ,:'  
 ':.  ':.    /___\    ,:'  ,:'   designed for Linux
  ':.       /_____\      ,:'     
           /       \             

   COMMANDS
	-check <file>	check capfile <file> for handshakes.
	-cracked    	display previously-cracked access points

   GLOBAL
	-all         	attack all targets.              [off]
	-i <iface>  	wireless interface for capturing [auto]
	-mac         	anonymize mac address            [off]
	-c <channel>	channel to scan for targets      [auto]
	-e <essid>  	target a specific access point by ssid (name)  [ask]
	-b <bssid>  	target a specific access point by bssid (mac)  [auto]
	-showb       	display target BSSIDs after scan               [off]
	-pow <db>   	attacks any targets with signal strenghth > db [0]
	-quiet       	do not print list of APs during scan           [off]


   WPA
	-wpa        	only target WPA networks (works with -wps -wep)   [off]
	-wpat <sec>   	time to wait for WPA attack to complete (seconds) [500]
	-wpadt <sec>  	time to wait between sending deauth packets (sec) [10]
	-strip      	strip handshake using tshark or pyrit             [off]
	-crack <dic>	crack WPA handshakes using <dic> wordlist file    [off]
	-dict <file>	specify dictionary to use when cracking WPA [phpbb.txt]
	-aircrack   	verify handshake using aircrack [on]
	-pyrit      	verify handshake using pyrit    [off]
	-tshark     	verify handshake using tshark   [on]
	-cowpatty   	verify handshake using cowpatty [off]

   WEP
	-wep        	only target WEP networks [off]
	-pps <num>  	set the number of packets per second to inject [600]
	-wept <sec> 	sec to wait for each attack, 0 implies endless [600]
	-chopchop   	use chopchop attack      [on]
	-arpreplay  	use arpreplay attack     [on]
	-fragment   	use fragmentation attack [on]
	-caffelatte 	use caffe-latte attack   [on]
	-p0841      	use -p0841 attack        [on]
	-hirte      	use hirte (cfrag) attack [on]
	-nofakeauth 	stop attack if fake authentication fails    [off]
	-wepca <n>  	start cracking when number of ivs surpass n [10000]
	-wepsave    	save a copy of .cap files to this directory [off]

   WPS
	-wps       	only target WPS networks         [off]
	-wpst <sec>  	max wait for new retry before giving up (0: never)  [660]
	-wpsratio <per>	min ratio of successful PIN attempts/total tries    [0]
	-wpsretry <num>	max number of retries for same PIN before giving up [0]

   EXAMPLE
	./wifite.py -wps -wep -c 6 -pps 600

 [+] quitting
```

### Bluetooth Tools

**bluelog**

Bluelog  is  a simple Bluetooth scanner that is designed to essentially
do just one thing, log all the discoverable devices in the area. It  is
intended  to  be  used as a site survey tool, identifying the number of
possible Bluetooth targets there are in the surrounding environment.

``` plain
Bluelog (v1.1.2) by Tom Nardi "MS3FGX" (MS3FGX@gmail.com)
----------------------------------------------------------------
Bluelog is a Bluetooth site survey tool, designed to tell you how
many discoverable devices there are in an area as quickly as possible.
As the name implies, its primary function is to log discovered devices
to file rather than to be used interactively. Bluelog could run on a
system unattended for long periods of time to collect data.

Bluelog also includes a mode called "Bluelog Live" which creates a
webpage of the results that you can serve up with your HTTP daemon of
choice. See the "README.LIVE" file for details.

For more information, see: www.digifail.com

Basic Options:
	-i <interface>     Sets scanning device, default is "hci0"
	-o <filename>      Sets output filename, default is "devices.log"
	-v                 Verbose, prints discovered devices to the terminal
	-q                 Quiet, turns off nonessential terminal outout
	-d                 Enables daemon mode, Bluelog will run in background
	-k                 Kill an already running Bluelog process
	-l                 Start "Bluelog Live", default is disabled

Logging Options:
	-n                 Write device names to log, default is disabled
	-m                 Write device manufacturer to log, default is disabled
	-c                 Write device class to log, default is disabled
	-f                 Use "friendly" device class, default is disabled
	-t                 Write timestamps to log, default is disabled
	-x                 Obfuscate discovered MACs, default is disabled
	-e                 Encode discovered MACs with CRC32, default disabled
	-b                 Enable BlueProPro log format, see README

Advanced Options:
	-r <retries>       Name resolution retries, default is 3
	-a <minutes>       Amnesia, Bluelog will forget device after given time
	-w <seconds>       Scanning window in seconds, see README
	-s                 Syslog only mode, no log file. Default is disabled
```

**bluemaho**

BlueMaho is GUI-shell (interface) for suite of tools for testing security of bluetooth devices. It is freeware, opensource, written on python, uses wxPyhon. It can be used for testing BT-devices for known vulnerabilities and major thing to do – testing to find unknown vulns. Also it can form nice statistics.

Features:

* scan for devices, show advanced info, SDP records, vendor etc

* track devices – show where and how much times device was seen, its name changes

* loop scan – it can scan all time, showing you online devices

* alerts with sound if new device found

* on_new_device – you can specify what command should it run when it founds new device

* it can use separate dongles – one for scaning (loop scan) and one for running tools or exploits

* send files

* change name, class, mode, BD_ADDR of local HCI devices

* save results in database

* form nice statistics (uniq devices by day/hour, vendors, services etc)

* test remote device for known vulnerabilities (see exploits for more details)

* test remote device for unknown vulnerabilities (see tools for more details)

* themes! you can customize it

{% img center /images/kali/wifi/bluemaho.png 'bluemaho' 'bluemaho' %}

**blueranger**

BlueRanger is a simple Bash script which uses Link Quality to locate Bluetooth device radios. It sends l2cap (Bluetooth) pings to create a connection between Bluetooth interfaces, since most devices allow pings without any authentication or authorization. The higher the link quality, the closer the device (in theory).

Use a Bluetooth Class 1 adapter for long range location detection. Switch to a Class 3 adapter for more precise short range locating. The recision and accuracy depend on the build quality of the Bluetooth adapter, interference, and response from the remote device. Fluctuations may occur even when neither device is in motion.

``` plain
BlueRanger 1.0 by JP Dunning (.ronin) 
<www.hackfromacave.com>
(c) 2009-2012 Shadow Cave LLC.

NAME
	blueranger

SYNOPSIS
        blueranger.sh <hciX> <bdaddr>

DESCRIPTION
	<hciX>         Local interface
	<bdaddr>       Remote Device Address
```

**bluesnarfer**

A Bluetooth bluesnarfing Utility.

{% img center /images/kali/wifi/bluesnarfer.png 'bluesnarfer' 'bluesnarfer' %}

**btscanner**

btscanner is a tool designed specifically to extract as  much  information  as  possible  from  a Bluetooth device without the requirement to
pair. A detailed information screen extracts HCI and  SDP  information,
and  maintains an open connection to monitor the RSSI and link quality.
btscanner is based on the BlueZ Bluetooth stack, which is included with
recent  Linux kernels, and the BlueZ toolset. btscanner also contains a
complete listing of the IEEE OUI numbers and class lookup tables. Using
the information gathered from these sources it is possible to make educated guesses as to the host device type.

``` plain
Usage: btscanner [options]
options
	--help	Display help
	--cfg=<file>	Use <file> as the config file
	--no-reset	Do not reset the Bluetooth adapter before scanning
```

**redfang**

RedFang is a small proof-of-concept application to find non discoverable Bluetooth devices. This is done by brute forcing the last six (6) bytes of the Bluetooth address of the device and doing a read_remote_name().

{% img center /images/kali/wifi/redfang.png 'redfang' 'redfang' %}

**spooftooph**

Spooftooph is designed to automate spoofing or cloning Bluetooth device information. Make a Bluetooth device hide in plain site.

Features:

* Clone and log Bluetooth device information

* Generate a random new Bluetooth profile

* Change Bluetooth profile every X seconds

* Specify device information for Bluetooth interface

* Select device to clone from scan log

{% img center /images/kali/wifi/spooftooph.png 'spooftooph' 'spooftooph' %}


### Other Wireless Tools

**zbassocflood**

Transmit a flood of associate requests to a target network.

``` plain
zbassocflood: Transmit a flood of associate requests to a target network.
jwright@willhackforsushi.com

Usage: zbassocflood [-pcDis] [-i devnumstring] [-p PANID] [-c channel]
                        [-s per-packet delay/float]

e.x. zbassocflood -p 0xBAAD -c 11 -s 0.1
```

**zbdsniff**

Decode plaintext key ZigBee delivery from a capture file.  Will
process libpcap or Daintree SNA capture files.

``` plain
zbdsniff: Decode plaintext key ZigBee delivery from a capture file.  Will
process libpcap or Daintree SNA capture files.    jwright@willhackforsushi.com

Usage: zbdsniff [capturefiles ...]
```

**zbdump**

A tcpdump-like tool for ZigBee/IEEE 802.15.4 networks

``` plain
zbdump - a tcpdump-like tool for ZigBee/IEEE 802.15.4 networks
Compatible with Wireshark 1.1.2 and later - jwright@willhackforsushi.com

Usage: zbdump [-fiwDch] [-f channel] [-w pcapfile] [-W daintreefile] 
         [-i devnumstring]
```

**zbfind**

zbfind provides a GTK-based GUI to the user which displays the results of a zbstumbler-like functionality. zbfind sends beacon requests as it cycles through channels and listens for a response, adding the response to a table as well as displaying signal strength on a gauge widget. 

{% img center /images/kali/wifi/zbfind.png 'zbfind' 'zbfind' %}

**zbgoodfind**

Search a binary file to identify the encryption key for a given
SNA or libpcap IEEE 802.15.4 encrypted packet

``` plain
zbgoodfind - search a binary file to identify the encryption key for a given
SNA or libpcap IEEE 802.15.4 encrypted packet - jwright@willhackforsushi.com

Usage: zbgoodfind [-frRFd] [-f binary file] [-r pcapfile] [-R daintreefile] 
         [-F Don't skip 2-byte FCS at end of each frame]
         [-d genenerate binary file (test mode)]
```

**zbreplay**

Replay ZigBee/802.15.4 network traffic from libpcap or Daintree files

``` plain
zbreplay: replay ZigBee/802.15.4 network traffic from libpcap or Daintree files
jwright@willhackforsushi.com

Usage: zbreplay [-rRfiDch] [-f channel] [-r pcapfile] [-R daintreefile] 
         [-i devnumstring] [-s delay/float] [-c countpackets]
```

**zbstumbler**

Transmit beacon request frames to the broadcast address while
channel hopping to identify ZC/ZR devices.

``` plain
zbstumbler: Transmit beacon request frames to the broadcast address while
channel hopping to identify ZC/ZR devices.  jwright@willhackforsushi.com

Usage: zbstumbler [-iscwD] [-i devnumstring] [-s per-channel delay] [-c channel]
                          [-w report.csv]
```


### RFID / NFC Tools

### NFC Tools

**mfcuk**

Toolkit containing samples and various tools based on and around libnfc and crapto1, with emphasis on Mifare Classic NXP/Philips RFID cards. Special emphasis of the toolkit is on the following:

* mifare classic weakness demonstration/exploitation

* demonstrate use of libnfc (and ACR122 readers)

* demonstrate use of Crapto1 implementation to confirm internal workings and to verify theoretical/practical weaknesses/attacks

**mfoc**

MFOC is an open source implementation of “offline nested” attack by Nethemba.
This program allow to recover authentication keys from MIFARE Classic card.
Please note MFOC is able to recover keys from target only if it have a known key: default one (hardcoded in MFOC) or custom one (user provided using command line).

**mfterm**

A terminal interface for working with Mifare tags.

The  program  is  used as an interactive shell to read and write Mifare
tags using libnfc and a libnfc compatible reader or to  simply  manipulate Mifare data dumps from files.

``` plain
A terminal interface for working with Mifare Classic tags.
Usage: mfterm [-v] [-h] [-k keyfile]

Options: 
  --help          (-h)   Show this help message.
  --version       (-v)   Display version information.
  --tag=tagfile   (-t)   Load a tag from the specified file.
  --keys=keyfile  (-k)   Load keys from the specified file.
  --dict=dictfile (-d)   Load dictionary from the specified file.

Report bugs to: anders@4zm.org
mfterm home page: <https://github.com/4zm/mfterm>
```

**mifare-classic-format**

``` plain
usage: mifare-classic-format [-fy] [keyfile]

Options:
  -f      Fast format (only erase MAD)
  -y      Do not ask for confirmation (dangerous)
  keyfile Use keys from dump in addition to internal default keys
```

**nfc-list**

nfc-list  is  a utility for listing any available tags like ISO14443-A,
FeliCa, Jewel or ISO14443-B (according to the device capabilities).  It
may  detect several tags at once thanks to a mechanism called anti-collision but all types of tags don't support anti-collision and there  is
some physical limitation of the number of tags the reader can discover.

This tool displays all available information at selection time.

``` plain
nfc-list uses libnfc 1.7.0
usage: nfc-list [-v]
  -v	 verbose display
```

**nfc-mfclassic**

nfc-mfclassic is a MIFARE Classic tool that allow to read or write DUMP
file using MIFARE keys provided in KEYS file.

``` plain
Usage: nfc-mfclassic r|R|w|W a|b <dump.mfd> [<keys.mfd> [f]]
  r|R|w|W       - Perform read from (r) or unlocked read from (R) or write to (w) or unlocked write to (W) card
                  *** note that unlocked write will attempt to overwrite block 0 including UID
                  *** unlocked read does not require authentication and will reveal A and B keys
                  *** unlocking only works with special Mifare 1K cards (Chinese clones)
  a|A|b|B       - Use A or B keys for action; Halt on errors (a|b) or tolerate errors (A|B)
  <dump.mfd>    - MiFare Dump (MFD) used to write (card to MFD) or (MFD to card)
  <keys.mfd>    - MiFare Dump (MFD) that contain the keys (optional)
  f             - Force using the keyfile even if UID does not match (optional)
```

### RFIDiot ACG

### RFIDiot FROSCH 

### RFIDiot PCSC


A collection of tools and libraries for exploring RFID technology, written 
in Python.

**ChAP.py**

Script that tries to select the EMV Payment Systems Directory on all inserted cards.

**bruteforce.py**

Try random numbers to login to sector 0

**cardselect.py**

Select card and display ID

**copytag.py**

Read all sectors from a standard tag and write them back to a blank

**demotag.py**

Test IAIK TUG DemoTag

**eeprom.py**

Display reader's eeprom settings

**fdxbnum.py**

Generate / decode FDX-B EM4x05 compliant IDs

**formatmifare1kvalue.py**

Format value blocks on a mifare standard tag

**froschtest.py**

Test frosch HTRM112 reader

**hidprox.py**

Show HID Prox card type and site/id code

**hitag2brute.py**

Brute Force hitag2 password

**hitag2reset.py**

Reset hitag2 password

**isotype.py**

Determine ISO tag type

**jcopmifare.py**

Test program for mifare emulation on JCOP

**jcopsetatrhist.py**

Set ATR History bytes on JCOP cards

**jcoptool.py**

JCOP card toolkit

**lfxtype.py**

Select card and display tag type

**loginall.py**

Attempt to login to each sector with transport keys

**mifarekeys.py**

Calculate 3DES key for Mifare access on JCOP cards

**mrpkey.py**

Calculate 3DES key for Machine Readable Passport

**multiselect.py**

Continuously select card and display ID

**nfcid.py**

Python code for Identifying NFC cards

**pn532emulate.py**

Switch NXP PN532 reader chip into TAG emulation mode

**pn532mitm.py**

NXP PN532 Man-In-The_Middle - log conversations between TAG and external reader

**q5reset.py**

Reset q5 tag

**readlfx.py**

Read all sectors from a LFX reader

**readmifare1k.py**

Read all sectors from a mifare standard tag

**readmifaresimple.py**

Read all sectors from a mifare tag

**readmifareultra.py**

Read all sectors from a Ultralight tag

**readtag.py**

Read all sectors from a standard tag

**rfidiot-cli.py**

CLI for rfidiot

**send_apdu.py**

Python code for Sending raw APDU commands

**sod.py**

Try to find X509 data in EF.SOD

**transit.py**

Generate / decode FDI Matalec Transit 500 and Transit 999 UIDs

**unique.py**

Generate EM4x02 and/or UNIQUE compliant IDs

**writelfx.py**

Read and then write all sectors from a LFX reader

**writemifare1k.py**

Write all blocks on a mifare standard tag

Check http://rfidiot.org/ for more information and examples


### Software Defined Radio

**gnuradio-companion**

A graphical tool for creating signal flow graphs and generating flow-graph source code.

{% img center /images/kali/wifi/gnuradio.png 'gnuradio-companion' 'gnuradio' %}

**gqrx**

Gqrx is a software defined radio receiver powered by the GNU Radio SDR framework and the Qt graphical toolkit. Gqrx supports many of the SDR hardware available, including Funcube Dongles, rtl-sdr, HackRF and USRP devices.

Currently it works on Linux and Mac and supports the following devices:. Funcube Dongle Pro and Pro+ RTL2832U-based DVB-T dongles (rtlsdr via USB and TCP) OsmoSDR USRP HackRF Jawbreaker Nuand bladeRF any other device supported by the gr-osmosdr library

The latest stable version of Gqrx is 2.2, it is available for Linux, FreeBSD and Mac and it offers the following features:

* Discover devices attached to the computer.

* Process I/Q data from the supported devices.

* Change frequency, gain and apply various corrections (frequency, I/Q balance).

* AM, SSB, FM-N and FM-W (mono and stereo) demodulators.

* Special FM mode for NOAA APT.

* Variable band pass filter.

* AGC, squelch and noise blankers.

* FFT plot and waterfall.

* Record and playback audio to / from WAV file.

* Spectrum analyzer mode where all signal processing is disabled.

{% img center /images/kali/wifi/gqrx.png 'gqrx' 'gqrx' %}

**gr-scan**

gr-scan is a program written in C++, and built upon GNU Radio, rtl-sdr, and the OsmoSDR Source Block. It is intended to scan a range of frequencies and print a list of discovered signals. It should work with any device that works with that block, including Realtek RTL2832U devices. This software was developed using a Compro U620F, which uses an E4000 tuner

{% img center /images/kali/wifi/gr-scan.png 'gr-scan' 'gr-scan' %}

**modes_gui**

Part of gr-air-modes

gr-air-modes implements a software-defined radio receiver for Mode S
transponder signals, including ADS-B reports from equipped aircraft.

**rtl_adsb**

A simple ADS-B decoder

{% img center /images/kali/wifi/rtl_adsb.png 'rtl_adsb' 'rtl_adsb' %}

**rtl_fm**

A simple narrow band FM demodulator for RTL2832 based DVB-T receivers

``` plain
Use:	rtl_fm -f freq [-options] [filename]
	-f frequency_to_tune_to [Hz]
	 (use multiple -f for scanning, requires squelch)
	 (ranges supported, -f 118M:137M:25k)
	[-s sample_rate (default: 24k)]
	[-d device_index (default: 0)]
	[-g tuner_gain (default: automatic)]
	[-l squelch_level (default: 0/off)]
	[-o oversampling (default: 1, 4 recommended)]
	[-p ppm_error (default: 0)]
	[-E sets lower edge tuning (default: center)]
	[-N enables NBFM mode (default: on)]
	[-W enables WBFM mode (default: off)]
	 (-N -s 170k -o 4 -A fast -r 32k -l 0 -D)
	filename (a '-' dumps samples to stdout)
	 (omitting the filename also uses stdout)

Experimental options:
	[-r output_rate (default: same as -s)]
	[-t squelch_delay (default: 20)]
	 (+values will mute/scan, -values will exit)
	[-M enables AM mode (default: off)]
	[-L enables LSB mode (default: off)]
	[-U enables USB mode (default: off)]
	[-R enables raw mode (default: off, 2x16 bit output)]
	[-F enables high quality FIR (default: off/square)]
	[-D enables de-emphasis (default: off)]
	[-C enables DC blocking of output (default: off)]
	[-A std/fast/lut choose atan math (default: std)]

Produces signed 16 bit ints, use Sox or aplay to hear them.
	rtl_fm ... - | play -t raw -r 24k -e signed-integer -b 16 -c 1 -V1 -
	             | aplay -r 24k -f S16_LE -t raw -c 1
	  -s 22.5k - | multimon -t raw /dev/stdin
```

**rtl_sdr**

An I/Q recorder for RTL2832 based DVB-T receivers

``` plain
Usage:	 -f frequency_to_tune_to [Hz]
	[-s samplerate (default: 2048000 Hz)]
	[-d device_index (default: 0)]
	[-g gain (default: 0 for auto)]
	[-b output_block_size (default: 16 * 16384)]
	[-n number of samples to read (default: 0, infinite)]
	[-S force sync output (default: async)]
	filename (a '-' dumps samples to stdout)
```

**rtlsdr-scanner**

A cross platform Python frequency scanning GUI for USB TV dongles, using the OsmoSDR rtl-sdr library.
In other words a cheap, simple Spectrum Analyser.
The scanner attempts to overcome the tuner’s frequency response by averaging scans from both the positive and negative frequency offets of the baseband data.

{% img center /images/kali/wifi/rtlsdr-scanner.png 'rtlsdr-scanner' 'rtlsdr-scanner' %}

**rtl_tcp**

An I/Q spectrum server for RTL2832 based DVB-T receivers

``` plain
Usage:	[-a listen address]
	[-p listen port (default: 1234)]
	[-f frequency to tune to [Hz]]
	[-g gain (default: 0 for auto)]
	[-s samplerate in Hz (default: 2048000 Hz)]
	[-b number of buffers (default: 32, set by library)]
	[-n max number of linked list buffers to keep (default: 500)]
	[-d device index (default: 0)]
```

**rtl_test**

A benchmark tool for RTL2832 based DVB-T receivers

``` plain
Usage:
	[-s samplerate (default: 2048000 Hz)]
	[-d device_index (default: 0)]
	[-t enable Elonics E4000 tuner benchmark]
	[-p enable PPM error measurement]
	[-b output_block_size (default: 16 * 16384)]
	[-S force sync output (default: async)]
```

> Noise proves nothing.  Often a hen who has merely laid an egg cackles
as if she laid an asteroid.

> -- Mark Twain






























  














