---
layout: post
title: "LFCS prep - Networking"
date: 2018-03-01 14:12:20 -0500
comments: true
categories: [linux, sysadmin]
keywords: linux, linux commands, linux networking, lfcs, rhcsa, lfcsa
description: Commands for managing Linux networking
---

Various commands for the LFCS Networking domain, which at the time of this post can be found at https://training.linuxfoundation.org/images/pdfs/LFCS_Domains_Competencies_V2.16.pdf

<!-- more -->

{% img center /images/sysadmin/network.png 'networking' 'linux networking' %}

#### hwclock

> query or set the hardware clock (RTC)

```
hwclock
Wed 18 Oct 2017 07:24:51 PM EEST  -1.042307 seconds
```

#### timedatectl

> Control the system time and date

```
timedatectl
      Local time: Wed 2017-10-18 17:26:49 EEST
  Universal time: Wed 2017-10-18 14:26:49 UTC
        RTC time: Wed 2017-10-18 17:26:49
       Time zone: Europe/Bucharest (EEST, +0300)
     NTP enabled: no
NTP synchronized: no
 RTC in local TZ: no
      DST active: yes
 Last DST change: DST began at
                  Sun 2017-03-26 02:59:59 EET
                  Sun 2017-03-26 04:00:00 EEST
 Next DST change: DST ends (the clock jumps one hour backwards) at
                  Sun 2017-10-29 03:59:59 EEST
                  Sun 2017-10-29 03:00:00 EET
```

#### date

> print or set the system date and time

```
[root@tron ~]# date
Tue Sep 19 12:40:58 EEST 2017
```

* get the date of a relative point in time

```
date -d "50 days"
Wed Nov  8 11:42:08 EET 2017
date -d "50 days ago"
Mon Jul 31 12:43:24 EEST 2017
```

#### ifconfig

> configure a network interface

- ifconfig <interface> up / down - activate / deactivate interface

- ifconfig <interface> promisc - enable / disable promiscuous mode
- ifconfig <interface> netmask addr - set network mask
- ifconfig <interface> address - assign IP to interface

#### ip

> show / manipulate routing, devices, policy routing and tunnels

* show information about all interfaces

```
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:47:d7:a8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.241.130/24 brd 192.168.241.255 scope global dynamic enp0s3
       valid_lft 1665sec preferred_lft 1665sec
    inet6 fe80::d7e8:7b48:831f:a1de/64 scope link
       valid_lft forever preferred_lft forever
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:e8:ed:ae brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:e8:ed:ae brd ff:ff:ff:ff:ff:ff
```

* show routing table

```
ip r
default via 192.168.241.2 dev enp0s3  proto static  metric 100
192.168.122.0/24 dev virbr0  proto kernel  scope link  src 192.168.122.1
192.168.241.0/24 dev enp0s3  proto kernel  scope link  src 192.168.241.130  metric 100
```

* show network interface statistics

```
ip -s link show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
    link/ether 00:0c:29:47:d7:a8 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    26927      137      0       0       0       0       
    TX: bytes  packets  errors  dropped carrier collsns
    112697     201      0       0       0       0       
```


#### hostnamectl

> Control the system hostname

```
hostnamectl
   Static hostname: tron
         Icon name: computer-vm
           Chassis: vm
        Machine ID: bae5d656813c4d17a18bc5ac0a89cfc2
           Boot ID: b0bbeef032cf4f6b9f4f764f9a043cb5
    Virtualization: vmware
  Operating System: Red Hat Enterprise Linux
       CPE OS Name: cpe:/o:redhat:enterprise_linux:7.3:GA:server
            Kernel: Linux 3.10.0-514.26.2.el7.x86_64
      Architecture: x86-64
```

To make hostname changes permanent, modify the content of **/etc/hostname**. You can set it with **hostnamectl set-hostname name**

#### nmcli

> command-line tool for controlling NetworkManager

```
nmcli -p
virbr0: connected to virbr0
	bridge, 52:54:00:E8:ED:AE, sw, mtu 1500
	inet4 192.168.122.1/24

enp0s3: connected to Wired connection 1
	"Intel 82545EM Gigabit Ethernet Controller (Copper) (PRO/1000 MT Single Port Adapter)"
	ethernet (e1000), 00:0C:29:47:D7:A8, hw, mtu 1500
	ip4 default
	inet4 192.168.241.130/24
	inet6 fe80::d7e8:7b48:831f:a1de/64

lo: unmanaged
	loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

virbr0-nic: unmanaged
	tun, 52:54:00:E8:ED:AE, sw, mtu 1500
```

* show information about interface

