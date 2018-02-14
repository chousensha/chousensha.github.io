---
layout: post
title: "LFCS prep - NTP configuration"
date: 2018-02-14 14:50:41 -0500
comments: true
categories: [linux, sysadmin]
keywords: ntp, chrony, ntp server, lfcs, rhcsa, lfcsa
description: Using chrony to manage time synchronization
---

In this post we'll look at NTP configuration by using chrony, which is installed by default on newer RedHat systems.

<!-- more -->

Ensure that the **chronyd** service is started and let's take a look at the current time setttings:

``` 
timedatectl
      Local time: Thu 2018-02-15 02:15:51 JST
  Universal time: Wed 2018-02-14 17:15:51 UTC
        RTC time: Wed 2018-02-14 17:15:52
       Time zone: Asia/Tokyo (JST, +0900)
     NTP enabled: no
NTP synchronized: no
 RTC in local TZ: no
      DST active: n/a
```

Now look inside chrony's config file, which is <code>/etc/chrony.conf</code>:

``` 
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server 0.rhel.pool.ntp.org iburst
server 1.rhel.pool.ntp.org iburst
server 2.rhel.pool.ntp.org iburst
server 3.rhel.pool.ntp.org iburst

# Ignore stratum in source selection.
stratumweight 0

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Enable kernel RTC synchronization.
rtcsync

# In first three updates step the system clock instead of slew
# if the adjustment is larger than 10 seconds.
makestep 10 3

# Allow NTP client access from local network.
#allow 192.168/16

# Listen for commands only on localhost.
bindcmdaddress 127.0.0.1
bindcmdaddress ::1

# Serve time even if not synchronized to any NTP server.
#local stratum 10

keyfile /etc/chrony.keys

# Specify the key used as password for chronyc.
commandkey 1

# Generate command key if missing.
generatecommandkey

# Disable logging of client accesses.
noclientlog

# Send a message to syslog if a clock adjustment is larger than 0.5 seconds.
logchange 0.5

logdir /var/log/chrony
#log measurements statistics tracking
```

There is a set of default servers configured, but you can add your own. You can also allow NTP client access in the local network by uncommenting or adding the **allow** directive with a range.

In the previous timedatectl output we saw that NTP and synchronization weren't enabled, so let's enable them:

``` 
timedatectl set-ntp 1
timedatectl | grep -i ntp
     NTP enabled: yes
NTP synchronized: no
```

We still have to manage the NTP synchronization. In the config file we have some server pools enabled, but we want to know exact names for the servers:

``` 
chronyc sources -v
210 Number of sources = 4

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^? ts1.sct.de                    0   8     0   10y     +0ns[   +0ns] +/-    0ns
^? greenstone-quarry.the-jad     0   8     0   10y     +0ns[   +0ns] +/-    0ns
^? 1b.ncomputers.org             0   8     0   10y     +0ns[   +0ns] +/-    0ns
^? shout.ovh                     0   8     0   10y     +0ns[   +0ns] +/-    0ns
```

Now we can pick a server to synchronize with:

``` 
ntpdate -vd ts1.sct.de
15 Feb 02:37:52 ntpdate[9511]: ntpdate 4.2.6p5@1.2349-o Wed Mar  1 09:00:52 UTC 2017 (1)
Looking for host ts1.sct.de and service ntp
host found : ts1.sct.de
transmit(193.141.27.1)
receive(193.141.27.1)
transmit(193.141.27.1)
receive(193.141.27.1)
transmit(193.141.27.1)
receive(193.141.27.1)
transmit(193.141.27.1)
receive(193.141.27.1)
server 193.141.27.1, port 123
stratum 2, precision -24, leap 00, trust 000
refid [193.141.27.1], delay 0.07736, dispersion 0.00209
transmitted 4, in filter 4
reference time:    de2ec48b.aba7efb5  Wed, Feb 14 2018 23:22:03.670
originate timestamp: de2ec847.3cbf5eb1  Wed, Feb 14 2018 23:37:59.237
transmit timestamp:  de2ef276.d94d8084  Thu, Feb 15 2018  2:37:58.848
filter delay:  0.07916  0.07736  0.08383  0.07954 
         0.00000  0.00000  0.00000  0.00000 
filter offset: -10799.6 -10799.6 -10799.6 -10799.6
         0.000000 0.000000 0.000000 0.000000
delay 0.07736, dispersion 0.00209
offset -10799.642908

15 Feb 02:37:58 ntpdate[9511]: step time server 193.141.27.1 offset -10799.642908 sec
```

To get more information about the NTP status, check the tracking:

``` 
chronyc tracking
Reference ID    : 193.141.27.1 (ts1.sct.de)
Stratum         : 3
Ref time (UTC)  : Wed Feb 14 14:41:11 2018
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +10799.637695312 seconds
RMS offset      : 10799.637695312 seconds
Frequency       : 0.000 ppm fast
Residual freq   : -1.456 ppm
Skew            : 1000000.000 ppm
Root delay      : 0.069487 seconds
Root dispersion : 1528.844971 seconds
Update interval : 0.0 seconds
Leap status     : Normal
```

You can see the server in the Reference ID and the Leap status that confirms the synchronization. Also re-check with timedatectl that NTP is working:

``` 
timedatectl | grep -i ntp
     NTP enabled: yes
NTP synchronized: yes
```

``` 
 ________________________________________
/ Give thought to your reputation.       \
| Consider changing name and moving to a |
\ new town.                              /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```




