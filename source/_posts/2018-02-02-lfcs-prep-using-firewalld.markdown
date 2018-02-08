---
layout: post
title: "LFCS prep - Using firewalld"
date: 2018-02-02 14:23:40 -0500
comments: true
categories: [linux, sysadmin]
keywords: firewalld, firewalld commands, lfcs, rhcsa, lfcsa
description: firewalld overview
---

Today we'll look at firewall configuration on CentOS / RedHat by using firewalld, the replacement for iptables. Since they are both mutually exclusive, if you decide to use firewalld, ensure that iptables is not running and cannot be started, by masking the service with <code>systemctl mask iptables</code>

<!-- more -->

With firewalld, traffic is classified into zones that can have their own rules and ports / services. The default zone is called the *public* zone

Now let's see a couple of commands for accomplishing various tasks.

* list the predefined zones

``` 
firewall-cmd --get-zones
work drop internal external trusted home dmz public block
```

* print the default zone

``` 
firewall-cmd --get-default-zone
public
```

* list active zones and their interfaces

``` 
firewall-cmd --get-active-zones
public
  interfaces: enp0s3
```

* set new default zone

``` 
firewall-cmd --set-default-zone=NAME
```

* list predefined services 

``` 
firewall-cmd --get-services 
RH-Satellite-6 amanda-client amanda-k5-client bacula bacula-client ceph ceph-mon dhcp dhcpv6 dhcpv6-client dns docker-registry dropbox-lansync freeipa-ldap freeipa-ldaps freeipa-replication ftp high-availability http https imap imaps ipp ipp-client ipsec iscsi-target kadmin kerberos kpasswd ldap ldaps libvirt libvirt-tls mdns mosh mountd ms-wbt mysql nfs ntp openvpn pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy proxy-dhcp ptp pulseaudio puppetmaster radius rpc-bind rsyncd samba samba-client sane smtp smtps snmp snmptrap squid ssh synergy syslog syslog-tls telnet tftp tftp-client tinc tor-socks transmission-client vdsm vnc-server wbem-https xmpp-bosh xmpp-client xmpp-local xmpp-server
```

* print information about the settings of the public zone

``` 
firewall-cmd --list-all --zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: dhcpv6-client ftp mountd nfs rpc-bind ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  sourceports: 
  icmp-blocks: 
  rich rules: 
```

* add a service

``` 
firewall-cmd --add-service samba
```

* to make changes persist through reboots, add the **--permanent** flag, and don't forget to reload after

* add multiple services

``` 
firewall-cmd --zone=NAME --add-service={serv1,serv2,serv3}
```

* add port

``` 
firewall-cmd add-port=8080/tcp
```

* add source to a zone

``` 
firewall-cmd --permanent --zone=NAME--add-source=RANGE
```

* add masquerade for a zone

``` 
firewall-cmd --zone=NAME --add-masquerade
```

* port forwarding (must enable masquerade first). Forward packets destined to port 22 to port 8888

``` 
firewall-cmd --zone=NAME --add-forward-port=port=22:proto=tcp:toport=8888
```

``` 
 ________________________________________
/ Your best consolation is the hope that \
| the things you failed to get weren't   |
\ really worth having.                   /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```




