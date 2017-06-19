---
layout: post
title: "enum4linux - Kali Linux tools"
date: 2017-06-19 10:08:57 -0400
comments: true
categories: [penetration testing, tools]
keywords: enum4linux, enum4linux kali, enum4linux examples, enum4linux tutorial
description: enum4linux tutorial
---

**Objective**: enumerate shares on a target and gather as much information as possible. enum4linux is a Perl script that can get the job done.

Homepage: https://labs.portcullis.co.uk/tools/enum4linux/

<!-- more -->

## enum4linux description

> A Linux alternative to enum.exe for enumerating data from Windows and Samba hosts.

> Enum4linux is a tool for enumerating information from Windows and Samba systems. It attempts to offer similar 
> functionality to enum.exe formerly available from www.bindview.com.

> It is written in Perl and is basically a wrapper around the Samba tools smbclient, rpclient, net and nmblookup.

> Key features:

> * RID cycling (When RestrictAnonymous is set to 1 on Windows 2000)
    
> * User listing (When RestrictAnonymous is set to 0 on Windows 2000)
    
> * Listing of group membership information
    
> * Share enumeration
    
> * Detecting if host is in a workgroup or a domain
    
> * Identifying the remote operating system
    
> * Password policy retrieval (using polenum)

## enum4linux options

``` 
num4linux
enum4linux v0.8.9 (http://labs.portcullis.co.uk/application/enum4linux/)
Copyright (C) 2011 Mark Lowe (mrl@portcullis-security.com)

Simple wrapper around the tools in the samba package to provide similar 
functionality to enum.exe (formerly from www.bindview.com).  Some additional 
features such as RID cycling have also been added for convenience.

Usage: ./enum4linux.pl [options] ip

Options are (like "enum"):
    -U        get userlist
    -M        get machine list*
    -S        get sharelist
    -P        get password policy information
    -G        get group and member list
    -d        be detailed, applies to -U and -S
    -u user   specify username to use (default "")  
    -p pass   specify password to use (default "")   

The following options from enum.exe aren't implemented: -L, -N, -D, -f

Additional options:
    -a        Do all simple enumeration (-U -S -G -P -r -o -n -i).
              This opion is enabled if you don't provide any other options.
    -h        Display this help message and exit
    -r        enumerate users via RID cycling
    -R range  RID ranges to enumerate (default: 500-550,1000-1050, implies -r)
    -K n      Keep searching RIDs until n consective RIDs don't correspond to
              a username.  Impies RID range ends at 999999. Useful 
	      against DCs.
    -l        Get some (limited) info via LDAP 389/TCP (for DCs only)
    -s file   brute force guessing for share names
    -k user   User(s) that exists on remote system (default: administrator,guest,krbtgt,domain admins,root,bin,none)
              Used to get sid with "lookupsid known_username"
    	      Use commas to try several users: "-k admin,user1,user2"
    -o        Get OS information
    -i        Get printer information
    -w wrkg   Specify workgroup manually (usually found automatically)
    -n        Do an nmblookup (similar to nbtstat)
    -v        Verbose.  Shows full commands being run (net, rpcclient, etc.)

RID cycling should extract a list of users from Windows (or Samba) hosts 
which have RestrictAnonymous set to 1 (Windows NT and 2000), or "Network 
access: Allow anonymous SID/Name translation" enabled (XP, 2003).

NB: Samba servers often seem to have RIDs in the range 3000-3050.

Dependancy info: You will need to have the samba package installed as this 
script is basically just a wrapper around rpcclient, net, nmblookup and 
smbclient.  Polenum from http://labs.portcullis.co.uk/application/polenum/ 
is required to get Password Policy info.
```

## enum4linux usage

* get everything you can from a host without any credentials