```
nmcli -p connection show enp0s3
===============================================================================
                      Connection profile details (enp0s3)
===============================================================================
connection.id:                          enp0s3
connection.uuid:                        b4c32cc7-c5ff-451c-8af2-9b9124050f99
connection.stable-id:                   --
connection.interface-name:              enp0s3
connection.type:                        802-3-ethernet
connection.autoconnect:                 yes
connection.autoconnect-priority:        0
connection.timestamp:                   1499955175
connection.read-only:                   no
connection.permissions:                 
connection.zone:                        --
connection.master:                      --
connection.slave-type:                  --
connection.autoconnect-slaves:          -1 (default)
connection.secondaries:                 
connection.gateway-ping-timeout:        0
connection.metered:                     unknown
connection.lldp:                        -1 (default)
-------------------------------------------------------------------------------
802-3-ethernet.port:                    --
802-3-ethernet.speed:                   0
802-3-ethernet.duplex:                  --
802-3-ethernet.auto-negotiate:          yes
802-3-ethernet.mac-address:             00:0C:29:47:D7:A8
802-3-ethernet.cloned-mac-address:      --
802-3-ethernet.generate-mac-address-mask:--
802-3-ethernet.mac-address-blacklist:   
802-3-ethernet.mtu:                     auto
802-3-ethernet.s390-subchannels:        
802-3-ethernet.s390-nettype:            --
802-3-ethernet.s390-options:            
802-3-ethernet.wake-on-lan:             1 (default)
802-3-ethernet.wake-on-lan-password:    --
-------------------------------------------------------------------------------
ipv4.method:                            auto
ipv4.dns:                               
ipv4.dns-search:                        
ipv4.dns-options:                       (default)
ipv4.dns-priority:                      0
ipv4.addresses:                         
ipv4.gateway:                           --
ipv4.routes:                            
ipv4.route-metric:                      -1
ipv4.ignore-auto-routes:                no
ipv4.ignore-auto-dns:                   no
ipv4.dhcp-client-id:                    --
ipv4.dhcp-timeout:                      0
ipv4.dhcp-send-hostname:                yes
ipv4.dhcp-hostname:                     --
ipv4.dhcp-fqdn:                         --
ipv4.never-default:                     no
ipv4.may-fail:                          yes
ipv4.dad-timeout:                       -1 (default)
-------------------------------------------------------------------------------
ipv6.method:                            auto
ipv6.dns:                               
ipv6.dns-search:                        
ipv6.dns-options:                       (default)
ipv6.dns-priority:                      0
ipv6.addresses:                         
ipv6.gateway:                           --
ipv6.routes:                            
ipv6.route-metric:                      -1
ipv6.ignore-auto-routes:                no
ipv6.ignore-auto-dns:                   no
ipv6.never-default:                     no
ipv6.may-fail:                          yes
ipv6.ip6-privacy:                       0 (disabled)
ipv6.addr-gen-mode:                     stable-privacy
ipv6.dhcp-send-hostname:                yes
ipv6.dhcp-hostname:                     --
ipv6.token:                             --
-------------------------------------------------------------------------------
```

#### route

> show / manipulate the IP routing table

```
route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.241.2   0.0.0.0         UG    100    0        0 enp0s3
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
192.168.241.0   0.0.0.0         255.255.255.0   U     100    0        0 enp0s3
```


#### firewall-cmd

> firewalld command line client

```
firewall-cmd --get-default-zone
public
```

* enumerate predefined zones

```
firewall-cmd --get-zones
work drop internal external trusted home dmz public block
```

#### tracepath,  tracepath6

 >traces path to a network host discovering MTU along this path

#### netstat

>  Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships

* show listening TCP ports in numbers

```
netstat -lnt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 ::1:631                 :::*                    LISTEN     
tcp6       0      0 ::1:25                  :::*                    LISTEN  
```

* list interfaces

```
netstat -i
'Kernel Interface table
Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
enp0s3    1500      139      0      0 0           203      0      0      0 BMRU
lo       65536       68      0      0 0            68      0      0      0 LRU
virbr0    1500        0      0      0 0             0      0      0      0 BMU
```

* statistics

```
netstat -s
Ip:
    190 total packets received
    0 forwarded
    0 incoming packets discarded
    180 incoming packets delivered
    216 requests sent out
    65 dropped because of missing route
Icmp:
    3 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
        timeout in transit: 2
        echo requests: 1
    1 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        echo replies: 1
IcmpMsg:
        InType8: 1
        InType11: 2
        OutType0: 1
Tcp:
    34 active connections openings
    0 passive connection openings
    34 failed connection attempts
    0 connection resets received
    0 connections established
    68 segments received
    68 segments send out
    0 segments retransmited
    0 bad segments received.
    34 resets sent
Udp:
    104 packets received
    0 packets to unknown port received.
    0 packet receive errors
    143 packets sent
    0 receive buffer errors
    0 send buffer errors
UdpLite:
TcpExt:
    0 packet headers predicted
IpExt:
    InNoRoutes: 3
    InMcastPkts: 18
    OutMcastPkts: 26
    InBcastPkts: 12
    InOctets: 31730
    OutOctets: 105262
    InMcastOctets: 4252
    OutMcastOctets: 5054
    InBcastOctets: 4212
    InNoECTPkts: 190
```

#### ss

> another utility to investigate sockets

* show statistics

``` 
ss -s
Total: 1114 (kernel 1133)
TCP:   0 (estab 0, closed 0, orphaned 0, synrecv 0, timewait 0/0), ports 0

Transport Total     IP        IPv6
*	  1133      -         -        
RAW	  1         0         1        
UDP	  1         1         0        
TCP	  0         0         0        
INET	  2         1         1        
FRAG	  0         0         0        
```

* show all TCP sockets

``` 
ss -t -a
State      Recv-Q Send-Q Local Address:Port                 Peer Address:Port                
ESTAB      0      0      192.168.217.132:40382                87.243.6.137:http
```

``` 
 _______________________________________
/ Q: Why did the programmer call his    \
| mother long distance? A: Because that |
\ was her name.                         /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
