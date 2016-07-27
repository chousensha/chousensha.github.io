---
layout: post
title: "CCNA Cheatsheet"
date: 2016-06-16 05:43:05 -0400
comments: true
categories: cisco, networking
keywords: cisco
description: draft
published: false
---

Intro

<!-- more -->

# ICND1

## General terms

**Internetwork** - 2 or more networks connected by a router

**Network segmentation** - breaking up a massive network into a number of smaller ones

**Collision domain** - a network segment where packet collisions can occur if 2 devices send data at the same time

**Broadcast domain** - a network segment where all devices can hear all the broadcasts in the domain and can reach other via the data link layer

**IOS** - Internetworking Operating  System

**Three-way handshake** - 1) Client sends a SYN packet to the server to request synchronization of the sequence numbers

2) Server sends back a SYN/ACK packet that acknowledges the request

3) Client sends an ACK packet and the connection is established

**Flow control** - control the amount of data sent by a sender to a receiver in order not to overwhelm the receiver's buffer capacity

**Windowing** - the amount of bytes a transmitter can send before receiving an ACK 

**CSMA/CD (Carrier Sense Multiple Access with Collision  Detection)** - Ethernet protocol that allows even bandwidth sharing and prevents 2 devices from simultaneous transmission on the same network

**CRC (cyclic redundancy check)** - Ethernet error detection 

**Attenuation** - loss of signal strength as it travels the wire (dB)

**Crosstalk** - cable signal interferance

**Fiber optics** - very fast, long distances, immune to interferance. *Single-mode* covers more distance than *multimode*

**Auto-detect** mechanism - checking the other port in order to determine the exchange speed

**Encapsulation** - adding protocol headers to data as it travels the OSI layers

**PDU** - protocol data unit

- Layer 1 = bits
- Layer 2 = frames
- Layer 3 = packets
- Layer 4 = segments
- Layer 5-7 = data

**Virtual circuit** - source and destination network socket address pair

**Gratuitous ARP** - an ARP request for a computer's own IP, to check if any other host on the network is using it. If a conflict is detected, the address is removed from the DHCP pool and isn't automatically reassigned until an admin resolves the issue

**Unicast** - communication between a single sender and a single receiver

**Multicast** - one-to-many or many-to-many, communication is simultaneously addressed to a group of receivers

**Subnet mask** - 32 bit value that splits the IP between network address and host address. The network bits are all set to 1, and the host bits are all set to 0

## Network devices

**Hubs** - layer 1 devices

- half-duplex

**Switches** - layer 2 devices, can break up collision domains

**Bridges** - same as switches

**Routers** - layer 3 devices, break up broadcast domains and also collision domains

*Router characteristics*:

* perform packet switching and packet filtering

* by default, don't forward broadcast or multicast packets

* internetwork communication

* path selection

* connect VLANs

* can provide QoS for specific traffic

Devices that operate at all 7 OSI layers:

* network management stations

* web / application servers

* servers

* network hosts

## Info

*Traffic congestion* in a LAN can be caused by:

* Too many hosts in a collision or broadcast domain

* broadcast storms

* too much multicast traffic

* low bandwidth

* using hubs

* ARP broadcasts

## The OSI Layer

*Benefits*:

* it divides and simplifies the communication process

* promotes industry standardization and allows for multiple vendors and different types of hardware and software

* layers are compartmentalized so that modifications to one layer won't affect the rest

**1) Physical layer**

- transmits bits

- physical medium characteristics (voltage, cables, etc.)

**2) Data link layer**

- frame encapsulation and error detection

- 2 sublayers: Media Access Control (MAC) works with the physical layer and Logical Link Control (LLC) that interacts with the network layer

- MAC addresses (48 bits)

*MAC address format*

{% img center /images/study/ccna/mac.PNG 'MAC address' 'MAC address' %}

- G/L (Global/Local bit)

- I/G (Individudal/Group bit)

**3) Network layer** 

- logical addressing between networks

*IP header*

{% img center /images/study/ccna/ip.png 'IP header' 'IP header' %}

**4) Transport layer**

- end-to-end transport

- if connection-oriented, it sets up virtual circuits by the "three-way handshake" and provides reliable transport with error correction and flow control (TCP)