``` 
enum4linux 192.168.217.131
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Mon Jun 19 10:06:33 2017

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 192.168.217.131
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ======================================================= 
|    Enumerating Workgroup/Domain on 192.168.217.131    |
 ======================================================= 
[E] Can't find workgroup/domain


 =============================================== 
|    Nbtstat Information for 192.168.217.131    |
 =============================================== 
Looking up status of 192.168.217.131
No reply from 192.168.217.131

 ======================================== 
|    Session Check on 192.168.217.131    |
 ======================================== 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 437.
[+] Server 192.168.217.131 allows sessions using username '', password ''
[+] Got domain/workgroup name: SAMBA

 ============================================== 
|    Getting domain SID for 192.168.217.131    |
 ============================================== 
Domain Name: SAMBA
Domain Sid: (NULL SID)
[+] Can't determine if host is part of domain or part of a workgroup

 ========================================= 
|    OS information on 192.168.217.131    |
 ========================================= 
[+] Got OS info for 192.168.217.131 from smbclient: Domain=[SAMBA] OS=[Windows 6.1] Server=[Samba 4.4.4]
[+] Got OS info for 192.168.217.131 from srvinfo:
	LOCALHOST      Wk Sv PrQ Unx NT SNT Samba File Server
	platform_id     :	500
	os version      :	6.1
	server type     :	0x809a03

 ================================ 
|    Users on 192.168.217.131    |
 ================================ 
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: smbuser	Name: Samba access is allowed for this user	Desc: 

user:[smbuser] rid:[0x3e8]

 ============================================ 
|    Share Enumeration on 192.168.217.131    |
 ============================================ 
WARNING: The "syslog" option is deprecated
Domain=[SAMBA] OS=[Windows 6.1] Server=[Samba 4.4.4]
Domain=[SAMBA] OS=[Windows 6.1] Server=[Samba 4.4.4]

	Sharename       Type      Comment
	---------       ----      -------
	sharename       Disk      Only authorized users
	IPC$            IPC       IPC Service (Samba File Server)

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------

[+] Attempting to map shares on 192.168.217.131
//192.168.217.131/sharename	Mapping: DENIED, Listing: N/A
//192.168.217.131/IPC$	Mapping: OK	Listing: DENIED

 ======================================================= 
|    Password Policy Information for 192.168.217.131    |
 ======================================================= 
[E] Unexpected error from polenum:
Traceback (most recent call last):
  File "/usr/bin/polenum", line 33, in <module>
    from impacket.dcerpc import dcerpc_v4, dcerpc, transport, samr
ImportError: cannot import name dcerpc_v4
[+] Retieved partial password policy with rpcclient:

Password Complexity: Disabled
Minimum Password Length: 5


 ================================= 
|    Groups on 192.168.217.131    |
 ================================= 

[+] Getting builtin groups:

[+] Getting builtin group memberships:

[+] Getting local groups:

[+] Getting local group memberships:

[+] Getting domain groups:

[+] Getting domain group memberships:

 ========================================================================== 
|    Users on 192.168.217.131 via RID cycling (RIDS: 500-550,1000-1050)    |
 ========================================================================== 
[I] Found new SID: S-1-22-1
[I] Found new SID: S-1-5-21-3832469351-2479326917-463392201
[I] Found new SID: S-1-5-32
[+] Enumerating users using SID S-1-5-21-3832469351-2479326917-463392201 and logon username '', password ''
S-1-5-21-3832469351-2479326917-463392201-500 *unknown*\*unknown* (8)
S-1-5-21-3832469351-2479326917-463392201-501 LOCALHOST\nobody (Local User)
[...]
S-1-5-21-3832469351-2479326917-463392201-513 LOCALHOST\None (Domain Group)
S-1-5-21-3832469351-2479326917-463392201-1000 LOCALHOST\smbuser (Local User)
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\nixhat (Local User)
S-1-22-1-1001 Unix User\smbuser (Local User)
[+] Enumerating users using SID S-1-5-32 and logon username '', password ''
S-1-5-32-500 *unknown*\*unknown* (8)
[...]
S-1-5-32-544 BUILTIN\Administrators (Local Group)
S-1-5-32-545 BUILTIN\Users (Local Group)
S-1-5-32-546 BUILTIN\Guests (Local Group)
S-1-5-32-547 BUILTIN\Power Users (Local Group)
S-1-5-32-548 BUILTIN\Account Operators (Local Group)
S-1-5-32-549 BUILTIN\Server Operators (Local Group)
S-1-5-32-550 BUILTIN\Print Operators (Local Group)


 ================================================ 
|    Getting printer info for 192.168.217.131    |
 ================================================ 
No printers returned.


enum4linux complete on Mon Jun 19 10:07:15 2017
```

I edited some not found output for sanity, but you can see that even without any previous information, we were able to gather quite a few pieces, like the workgroup name, the server version, and the existing shares and users

* perform some enumeration while also showing the commands being run

``` 
enum4linux -n -v 192.168.217.140
[V] Dependent program "nmblookup" found in /usr/bin/nmblookup
[V] Dependent program "net" found in /usr/bin/net
[V] Dependent program "rpcclient" found in /usr/bin/rpcclient
[V] Dependent program "smbclient" found in /usr/bin/smbclient
[V] Dependent program "polenum" found in /usr/bin/polenum
[V] Dependent program "ldapsearch" found in /usr/bin/ldapsearch
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Mon Jun 19 10:50:48 2017

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 192.168.217.140
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ======================================================= 
|    Enumerating Workgroup/Domain on 192.168.217.140    |
 ======================================================= 
[V] Attempting to get domain name with command: nmblookup -A '192.168.217.140'
[+] Got domain/workgroup name: WORKGROUP

 =============================================== 
|    Nbtstat Information for 192.168.217.140    |
 =============================================== 
Looking up status of 192.168.217.140
	WIN-D7GA2J1M0TU <00> -         M <ACTIVE>  Workstation Service
	WORKGROUP       <00> - <GROUP> M <ACTIVE>  Domain/Workgroup Name
	WIN-D7GA2J1M0TU <20> -         M <ACTIVE>  File Server Service
	WORKGROUP       <1e> - <GROUP> M <ACTIVE>  Browser Service Elections
	WORKGROUP       <1d> -         M <ACTIVE>  Master Browser
	..__MSBROWSE__. <01> - <GROUP> M <ACTIVE>  Master Browser

	MAC Address = 00-0C-29-5C-13-CA

 ======================================== 
|    Session Check on 192.168.217.140    |
 ======================================== 
[V] Attempting to make null session using command: smbclient -W 'WORKGROUP' //'192.168.217.140'/ipc$ -U''%'' -c 'help' 2>&1
[+] Server 192.168.217.140 allows sessions using username '', password ''

 ============================================== 
|    Getting domain SID for 192.168.217.140    |
 ============================================== 
[V] Attempting to get domain SID with command: rpcclient -W 'WORKGROUP' -U''%'' 192.168.217.140 -c 'lsaquery' 2>&1
could not initialise lsa pipe. Error was NT_STATUS_ACCESS_DENIED
could not obtain sid from server
error: NT_STATUS_ACCESS_DENIED
[+] Can't determine if host is part of domain or part of a workgroup
enum4linux complete on Mon Jun 19 10:50:48 2017
```

