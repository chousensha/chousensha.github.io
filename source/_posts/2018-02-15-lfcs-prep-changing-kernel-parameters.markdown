---
layout: post
title: "LFCS prep - Changing kernel parameters"
date: 2018-02-15 13:28:47 -0500
comments: true
categories: [linux, sysadmin]
keywords: kernel parameters, sysctl, /proc/sys, kernel runtime, lfcs, rhcsa, lfcsa
description: Modifying kernel parameters at runtime and persistently across boots
---

In this post we'll look at changing kernel parameters both at runtime and at boot. Such changes can be made with the **sysctl** command or by inputting values in different files inside <code>/proc/sys</code>

<!-- more -->

Let's see a sample /proc/sys folder, which can differ between systems:

``` 
ls /proc/sys
abi  crypto  debug  dev  fs  kernel  net  sunrpc  vm
```

The most important entries are:

* dev contains parameters for system devices

* fs contains filesystem parameters

* kernel is used for kernel parameters

* net contains network parameters

* vm contains virtual memory parameters

To modify runtime parameters, we can use **sysctl**. Let's take a look at all the available parameters:

``` 
sysctl -a | wc -l
951
```

Too many to list here, so let's glance over a few random ipv4 parameters:

``` 
sysctl -a | grep net.ipv4 | tail
net.ipv4.tcp_tso_win_divisor = 3
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_tw_reuse = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096    16384    4194304
net.ipv4.tcp_workaround_signed_windows = 0
net.ipv4.udp_mem = 43428    57905    86856
net.ipv4.udp_rmem_min = 4096
net.ipv4.udp_wmem_min = 4096
net.ipv4.xfrm4_gc_thresh = 32768
```

The dots in the options represent actual slashes in the directory structure under /proc/sys. Now let's take a value and modify it. I will use the packet forwarding feature for this example

``` 
cat /proc/sys/net/ipv4/ip_forward
1
```

So the system is configured to forward packets as a router. To change this, we can put a 0 value in the file:

``` 
echo 0 > /proc/sys/net/ipv4/ip_forward
```

Another option is to write a value with sysctl. Here I enable IP forwarding again:

``` 
sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
``` 

Changes made in this way won't persist a reboot. To make them permanent, you have to edit the configuration file.
This used to be <code>/etc/sysctl.conf</code> , and can still be used, but newer systems look for config files inside **/etc/sysctl.d/**, **/run/sysctl.d/**, and **/usr/lib/sysctl.d/** (in order of precedence). On such systems, you can modify the <code>/usr/lib/sysctl.d/00-system</code> file for your kernel settings.

I edited mine to disable IP forwarding:

``` 
# Disable IP forwarding
net.ipv4.ip_forward = 0
```

If you don't want to reboot, you can have sysctl re-read its config files by doing **sysctl -p** with the config file of your choice (by default, it will read **/etc/sysctl.conf**)

``` 
 sysctl -p /usr/lib/sysctl.d/00-system.conf 
net.ipv4.ip_forward = 0
cat /proc/sys/net/ipv4/ip_forward
0
```

``` 
 ______________________________________
/ You're definitely on their list. The \
| question to ask next is what list it |
\ is.                                  /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
