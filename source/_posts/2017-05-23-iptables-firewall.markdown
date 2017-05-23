---
layout: post
title: "iptables firewall"
date: 2017-05-23 12:27:45 -0400
comments: true
categories: [linux, sysadmin]
keywords: iptables, iptables firewall, iptables tutorial, iptables examples, linux firewall, linux iptables, iptables linux
description: Intro to the iptables firewall
---



iptables is a CLI tool for configuring firewall functionality in Linux. It operates on a series of tables, which on a CentOS 7 system are:

<!-- more -->

* **filter** - the default table used for packet filtering

* **nat** - for nat

* **mangle** - specialized packet alteration

* **raw** - used mainly for configuring connection exemptions

* **security** - Mandatory Access Control networking rules

For the purpose of this post, we will be focusing on the filter table, which uses sets of rules to send (or not) packets on their merry way. These rules are called chains and they are as follows:

* INPUT - incoming connections

* OUTPUT - outbound connections

* FORWARD - packets being forwarded through the system

Let's see the currently configured rules:

``` plain
iptables -t filter -L | grep policy
Chain INPUT (policy ACCEPT)
Chain FORWARD (policy ACCEPT)
Chain OUTPUT (policy ACCEPT)
```

Well, everything is set to ACCEPT. The opposite of accepting connections is DROP, where the packets are silently dropped. Let's see it in action. First, change the policy to drop the packets: <code>iptables -P INPUT DROP</code>. Then try to ping the machine:

``` plain
ping 192.168.217.131

Pinging 192.168.217.131 with 32 bytes of data:
Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.217.131:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

It is possible to also use the REJECT extension,  where packets are being dropped but the source host receives an error, thereby being notified that there may be filtering in place: <code>iptables -I INPUT -j REJECT</code>. Here the REJECT was inserted at the beginning of the INPUT chain, to ensure that it will be matched before anything else:

``` plain
ping 192.168.217.131

Pinging 192.168.217.131 with 32 bytes of data:
Reply from 192.168.217.131: Destination port unreachable.
Reply from 192.168.217.131: Destination port unreachable.
Reply from 192.168.217.131: Destination port unreachable.
Reply from 192.168.217.131: Destination port unreachable.

Ping statistics for 192.168.217.131:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```

Some example scenarios:

### only allow SSH connections from specific IP

Let's assume that a computer has to be locked down, and only accept SSH connections from a certain IP:

``` plain
iptables -I INPUT -p tcp --dport 22 -s 192.168.217.137 -j ACCEPT
```

With a policy of dropping packets, running the above will allow TCP connections for port 22, from the 192.168.217.137 source address, while still denying everything else.

### traffic forwarding

Next, we have a machine with the IP 192.168.217.131 that we want to use to forward traffic to 192.168.217.137. How would we accomplish that?

First, we enable forwarding in the forwarding machine's kernel by putting a 1 inside <code>/proc/sys/net/ipv4/ip_forward</code>. On the .137 machine, I have a netcat listener on port 8000. On the .131 box, I also have netcat listening on port 4444. All connections coming to port 4444 on this machine will be routed to port 8000 on .137. The iptables rules to make that happen are:

* <code>iptables -t nat -A PREROUTING -p tcp --dport 4444 -j DNAT --to-destination 192.168.217.137:8000</code> - we operate on the nat table. PREROUTING is used for altering packets as soon as they  come  in. We append a rule to this chain, stating that for TCP packets coming to port 4444 on this host, the destination IP will be changed to 192.168.217.137, on port 8000

* <code>iptables -t nat -A POSTROUTING -j MASQUERADE</code> - next we append to the POSTROUTING chain, that alters packets as they are about to go out, telling iptables to masquerate packets: replacing the IP of the sender to the IP of the forwarding machine

Now I connect to port 4444 on 192.168.217.131 and send some random text, and checking my 192.168.217.137 listener, I see the traffic:

``` plain
c -vnlp 8000
listening on [any] 8000 ...
connect to [192.168.217.137] from (UNKNOWN) [192.168.217.131] 1859
dasa
knock kncok
```

### list rules of a table

* <code>iptables -L -n -v</code> - display the rules of the default filter table, also printing the number of packets and bytes processed by each chain, and use numerical format for ports and addresses

``` plain
Chain INPUT (policy ACCEPT 172 packets, 21210 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 37 packets, 1523 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 82 packets, 6554 bytes)
 pkts bytes target     prot opt in     out     source               destination   
```

### flush all the rules

* <code>iptables -F</code> - delete your rules and start anew   

### save rules to survive reboots

``` plain
service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
```

### block specific IP

* <code>iptables -A INPUT -s 192.168.217.137 -j DROP</code> - packets from .137 will be dropped

### block outgoing connections to a host or range

Let's imagine that you are tired of your users spending all their day on Twitter. First, find out Twitter's IPs:

``` plain
host twitter.com
twitter.com has address 104.244.42.65
twitter.com has address 104.244.42.1
twitter.com mail is handled by 20 alt1.aspmx.l.google.com.
twitter.com mail is handled by 30 aspmx2.googlemail.com.
twitter.com mail is handled by 20 alt2.aspmx.l.google.com.
twitter.com mail is handled by 10 aspmx.l.google.com.
twitter.com mail is handled by 30 aspmx3.googlemail.com.
```

Next, do a whois lookup on the IP, looking for the CIDR range it belongs to:

``` plain
hois 104.244.42.65 | grep CIDR
CIDR:           104.244.40.0/21
```

Block access to Twitter's IP range: <code>iptables -A OUTPUT -p tcp -d 66.220.144.0/20 -j DROP</code>

### log dropped packets

* <code>iptables -A INPUT -i eth0 -j LOG --log-prefix "Packets dropped by firewall:"</code> - turn on kernel logging for matching packets and prefix the log messages with some text to make them stand out


``` plain
root@pwnbox:~#grep "Packets dropped by firewall:" /var/log/messages
May 23 12:11:34 pwnbox kernel: [ 3285.154203] Packets dropped by firewall:IN=eth0 OUT= MAC=00:0c:29:22:f9:ae:00:50:56:ed:cb:c0:08:00 SRC=192.168.217.2 DST=192.168.217.137 LEN=288 TOS=0x00 PREC=0x00 TTL=128 ID=20858 PROTO=UDP SPT=53 DPT=57477 LEN=268
May 23 12:11:34 pwnbox kernel: [ 3285.158231] Packets dropped by firewall:IN=eth0 OUT= MAC=00:0c:29:22:f9:ae:00:50:56:ed:cb:c0:08:00 SRC=192.168.217.2 DST=192.168.217.137 LEN=356 TOS=0x00 PREC=0x00 TTL=128 ID=20859 PROTO=UDP SPT=53 DPT=36464 LEN=336
May 23 12:11:34 pwnbox kernel: [ 3285.162645] Packets dropped by firewall:IN=eth0 OUT= MAC=00:0c:29:22:f9:ae:00:50:56:ed:cb:c0:08:00 SRC=192.168.217.2 DST=192.168.217.137 LEN=360 TOS=0x00 PREC=0x00 TTL=128 ID=20860 PROTO=UDP SPT=53 DPT=47777 LEN=340
...
```

This post only scratched the tip of the iceberg when it comes to Linux firewalls. The key takeaway should be that iptables is a very powerful utility that can be customized to meet your specific networking needs.

``` plain
___________________________
< You will wish you hadn't. >
 ---------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

