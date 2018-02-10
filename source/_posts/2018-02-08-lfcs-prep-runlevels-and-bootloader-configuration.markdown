---
layout: post
title: "LFCS prep - Runlevels and bootloader configuration"
date: 2018-02-08 12:47:30 -0500
comments: true
categories: [linux, sysadmin]
keywords: runlevels, systemd targets, bootloader, grub, grub2, lfcs, rhcsa, lfcsa
description: System runlevels and grub2 bootloader troubleshooting
---

Today we'll look at the following LFCS objectives:

* Log into graphical and text mode consoles

* Boot systems into different runlevels manually

* Install, configure and troubleshoot the  bootloader

<!-- more -->

## systemd targets

First, let's take a look at all the available targets:

```
systemctl list-units --type=target --all
  UNIT                   LOAD      ACTIVE   SUB    DESCRIPTION
  basic.target           loaded    active   active Basic System
  cryptsetup.target      loaded    active   active Encrypted Volumes
  emergency.target       loaded    inactive dead   Emergency Mode
  final.target           loaded    inactive dead   Final Step
  getty.target           loaded    active   active Login Prompts
  graphical.target       loaded    active   active Graphical Interface
  local-fs-pre.target    loaded    active   active Local File Systems (Pre)
  local-fs.target        loaded    active   active Local File Systems
  multi-user.target      loaded    active   active Multi-User System
  network-online.target  loaded    active   active Network is Online
  network-pre.target     loaded    inactive dead   Network (Pre)
  network.target         loaded    active   active Network
  nfs-client.target      loaded    active   active NFS client services
  nss-lookup.target      loaded    inactive dead   Host and Network Name Lookups
  nss-user-lookup.target loaded    active   active User and Group Name Lookups
  paths.target           loaded    active   active Paths
  remote-fs-pre.target   loaded    active   active Remote File Systems (Pre)
  remote-fs.target       loaded    active   active Remote File Systems
  rescue.target          loaded    inactive dead   Rescue Mode
  rpcbind.target         loaded    inactive dead   RPC Port Mapper
  shutdown.target        loaded    inactive dead   Shutdown
  slices.target          loaded    active   active Slices
  sockets.target         loaded    active   active Sockets
  swap.target            loaded    active   active Swap
  sysinit.target         loaded    active   active System Initialization
● syslog.target          not-found inactive dead   syslog.target
  time-sync.target       loaded    inactive dead   System Time Synchronized
  timers.target          loaded    active   active Timers
  umount.target          loaded    inactive dead   Unmount All Filesystems

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

29 loaded units listed.
To show all installed unit files use 'systemctl list-unit-files'.
```

Out of these, we will only look at the most relevant ones:

- **emergency.target** = emergency mode. All you have is systemd and a shell. This is what you use if, for instance, you experience disk failure

- **rescue.target** = single user mode. It's the equivalent of runlevel 1. In this mode, the early boot services are started and local mount points are mounted. No networking

- **multi-user.target** = multi-user text mode with networking. Equivalent to runlevel 3

- **graphical.target** = multi-user with GUI. Equivalent to runlevel 5

Let's see what the default target is on my system:

```
systemctl get-default
graphical.target
```

You can set a new default target with <code>systemctl set-default</code>

It is possible to switch to a different target without rebooting, by using <code>systemctl isolate <target name></code>. However, only targets with AllowIsolate=yes in their unit files can be isolated. Isolation will start or stop all necessary services for that particular target. You can use isolation to quickly switch between targets. You can achieve the same with the systemctl command:

```
systemctl rescue
systemctl emergency
```

Additionally, you can boot a desired target through the boot menu, by selecting *e* at the desired entry, and appending at the end of the kernel command line (the one starting with linux16) the entry **systemd.unit=<target name>**

## bootloader troubleshooting

Here we're only going to look at the grub2 bootloader. Its main configuration file is <code>/boot/grub2/grub.cfg</code>, but you're not supposed to edit it directly, but use <code>grub2-mkconfig</code> to generate the configuration. So you first edit the GRUB parameters in <code>/etc/default/grub</code>, and then run grub2-mkconfig to create the cfg file. Let's see the defaults I have:

```
cat /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

After editing those, you can run <code>grub2-mkconfig > /boot/grub2/grub.cfg</code> to create the new boot config file.

If you need to reinstall the bootloader, you can do so with <code>grub2-install</code>

### Learn more

[SysVinit to Systemd Cheatsheet](https://fedoraproject.org/wiki/SysVinit_to_Systemd_Cheatsheet)

[emergency mode vs rescue mode](https://lists.freedesktop.org/archives/systemd-devel/2016-February/035709.html)

``` 
 _________________________________________
/ You will always get the greatest        \
\ recognition for the job you least like. /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