* get OS information

``` 
enum4linux -o 192.168.217.140
[...]
 ========================================= 
|    OS information on 192.168.217.140    |
 ========================================= 
[+] Got OS info for 192.168.217.140 from smbclient: Domain=[WIN-D7GA2J1M0TU] OS=[Windows 7 Ultimate 7601 Service Pack 1] Server=[Windows 7 Ultimate 6.1]
[E] Can't get OS info with srvinfo: NT_STATUS_ACCESS_DENIED
```

* list shares on a server with credentials

``` 
enum4linux -u Administrator -p Password123! -S 192.168.217.141
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Mon Jun 19 12:40:55 2017

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 192.168.217.141
RID Range ........ 500-550,1000-1050
Username ......... 'Administrator'
Password ......... 'Password123!'
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ======================================================= 
|    Enumerating Workgroup/Domain on 192.168.217.141    |
 ======================================================= 
[+] Got domain/workgroup name: SANGHELIOS0

 ======================================== 
|    Session Check on 192.168.217.141    |
 ======================================== 
[+] Server 192.168.217.141 allows sessions using username 'Administrator', password 'Password123!'

 ============================================== 
|    Getting domain SID for 192.168.217.141    |
 ============================================== 
Domain Name: SANGHELIOS0
Domain Sid: S-1-5-21-1024350911-1337957381-1412282408
[+] Host is part of a domain (not a workgroup)

 ============================================ 
|    Share Enumeration on 192.168.217.141    |
 ============================================ 
WARNING: The "syslog" option is deprecated
Domain=[SANGHELIOS0] OS=[Windows Server 2012 Datacenter 9200] Server=[Windows Server 2012 Datacenter 6.2]

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	ops             Disk      
	SYSVOL          Disk      Logon server share 
	testshare       Disk      A share on Windows Server
Connection to 192.168.217.141 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
NetBIOS over TCP disabled -- no workgroup available

[+] Attempting to map shares on 192.168.217.141
//192.168.217.141/ADMIN$	Mapping: OK, Listing: OK
//192.168.217.141/C$	[E] Can't understand response:
WARNING: The "syslog" option is deprecated
Domain=[SANGHELIOS0] OS=[Windows Server 2012 Datacenter 9200] Server=[Windows Server 2012 Datacenter 6.2]
  $Recycle.Bin                      DHS        0  Thu Jul 26 03:38:59 2012
  Boot                              DHS        0  Sun Oct 16 18:44:49 2016
  bootmgr                          AHSR   398156  Wed Jul 25 23:44:30 2012
  BOOTNXT                           AHS        1  Sat Jun  2 10:30:55 2012
  BOOTSECT.BAK                     AHSR     8192  Sun Oct 16 18:44:50 2016
  Documents and Settings            DHS        0  Thu Jul 26 03:14:09 2012
  pagefile.sys                      AHS 402653184  Mon Jun 19 12:08:41 2017
  PerfLogs                            D        0  Thu Jul 26 03:44:15 2012
  Program Files                      DR        0  Sun Oct 16 07:53:13 2016
  Program Files (x86)                 D        0  Thu Jul 26 04:04:58 2012
  ProgramData                        DH        0  Sun Oct 23 08:55:39 2016
  Recovery                          DHS        0  Sun Oct 16 07:48:22 2016
  StorageReports                      D        0  Sat Oct 22 17:17:58 2016
  System Volume Information         DHS        0  Sat Oct 22 17:17:27 2016
  testshare                           D        0  Mon Jun 19 12:25:50 2017
  Users                              DR        0  Sat Oct 22 16:06:28 2016
  Windows                             D        0  Sat Oct 22 16:49:16 2016

		7863807 blocks of size 4096. 5348507 blocks available
//192.168.217.141/IPC$	Mapping: OK	Listing: DENIED
//192.168.217.141/NETLOGON	Mapping: OK, Listing: OK
//192.168.217.141/ops	Mapping: OK, Listing: OK
//192.168.217.141/SYSVOL	Mapping: OK, Listing: OK
//192.168.217.141/testshare	Mapping: OK, Listing: OK
```

Once you're on the same network as your target, enum4linux is a great resource to help in gathering information about the target, that you can later use for an attack.

``` 
 ________________________________________
/ Your lucky number is 3552664958674928. \
\ Watch for it everywhere.               /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


