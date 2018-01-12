---
layout: post
title: "LFCS prep - NetworkManager configuration"
date: 2018-01-11 13:31:50 -0500
comments: true
categories: [linux, sysadmin]
keywords: nmcli, nmcli tutorial, nmcli centos, network manager, networkmanager, networkmanager configuration, lfcs, rhcsa, lfcsa
description: Managing network settings with nmcli
---

Today we'll look over configuring various network settings with **nmcli**, the command line tool for NetworkManager.

<!-- more -->

The first step is to understand how network configuration settings look. Inside the <code>/etc/sysconfig/network-scripts/</code> directory, you can find config files for your network interfaces, under the format of **ifcfg-name**. On my system, it looks like this:

``` 
cat ifcfg-Wired_connection_1 
HWADDR=00:0C:29:47:D7:A8
TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME="Wired connection 1"
UUID=8d922966-5be4-4633-a99d-71e836e2b31a
ONBOOT=yes
DNS1=8.8.8.8
PEERDNS=yes
PEERROUTES=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
```

You can find a more complete overview of the options at https://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-networkscripts-interfaces.html , but from a glance at this file, we see the interface is an Ethernet device, it uses DHCP, it's set as the default route for both IPv4 and IPv6 traffic, it's activated at boot time, and also some other details like its MAC address and name.

nmcli is a very powerful tool, and you can press Tab to see options for its use at every stage. 

``` 
nmcli help
Usage: nmcli [OPTIONS] OBJECT { COMMAND | help }

OPTIONS
  -t[erse]                                   terse output
  -p[retty]                                  pretty output
  -m[ode] tabular|multiline                  output mode
  -c[olors] auto|yes|no                      whether to use colors in output
  -f[ields] <field1,field2,...>|all|common   specify fields to output
  -e[scape] yes|no                           escape columns separators in values
  -a[sk]                                     ask for missing parameters
  -s[how-secrets]                            allow displaying passwords
  -w[ait] <seconds>                          set timeout waiting for finishing operations
  -v[ersion]                                 show program version
  -h[elp]                                    print this help

OBJECT
  g[eneral]       NetworkManager's general status and operations
  n[etworking]    overall networking control
  r[adio]         NetworkManager radio switches
  c[onnection]    NetworkManager's connections
  d[evice]        devices managed by NetworkManager
  a[gent]         NetworkManager secret agent or polkit agent
  m[onitor]       monitor NetworkManager changes
```

Here we will go over a few examples:

## Device options

``` 
nmcli device help
Usage: nmcli device { COMMAND | help }

COMMAND := { status | show | set | connect | reapply | modify | disconnect | delete | monitor | wifi | lldp }

  status

  show [<ifname>]

  set [ifname] <ifname> [autoconnect yes|no] [managed yes|no]

  connect <ifname>

  reapply <ifname>

  modify <ifname> ([+|-]<setting>.<property> <value>)+

  disconnect <ifname> ...

  delete <ifname> ...

  monitor <ifname> ...

  wifi [list [ifname <ifname>] [bssid <BSSID>]]

  wifi connect <(B)SSID> [password <password>] [wep-key-type key|phrase] [ifname <ifname>]
                         [bssid <BSSID>] [name <name>] [private yes|no] [hidden yes|no]

  wifi hotspot [ifname <ifname>] [con-name <name>] [ssid <SSID>] [band a|bg] [channel <channel>] [password <password>]

  wifi rescan [ifname <ifname>] [[ssid <SSID to scan>] ...]

  lldp [list [ifname <ifname>]]
```

* show device status

``` 
nmcli dev status
DEVICE      TYPE      STATE      CONNECTION         
virbr0      bridge    connected  virbr0             
enp0s3      ethernet  connected  Wired connection 1 
lo          loopback  unmanaged  --                 
virbr0-nic  tun       unmanaged  --          
```

* list device details

``` 
nmcli dev show enp0s3 
GENERAL.DEVICE:                         enp0s3
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         00:0C:29:47:D7:A8
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     Wired connection 1
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveConnection/0
WIRED-PROPERTIES.CARRIER:               on
IP4.ADDRESS[1]:                         192.168.241.130/24
IP4.GATEWAY:                            192.168.241.2
IP4.DNS[1]:                             192.168.241.2
IP4.DNS[2]:                             8.8.8.8
IP4.DOMAIN[1]:                          localdomain
IP6.ADDRESS[1]:                         fe80::d7e8:7b48:831f:a1de/64
IP6.GATEWAY:                            
```

## Connection options