*TCP header*

{% img center /images/study/ccna/tcpheader.jpg 'TCP header' 'TCP header' %}

- if connectionless, non-guaranteed delivery (UDP)

*UDP header*

{% img center /images/study/ccna/udp.gif 'UDP header' 'UDP header' %}

**5) Session layer**

- handles sessions and separates data between them

**6) Presentation layer**

- formats the data (translation, compression, encryption)

**7) Application layer**

- the user interface with which users interact with the computer

- establishes the status of the communication parties and the necessary resources

- provides all the different services (databases, file sharing, printing, etc.)

##The TCP/IP model

**1) Network interface layer**

- can also be called network access layer 
- includes the OSI physical and data link layers

**Internet layer**

- the equivalent of the OSI network layer

#### Common IP protocols

- ICMP, ARP, IP in IP (IP tunneling), TCP, UDP, EIGRP, OSPF, IPv6, GRE, L2TP

**Host-to-host layer**

- the equivalent of the OSI transport layer

**Application layer**

- includes the OSI session, presentation and application layers
- some often encountered protocols: telnet, ssh, ftp, tftp, snmp, http, https, ntp, dns, dhcp, bootp, apipa

#### DHCP address acquisition

1) The client sends a DHCP Discover broadcast on layers 2 and 3

2) The DHCP server that received the message sends back a unicast DHCP Offer

3) Client sends a DHCP Request message to confirm that the offerred address is still available

4) Server sends a DHCP Acknowledgment to end the exchange

**APIPA (Automatic Private IP Addressing)**

- Windows feature that allows a host to self-configure its IP and subnet mask when DHCP is unavailable
- address range: **169.254.0.1-169.254.255.254**
- default Class B subnet mask of 255.255.0.0
- having an Apipa address means there is a problem with DHCP


## Ethernet

- RJ45 connectors

The steps of data transmission on an Ethernet LAN are:

1) Check if any other computer is using the wire. If not, begin transmission

2) If a collision occurs, a jam signal informs all computers about it

3) A random backoff algorithm is triggered by the collision

4) Every computer stops transmitting until the backoff timer expires

5) After the timer, all computers have equal priority to transmit

**Ethernet frame format**

{% img center /images/study/ccna/etherframe.jpg 'Ethernet frame' 'Ethernet frame' %}

- it is illegal for a source address to be either a broadcast or multicast address

**Ethernet standards summary**

* 10Base-T (IEEE 802.3) - 10 Mbps Cat5 UTP, 100m

* 100Base-TX (IEEE 802.3u) - Fast Ethernet, 100 Mbps, Cat5, Cat5E, Cat6 UTP, 100m

* 100Base-FX (IEEE 802.3u) - Fast Ethernet over fiber, 412m

* 1000Base-CX (IEEE 802.3z) - Gigabit Ethernet, STP cable, 25m

* 1000Base-T (IEEE 802.3ab) - Gigabit Ethernet, Cat5 UTP, 100m

* 1000Base-SX (IEEE 802.3z) - Gigabit Ethernet over fiber, 220-550m

* 1000Base-LX (IEEE 802.3z) - Single-mode fiber Gigabit Ethernet, 3-10km

* 1000Base-ZX (Cisco standard) - Single-mode fiber Gigabit Ethernet, 70km

* 10GBase-T (802.3.an) - 10 Gbps, Cat5E, Cat6, Cat7 UTP, 100m

**Full duplex Ethernet**

- switch to switch
- host to host
- router to router
- switch to host
- switch to router
- router to host

- No collisions
- dedicated switch port
- switch and NIC must support full duplex

**Ethernet cables**

**Straight-through**

- host to switch / hub

- router to switch / hub

**Crossover**

- hub to hub
- switch to switch
- router to router
- host to host
- hub to switch
- router to host


**Rollover**

- host to Cisco router console 

Console emulation configuration: Bit Rate = 9600, Data Bits = 8, Parity = None, Flow Control = None

## The Cisco 3-layer model

**Core layer**

- switches large amounts of traffic at very high speed
- fault tolerance should be implemented at this layer as it needs high reliability
- avoid operations that slow down traffic, such as access lists, VLAN routing and packet filtering
- it is preferable to upgrade routers over adding new ones

