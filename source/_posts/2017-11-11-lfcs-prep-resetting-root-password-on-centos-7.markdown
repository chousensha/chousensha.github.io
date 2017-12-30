---
layout: post
title: "LFCS prep - Resetting root password on CentOS 7"
date: 2017-11-11 16:53:18 -0500
comments: true
categories: [linux, sysadmin]
keywords: reset root password, centos root, centos reset, centos reset root password, lfcs, rhcsa, red hat reset root password, lfcsa
description: How to reset root password on CentOS 7
---


So, you got physical access to a machine, but you don't remember (or never knew) the root password. This post will walk you through how to reset the root password on a CentOS 7 system.

<!-- more -->

{% img center /images/sysadmin/rootreset/boot.png 'boot' 'edit grub boot entry' %}

At the boot menu, press **e** to edit the desired boot entry

{% img center /images/sysadmin/rootreset/initram.png 'initram' 'initram' %}

Now go to the end of the line that starts with **linux16** (you can quickly do it with **Ctrl-e** or **End** key, **Ctrl-a** or **Home** for the beginning of a line) and add <code>rd.break enforcing=0</code> to it. The first paramenter will interrupt the boot process to drop into a shell, and the second one sets SELinux to permissive mode, needed for a quicker way of resetting the password that doesn't require file relabeling at the end.

Also remove the **rhgb** and **quiet** parameters to see more debugging information. These are described in https://www.redhat.com/archives/rhl-list/2004-May/msg07775.html

> rhgb = redhat graphical boot - This is a GUI mode booting screen with most of the information hidden while the user 
> sees a rotating activity icon spining and brief information as to what the computer is doing.

> quiet = hides the majority of boot messages before rhgb starts. These are supposed to make the common user more 
> comfortable. They get alarmed about seeing the kernel and initializing messages, so they hide them for their comfort.

Next, press **Ctrl-x** to boot with the new options. You will be dropped into an initramfs switch_root shell.

{% img center /images/sysadmin/rootreset/initramfs.png 'initramfs switch_root' 'initramfs switch_root shell' %}

This could be confusing to someone new. Basically, initramfs is a temporary filesystem loaded into memory, that the kernel can access really early, in order to offload some startup tasks. It is also called early user space, and it takes care of steps that would be too cumbersome to include in the kernel, such as loading kernel modules and drivers needed for boot. This allows the kernel to remain relatively lean and not do by itself all the possible configurations. This initramfs image is a CPIO archive that gets unpacked into a tmpfs (a temporary filesystem stored in memory) and becomes the initial root filesystem.

Now, the filesystem is already mounted read-only inside <code>/sysroot</code>. To makey any changes, you have to mount it as read-write: <code>mount -o remount,rw /sysroot</code>

{% img center /images/sysadmin/rootreset/sysroot.png 'sysroot' 'sysroot mount' %}

Finally, change the root directory to /sysroot in order to run a shell from it: <code>chroot /sysroot</code>. Now your prompt will change to <code>sh-4.2#</code> and from here you can change the password normally, by using *passwd*. You should get the message that all authentication tokens updated successfully. Since you updated the password, you changed */etc/shadow* and now it won't be labeled correctly with a SELinux security context. You have to force a relabel of all the files at next boot, by creating an *.autorelabel* file: <code>touch /.autorelabel</code>.

Now type exit twice to exit the chroot environment first and then to exit the initramfs shell which will cause a reboot. The relabeling will take a while, and then you can use the system.

To skip the relabeling process, if you used *enforce=0*, you can restore the context of the shadow file with <code>restorecon /etc/shadow</code>. Then you have to remember to set SELinux back to enforcing, by running <code>setenforce 1</code>

Apparently, when you have an RHCSA exam, you start from getting a machine that you have to reset the root password for, which is really cool. Don't get caught unprepared!

### Learn more

https://www.certdepot.net/rhel7-interrupt-boot-gain-access-system/

[Kernel command line parameters](http://man7.org/linux/man-pages/man7/dracut.cmdline.7.html)

[switch_root and initramfs](https://unix.stackexchange.com/questions/126217/when-would-you-use-pivot-root-over-switch-root)

``` 
 _________________________________________
/ Be careful of reading health books, you \
| might die of a misprint.                |
|                                         |
\ -- Mark Twain                           /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