``` 
nmcli con help
Usage: nmcli connection { COMMAND | help }

COMMAND := { show | up | down | add | modify | clone | edit | delete | monitor | reload | load | import | export }

  show [--active] [--order <order spec>]
  show [--active] [id | uuid | path | apath] <ID> ...

  up [[id | uuid | path] <ID>] [ifname <ifname>] [ap <BSSID>] [passwd-file <file with passwords>]

  down [id | uuid | path | apath] <ID> ...

  add COMMON_OPTIONS TYPE_SPECIFIC_OPTIONS SLAVE_OPTIONS IP_OPTIONS [-- ([+|-]<setting>.<property> <value>)+]

  modify [--temporary] [id | uuid | path] <ID> ([+|-]<setting>.<property> <value>)+

  clone [--temporary] [id | uuid | path ] <ID> <new name>

  edit [id | uuid | path] <ID>
  edit [type <new_con_type>] [con-name <new_con_name>]

  delete [id | uuid | path] <ID>

  monitor [id | uuid | path] <ID> ...

  reload

  load <filename> [ <filename>... ]

  import [--temporary] type <type> file <file to import>

  export [id | uuid | path] <ID> [<output file>] 
```

* look at the available connections and list only the active ones

``` 
nmcli con show
NAME                UUID                                  TYPE            DEVICE 
Wired connection 1  8d922966-5be4-4633-a99d-71e836e2b31a  802-3-ethernet  enp0s3 
virbr0              321062a9-495a-4095-b4b5-d1af9c0cc65c  bridge          virbr0 
enp0s3              b4c32cc7-c5ff-451c-8af2-9b9124050f99  802-3-ethernet  --  
nmcli con show --active
NAME                UUID                                  TYPE            DEVICE 
Wired connection 1  8d922966-5be4-4633-a99d-71e836e2b31a  802-3-ethernet  enp0s3 
virbr0              321062a9-495a-4095-b4b5-d1af9c0cc65c  bridge          virbr0 
```