**Distribution layer**

- sits between core and access layer
- implements security and network policies, packet filtering and queueing, address translation, firewalls, VLAN routing
- broadcast and multicast domains are defined here

**Access layer**

- where users access network resources
- network segmentation through separate collision domains

## Classful network addressing

### Class A
- *network.host.host.host*
- binary format: 0xxxxxxx
- 126 possible networks, 16,777,214 hosts per network
- network ID range: **1.0.0.0-126.0.0.0**

### Class B
- *network.network.host.host*
- binary format: 10xxxxxx 
- 16,384 networks, 65,534 hosts per network
- network ID range: **128.0.0.0-191.255.0.0**

### Class C
- *network.network.network.host*
- binary format: 110xxxxx
- 2,097,152 networks, 254 hosts per network
- network ID range: **192.0.0.0-223.255.255.0**

### Class D (multicast)
- 224-239

### Class E (research)
- 240-255

### Loopback
- 127.0.0.0 – 127.255.255.255

### Private IPs
- class A: **10.0.0.0 - 10.255.255.255**
- class B: **172.16.0.0 - 172.31.255.255**
- class C: **192.168.0.0 - 192.168.255.255**

## CIDR subnetting

- class A: *network.host.host.host*, default subnet mask: *255.0.0.0*
- class B: *network.network.host.host*, default subnet mask: *255.255.0.0*
- class C: *network.network.network.host*, default subnet mask: *255.255.255.0*

The **slash notation** tells you how many bits are turned on

Largest subnet mask is /30, but some devices may allow a /31

- /8 to /15 - class A
- /16 to /23 - class A, B
- /24 to /30 - class A, B, C

[Subnet reference table](https://en.wikipedia.org/wiki/IPv4_subnetting_reference)

{% img center /images/study/ccna/masks.PNG 'subnet masks' 'subnet masks reference table' %}

Subnetting process:

1) How many subnets -> **2^x**, x = no. of 1 bits in the mask

2) How many hosts per subnet -> **2^y - 2**, y = no. of 0 bits in the mask

3) Valid subnets -> 256 - subnet mask value = block size. To enumerate the subnets, count from 0 in block size to the subnet mask value

4) Broadcast address of each subnet -> address before next subnet

5) Valid hosts in each subnet -> all hosts between subnet address and broadcast address

Using the first and last subnet is generally discouraged

### Class C subnetting

Use 4th octet. Below is an example:

Address: 192.168.80.0, subnet mask: 255.255.255.128/25

128 in binary = 10000000, so there are 2^1 subnets (2), and 126 hosts / subnet (2^7 - 2)

256 - 128 = 128 block size, so the subnets are 192.168.80.**0** and 192.168.80.**128**

0 subnet broadcast address is 192.168.80.**127**, and hosts are between 192.168.80.**1** and 192.168.80.**126**

128 subnet broadcast address is 192.168.80.**255**, and hosts are between 192.168.80.**129** and 192.168.80.**254**

### Class B subnetting

Start with 3rd octet. Example:

Address: 172.16.0.0, subnet mask: 255.255.128.0/17

Now keep in mind both 128 (10000000) and 0 (00000000), which give us 2 subnets (2^1) and 32,766 hosts (2^15 - 2)

256 - 128 = 128 block size, so the subnets are 172.16.**0.0** and 172.16.**128.0**

0.0 subnet broadcast address is 172.16.**127.255** and hosts are between 172.16.**0.1** and 172.16.**127.254**

128.0 subnet broadcast address is 172.16.**255.255** and hosts are between 172.16.**128.1** and 172.16.**255.254**

### Class A subnetting

Start with 2nd octet. Example:

Adress: 10.0.0.0, subnet mask: 255.240.0.0/12

240 = 11110000, so 16 subnets (2^4), and 1,048,574 hosts (2^20 - 2)

256 - 240 = 16, so subnets are 0, 16, 32, 48, 64, 80, 96, 112, 128, 144, 160, 176, 192, 208, 224

0 subnet broadcast address is 10.**15.255.255** and hosts are between 10.**0.0.1** and 10.**15.255.254**

## IOS commands
- *ip domain-name* some_domain -  Specifies the DNS domain name.


