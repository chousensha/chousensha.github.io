---
layout: post
title: "DHCP server on CentOS"
date: 2017-04-13 13:59:25 -0400
comments: true
categories: [linux, sysadmin]
keywords: dhcp, dhcp linux, dhcp server, dhcp centos, dhcp server linux, dhcp server centos, install dhcp server, configure dhcp server
description: Install and configure DHCP server on CentOS
---



In this post I will continue the series on configuring various servers on the CentOS 7 distribution. Let's see how we can get a DHCP server up and running!

<!-- more -->

## Installing DHCP server

First, we need to intall the server component, which can be done with the <code>yum install dhcp</code> command:

``` plain
=====================================================================================================================================================
 Package                       Arch                            Version                                           Repository                     Size
=====================================================================================================================================================
Installing:
 dhcp                          x86_64                          12:4.2.5-47.el7.centos                            base                          511 k

Transaction Summary
=====================================================================================================================================================
Install  1 Package

Total download size: 511 k
Installed size: 1.4 M
Is this ok [y/d/N]: y
Downloading packages:
dhcp-4.2.5-47.el7.centos.x86_64.rpm                                                                                           | 511 kB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : 12:dhcp-4.2.5-47.el7.centos.x86_64                                                                                                1/1
  Verifying  : 12:dhcp-4.2.5-47.el7.centos.x86_64                                                                                                1/1

Installed:
  dhcp.x86_64 12:4.2.5-47.el7.centos                                                                                                                 

Complete!
```

We now have a <code>/etc/dhcp/dhcpd.conf</code> file for configuring the server. There isn't much in it to start with, except pointers to the <code>dhcpd.conf</code> man page and a sample config file in <code>/usr/share/doc/dhcp*/dhcpd.conf.example</code>:

``` plain
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#

# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

# Use this to enble / disable dynamic dns updates globally.
#ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

# No service will be given on this subnet, but declaring it helps the
# DHCP server to understand the network topology.

subnet 10.152.187.0 netmask 255.255.255.0 {
}

# This is a very basic subnet declaration.

subnet 10.254.239.0 netmask 255.255.255.224 {
  range 10.254.239.10 10.254.239.20;
  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
}

# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.

subnet 10.254.239.32 netmask 255.255.255.224 {
  range dynamic-bootp 10.254.239.40 10.254.239.60;
  option broadcast-address 10.254.239.31;
  option routers rtr-239-32-1.example.org;
}

# A slightly different configuration for an internal subnet.
subnet 10.5.5.0 netmask 255.255.255.224 {
  range 10.5.5.26 10.5.5.30;
  option domain-name-servers ns1.internal.example.org;
  option domain-name "internal.example.org";
  option routers 10.5.5.1;
  option broadcast-address 10.5.5.31;
  default-lease-time 600;
  max-lease-time 7200;
}

# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.

host passacaglia {
  hardware ethernet 0:0:c0:5d:bd:95;
  filename "vmunix.passacaglia";
  server-name "toccata.fugue.com";
}

# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
host fantasia {
  hardware ethernet 08:00:07:26:c0:a5;
  fixed-address fantasia.fugue.com;
}

# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.

class "foo" {
  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
}

shared-network 224-29 {
  subnet 10.17.224.0 netmask 255.255.255.0 {
    option routers rtr-224.example.org;
  }
  subnet 10.0.29.0 netmask 255.255.255.0 {
    option routers rtr-29.example.org;
  }
  pool {
    allow members of "foo";
    range 10.17.224.10 10.17.224.250;
  }
  pool {
    deny members of "foo";
    range 10.0.29.10 10.0.29.230;
  }
}
```

This is how the config file looks like. We'll use this example as a basis for making our own. Copy the example file and name it **dhcpd.conf** file:

``` plain
cp /usr/share/doc/dhcp*/dhcpd.conf.example /etc/dhcp/dhcpd.conf
```

Now edit it and make changes according to your network:

``` plain
# option definitions common to all supported networks...
option domain-name "localdomain.com";
# DNS server address - look in your /etc/resolv.conf
option domain-name-servers 192.168.217.2;

default-lease-time 600;
max-lease-time 7200;

# declare your subnet config
subnet 192.168.217.0 netmask 255.255.255.0 {
  # range of IPs to serve
  range 192.168.217.10 192.168.217.20;
  # the address of the routers - look for the gateway address in the route -n # command (entry containing UG)
  option routers 192.168.217.2;
}

host kaliclient {
hardware ethernet 00:0c:29:22:f9:ae;
fixed-address 192.168.217.12;
}
```

Here I declared the subnet for which the server would handle addresses, and I reserved an IP address for a client. For more options, you can look at the <code>dhcpd-options</code> man page.

Time to start the server. First, verify that the <code>/var/lib/dhcpd/dhcpd.leases</code> file exists, otherwise you will need to create an empty one before starting the server with the command <code>systemctl start dhcpd</code>. I changed my VMs connection settings to host-only, and then looked at the new IP configuration:

``` plain
# ifconfig on the host running the DHCP server
inet 192.168.217.10  netmask 255.255.255.0  broadcast 192.168.217.255

# ifconfig on the kali client
inet 192.168.217.12  netmask 255.255.255.0  broadcast 192.168.217.255
```

Success! Our DHCP server kicked in and gave addresses to 2 machines on the network!


``` plain
/ F.S. Fitzgerald to Hemingway:        \
|                                      |
| "Ernest, the rich are different from |
| us." Hemingway:                      |
|                                      |
\ "Yes. They have more money."         /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