* list the settings of a particular connection (the double quotes aren't mandatory)

``` 
nmcli con show "Wired connection 1" 
connection.id:                          Wired connection 1
connection.uuid:                        8d922966-5be4-4633-a99d-71e836e2b31a
connection.stable-id:                   --
connection.interface-name:              --
connection.type:                        802-3-ethernet
connection.autoconnect:                 yes
connection.autoconnect-priority:        0
connection.timestamp:                   1515583502
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
ipv4.method:                            auto
ipv4.dns:                               8.8.8.8
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
ipv6.ip6-privacy:                       -1 (unknown)
ipv6.addr-gen-mode:                     stable-privacy
ipv6.dhcp-send-hostname:                yes
ipv6.dhcp-hostname:                     --
ipv6.token:                             --
GENERAL.NAME:                           Wired connection 1
GENERAL.UUID:                           8d922966-5be4-4633-a99d-71e836e2b31a
GENERAL.DEVICES:                        enp0s3
GENERAL.STATE:                          activated
GENERAL.DEFAULT:                        yes
GENERAL.DEFAULT6:                       no
GENERAL.VPN:                            no
GENERAL.ZONE:                           --
GENERAL.DBUS-PATH:                      /org/freedesktop/NetworkManager/ActiveConnection/0
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/Settings/0
GENERAL.SPEC-OBJECT:                    /
GENERAL.MASTER-PATH:                    --
IP4.ADDRESS[1]:                         192.168.241.130/24
IP4.GATEWAY:                            192.168.241.2
IP4.DNS[1]:                             192.168.241.2
IP4.DNS[2]:                             8.8.8.8
IP4.DOMAIN[1]:                          localdomain
DHCP4.OPTION[1]:                        requested_routers = 1
DHCP4.OPTION[2]:                        requested_domain_search = 1
DHCP4.OPTION[3]:                        requested_time_offset = 1
DHCP4.OPTION[4]:                        requested_domain_name = 1
DHCP4.OPTION[5]:                        requested_rfc3442_classless_static_routes = 1
DHCP4.OPTION[6]:                        requested_classless_static_routes = 1
DHCP4.OPTION[7]:                        requested_wpad = 1
DHCP4.OPTION[8]:                        requested_broadcast_address = 1
DHCP4.OPTION[9]:                        next_server = 192.168.241.254
DHCP4.OPTION[10]:                       expiry = 1515584826
DHCP4.OPTION[11]:                       requested_interface_mtu = 1
DHCP4.OPTION[12]:                       requested_subnet_mask = 1
DHCP4.OPTION[13]:                       domain_name = localdomain
DHCP4.OPTION[14]:                       dhcp_message_type = 5
DHCP4.OPTION[15]:                       ip_address = 192.168.241.130
DHCP4.OPTION[16]:                       routers = 192.168.241.2
DHCP4.OPTION[17]:                       requested_static_routes = 1
DHCP4.OPTION[18]:                       subnet_mask = 255.255.255.0
DHCP4.OPTION[19]:                       requested_domain_name_servers = 1
DHCP4.OPTION[20]:                       requested_nis_servers = 1
DHCP4.OPTION[21]:                       requested_ntp_servers = 1
DHCP4.OPTION[22]:                       domain_name_servers = 192.168.241.2
DHCP4.OPTION[23]:                       dhcp_lease_time = 1800
DHCP4.OPTION[24]:                       requested_ms_classless_static_routes = 1
DHCP4.OPTION[25]:                       broadcast_address = 192.168.241.255
DHCP4.OPTION[26]:                       requested_nis_domain = 1
DHCP4.OPTION[27]:                       network_number = 192.168.241.0
DHCP4.OPTION[28]:                       requested_host_name = 1
DHCP4.OPTION[29]:                       dhcp_server_identifier = 192.168.241.254
IP6.ADDRESS[1]:                         fe80::d7e8:7b48:831f:a1de/64
IP6.GATEWAY:                            
```

* adding connections

``` 
nmcli con add help
Usage: nmcli connection add { ARGUMENTS | help }

ARGUMENTS := COMMON_OPTIONS TYPE_SPECIFIC_OPTIONS SLAVE_OPTIONS IP_OPTIONS [-- ([+|-]<setting>.<property> <value>)+]

  COMMON_OPTIONS:
                  type <type>
                  ifname <interface name> | "*"
                  [con-name <connection name>]
                  [autoconnect yes|no]
                  [save yes|no]
                  [master <master (ifname, or connection UUID or name)>]
                  [slave-type <master connection type>]

  TYPE_SPECIFIC_OPTIONS:
    ethernet:     [mac <MAC address>]
                  [cloned-mac <cloned MAC address>]
                  [mtu <MTU>]

    wifi:         ssid <SSID>
                  [mac <MAC address>]
                  [cloned-mac <cloned MAC address>]
                  [mtu <MTU>]
                  [mode infrastructure|ap|adhoc]

    wimax:        [mac <MAC address>]
                  [nsp <NSP>]

    pppoe:        username <PPPoE username>
                  [password <PPPoE password>]
                  [service <PPPoE service name>]
                  [mtu <MTU>]
                  [mac <MAC address>]

    gsm:          apn <APN>
                  [user <username>]
                  [password <password>]

    cdma:         [user <username>]
                  [password <password>]

    infiniband:   [mac <MAC address>]
                  [mtu <MTU>]
                  [transport-mode datagram | connected]
                  [parent <ifname>]
                  [p-key <IPoIB P_Key>]

    bluetooth:    [addr <bluetooth address>]
                  [bt-type panu|dun-gsm|dun-cdma]

    vlan:         dev <parent device (connection UUID, ifname, or MAC)>
                  id <VLAN ID>
                  [flags <VLAN flags>]
                  [ingress <ingress priority mapping>]
                  [egress <egress priority mapping>]
                  [mtu <MTU>]

    bond:         [mode balance-rr (0) | active-backup (1) | balance-xor (2) | broadcast (3) |
                        802.3ad    (4) | balance-tlb   (5) | balance-alb (6)]
                  [primary <ifname>]
                  [miimon <num>]
                  [downdelay <num>]
                  [updelay <num>]
                  [arp-interval <num>]
                  [arp-ip-target <num>]
                  [lacp-rate slow (0) | fast (1)]

    bond-slave:   master <master (ifname, or connection UUID or name)>

    team:         [config <file>|<raw JSON data>]

    team-slave:   master <master (ifname, or connection UUID or name)>
                  [config <file>|<raw JSON data>]

    bridge:       [stp yes|no]
                  [priority <num>]
                  [forward-delay <2-30>]
                  [hello-time <1-10>]
                  [max-age <6-40>]
                  [ageing-time <0-1000000>]
                  [multicast-snooping yes|no]
                  [mac <MAC address>]

    bridge-slave: master <master (ifname, or connection UUID or name)>
                  [priority <0-63>]
                  [path-cost <1-65535>]
                  [hairpin yes|no]

    vpn:          vpn-type vpnc|openvpn|pptp|openconnect|openswan|libreswan|ssh|l2tp|iodine|...
                  [user <username>]

    olpc-mesh:    ssid <SSID>
                  [channel <1-13>]
                  [dhcp-anycast <MAC address>]

    adsl:         username <username>
                  protocol pppoa|pppoe|ipoatm
                  [password <password>]
                  [encapsulation vcmux|llc]

    tun:          mode tun|tap
                  [owner <UID>]
                  [group <GID>]
                  [pi yes|no]
                  [vnet-hdr yes|no]
                  [multi-queue yes|no]

    ip-tunnel:    mode ipip|gre|sit|isatap|vti|ip6ip6|ipip6|ip6gre|vti6
                  remote <remote endpoint IP>
                  [local <local endpoint IP>]
                  [dev <parent device (ifname or connection UUID)>]

    macvlan:      dev <parent device (connection UUID, ifname, or MAC)>
                  mode vepa|bridge|private|passthru|source
                  [tap yes|no]

    vxlan:        id <VXLAN ID>
                  remote <IP of multicast group or remote address>
                  [local <source IP>]
                  [dev <parent device (ifname or connection UUID)>]
                  [source-port-min <0-65535>]
                  [source-port-max <0-65535>]
                  [destination-port <0-65535>]

  SLAVE_OPTIONS:
    bridge:       [priority <0-63>]
                  [path-cost <1-65535>]
                  [hairpin yes|no]

    team:         [config <file>|<raw JSON data>]

  IP_OPTIONS:
                  [ip4 <IPv4 address>] [gw4 <IPv4 gateway>]
                  [ip6 <IPv6 address>] [gw6 <IPv6 gateway>]
```

* add a new DHCP connection that will start on boot

``` 
nmcli con add con-name newdhcp type ethernet ifname enp0s3 
Connection 'newdhcp' (8c90fcd7-5e40-43c6-819b-47cab3051b5e) successfully added.
nmcli con up newdhcp 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
nmcli con show --active 
NAME     UUID                                  TYPE            DEVICE 
newdhcp  8c90fcd7-5e40-43c6-819b-47cab3051b5e  802-3-ethernet  enp0s3 
virbr0   321062a9-495a-4095-b4b5-d1af9c0cc65c  bridge          virbr0
```

* add a static connection

``` 
nmcli con add con-name static-con ifname enp0s3 autoconnect no type ethernet ip4 192.168.241.120/24 gw4 192.168.241.2
Connection 'static-con' (258c2964-ac43-4ed8-bffa-7ad09967b748) successfully added.
```

Let's see the IP we started with:

``` 
ifconfig | grep inet
        inet 192.168.241.130  netmask 255.255.255.0  broadcast 192.168.241.255
```

And after switching to the static connection:

``` 
nmcli con up static-con 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
nmcli con show --active 
NAME        UUID                                  TYPE            DEVICE 
static-con  258c2964-ac43-4ed8-bffa-7ad09967b748  802-3-ethernet  enp0s3 
virbr0      321062a9-495a-4095-b4b5-d1af9c0cc65c  bridge          virbr0 
ifconfig | grep inet
        inet 192.168.241.120  netmask 255.255.255.0  broadcast 192.168.241.255
```

* modifying connections

``` 
nmcli con mod help
Usage: nmcli connection modify { ARGUMENTS | help }

ARGUMENTS := [id | uuid | path] <ID> ([+|-]<setting>.<property> <value>)+

Modify one or more properties of the connection profile.
The profile is identified by its name, UUID or D-Bus path. For multi-valued
properties you can use optional '+' or '-' prefix to the property name.
The '+' sign allows appending items instead of overwriting the whole value.
The '-' sign allows removing selected items instead of the whole value.

Examples:
nmcli con mod home-wifi wifi.ssid rakosnicek
nmcli con mod em1-1 ipv4.method manual ipv4.addr "192.168.1.2/24, 10.10.1.5/8"
nmcli con mod em1-1 +ipv4.dns 8.8.4.4
nmcli con mod em1-1 -ipv4.dns 1
nmcli con mod em1-1 -ipv6.addr "abbe::cafe/56"
nmcli con mod bond0 +bond.options mii=500
nmcli con mod bond0 -bond.options downdelay
```

* add a DNS server to an existing connection

``` 
nmcli con show static-con | grep ipv4.dns
ipv4.dns:                               
ipv4.dns-search:                        
ipv4.dns-options:                       (default)
ipv4.dns-priority:                      0
nmcli con mod static-con ipv4.dns 8.8.8.8
nmcli con up static-con
nmcli con show static-con | grep ipv4.dns
ipv4.dns:                               8.8.8.8
ipv4.dns-search:                        
ipv4.dns-options:                       (default)
ipv4.dns-priority:                      0
```

### Learn more

- [NetworkManager documentation](https://developer.gnome.org/NetworkManager/)
- [RHEL7 networking guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/)
- [CertDepot NetworkManager guide](https://www.certdepot.net/rhel7-get-started-nmcli/)

``` 
 ____________________________________
/ You're currently going through a   \
| difficult transition period called |
\ "Life."                            /
 ------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


