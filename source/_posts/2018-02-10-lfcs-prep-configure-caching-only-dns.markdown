---
layout: post
title: "LFCS prep - Configure caching-only DNS"
date: 2018-02-10 14:04:48 -0500
comments: true
categories: [linux, sysadmin]
keywords: dns, dns server, caching-only dns, dns zone, lfcs, rhcsa, lfcsa
description: How to configure a caching-only DNS server for the LFCS exam objectives
---

Today we'll take a look at setting up a caching only DNS server for the LFCS exam objectives.

<!-- more -->

First, install the necessary DNS packages: <code>yum install -y bind bind-utils</code>.

Then you will have to do some editing in <code>/etc/named.conf</code>. In particular, you are interested in the below:

``` 
listen-on port 53 { 127.0.0.1; any; };
allow-query     { localhost; any; };
allow-query-cache     { localhost; any; };
```

The *allow-query* option deals with who can send queries to the server, while the *allow-query-cache* allows access to cached records

You have to ensure the named.conf file has the proper permissions. It needs to be owned by root and belong to the group named:

``` 
ls -l /etc/named.conf
-rw-r-----. 1 root named 1754 Feb 10 19:46 /etc/named.conf
```

Check the SELinux contexts:

``` 
ls -lZ /etc/named.*
-rw-r-----. root named unconfined_u:object_r:etc_t:s0   /etc/named.conf
-rw-r--r--. root named system_u:object_r:etc_t:s0       /etc/named.iscdlv.key
-rw-r-----. root named system_u:object_r:named_conf_t:s0 /etc/named.rfc1912.zones
-rw-r--r--. root named system_u:object_r:etc_t:s0       /etc/named.root.key
```

Check the config file for syntax errors before trying anything:

``` 
named-checkconf /etc/named.conf
```

Start the DNS service:

``` 
systemctl start named
```

Open port 53 on the firewall:

``` 
firewall-cmd --add-port=53/udp
```

On the client, add the DNS server:

``` 
nmcli con mod ens33 ipv4.dns "192.168.217.131"
```

Restart the connection and NetworkManager. Check that the nameserver has been added in <code>/etc/resolv.conf</code>. Now you can test it:

``` 
nslookup github.com
Server:		192.168.217.131
Address:	192.168.217.131#53

Non-authoritative answer:
Name:	github.com
Address: 192.30.253.113
Name:	github.com
Address: 192.30.253.112
```

If you need to add a zone to your DNS server, take a look at the sample zone directives in */etc/named.rfc1912.zones*:

``` 
zone "localhost.localdomain" IN {
	type master;
	file "named.localhost";
	allow-update { none; };
};
```

You have to add a similar configuration for your zone inside <code>/etc/named.conf</code>. Then you also have to create a zone file inside **/var/named**. For reference, look at an existing one:

``` 
cat /var/named/named.localhost 
$TTL 1D
@	IN SOA	@ rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	NS	@
	A	127.0.0.1
	AAAA	::1
```

This should be all that is needed in terms of DNS configuration for LFCS objectives.

``` 
 ____________________________________
/ This life is yours. Some of it was \
| given to you; the rest, you made   |
\ yourself.                          /
 ------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

