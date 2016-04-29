---
layout: post
title: "Slackware install guide"
date: 2016-04-28 13:23:59 -0400
comments: true
categories: [linux, sysadmin]
keywords: slackware, linux, install, installation guide
description: How to install Slackware
---

Ever since I got into Linux and wanting to learn more and get better at it, I always held an interest towards Slackware. Being one of the oldest distributions around, with a hardcore community and an old-school reputation, it always came in the top answers when it comes to learning Linux without fancy hand holding and the like (along with Arch). But I always liked Slackware, its name is awesome, and well, slackwarez for a slacker! So I've finally set up some time to install it in a VM, and will get to work on it to deepen my Linux knowledge.

So in this post I will list the steps I went through to install Slackware 14.1 in VMware.

<!-- more -->

The cool thing about this installation was that I had to do some things manually without a GUI where all you do is press next. After selecting your installation media, which was the DVD ISO for me, power up the machine and you will get some command line action!

The first screen is for selecting which kernel to boot to

{% img center /images/sysadmin/slackware/bootkernel.png 'boot to kernel' 'kernel booting' %}

Select keyboard map

{% img center /images/sysadmin/slackware/keyboard.png 'keyboard map' 'select key map' %}

Log in as root

{% img center /images/sysadmin/slackware/root.png 'root login' 'root login' %}

### Partitioning

You have to set up partitions before beginning the installation process

{% img center /images/sysadmin/slackware/partition.png 'partition' 'partitioning' %}

I will use cfdisk for this. As you can see, my disk is /dev/sda and is yet yet unpartitioned, so just a big lump of free space. 

> cfdisk - display or manipulate a disk partition table

{% img center /images/sysadmin/slackware/cfdisk.png 'cfdisk' 'cfdisk' %}

I will make 3 partitions, for swap, root and home. Select [New] to create a new partition. The MBR partitioning scheme supports up to 4 primary partitions, and if you need more than that you can make one of them an extended partition and create  logical partitions inside it. Choose [Primary] to continue

{% img center /images/sysadmin/slackware/primary.png 'primary partition' 'primary partition' %}

Since this will be the swap partition, I will make it a size of 512 MB.

{% img center /images/sysadmin/slackware/size.png 'partition size' 'swap size' %}

Next I selected to place it at the beginning of the drive, for simplicity. I've also read that doing so might make it faster.

{% img center /images/sysadmin/slackware/beginning.png 'partition beginning' 'beginning partition' %}

Note how the first partition labeled sda1 was created. Next you have to choose the partition's type

{% img center /images/sysadmin/slackware/type.png 'partition type' 'partition type' %}

See how many different file systems can be created. Choose 82 for swap.

{% img center /images/sysadmin/slackware/swap.png 'type swap' 'swap partition' %}

Next make the root and home partitions, as described above. Only difference will be that you need to make the root partition bootable

{% img center /images/sysadmin/slackware/bootable.png 'bootable partition' 'root bootable' %}

Now write the changes to disk. You will be asked to confirm that you want to write the data

{% img center /images/sysadmin/slackware/write.png 'write partitions' 'write data' %}

After it's done you will see a message at the bottom: "Wrote partition table to disk". You can quit cfdisk now

### Setup

Type setup to begin the installation setup process 

{% img center /images/sysadmin/slackware/slacksetup.png 'slackware setup' 'install setup' %}

Choose ADDSWAP to format the swap partition that was created earlier. It will be automatically detected by the setup wizard

{% img center /images/sysadmin/slackware/swapsetup.png 'add swap' 'swap setup' %}

You can choose to check for bad blocks if you want, but I skipped it

{% img center /images/sysadmin/slackware/badblocks.png 'check bad blocks' 'bad blocks' %}

You will get a message when the swap space is configured

{% img center /images/sysadmin/slackware/swapdone.png 'swap conf' 'swap conf' %}

Next you have to choose the root partition, which in my case is sda2

{% img center /images/sysadmin/slackware/sda2root.png 'root partition' 'root sda2' %}

I selected the quick format option

{% img center /images/sysadmin/slackware/formatroot.png 'root format' 'quick format' %}

For the filesystem I chose ext4

{% img center /images/sysadmin/slackware/ext4.png 'ext4' 'ext4' %}

The last partition is the home one

{% img center /images/sysadmin/slackware/sda3home.png 'sda3 home' 'home' %}

After formatting and choosing its filesystem, you have to specify where you want it mounted. Type /home

{% img center /images/sysadmin/slackware/home.png 'mount home' 'home' %}

The setup of the partitions is now complete

{% img center /images/sysadmin/slackware/partitionsdone.png 'partitions complete' 'finished partitioning' %}

Next you have to choose the source media for the installation. In my case, it is the DVD

{% img center /images/sysadmin/slackware/sourcemedia.png 'source media' 'source dvd' %}

Let the wizard auto scan for the DVD

{% img center /images/sysadmin/slackware/autoscan.png 'autoscan' 'dvd scan' %}

Select the general packages that you want, I kept all except for the KDE ones, since I will be using XFCE for my GUI

{% img center /images/sysadmin/slackware/packages.png 'package selection' 'install packages' %}

I chose the full option for simplicity

{% img center /images/sysadmin/slackware/full.png 'full install' 'full' %}

After the installation process, you can create a boot stick if you want, but I skipped it

{% img center /images/sysadmin/slackware/bootdisk.png 'boot stick' 'boot disk' %}

The bootloader used by Slackware is LILO. I selected the simple install and the standard console

{% img center /images/sysadmin/slackware/lilo.png 'lilo' 'lilo' %}

{% img center /images/sysadmin/slackware/lilosplash.png 'lilo splash screen' 'lilo console' %}

Skip the extra parameters unless you know what you're doing

{% img center /images/sysadmin/slackware/kernelparams.png 'lilo extra parameters' 'lilo parameters' %}

You will next be prompted where to install they bootloader. I chose the MBR, since this is a VM dedicated to Slackware. But if this was on a dual booting system with Windows, you would want to install it on root

{% img center /images/sysadmin/slackware/lilombr.png 'lilo mbr' 'lilo install' %}

Choose your mouse type

{% img center /images/sysadmin/slackware/mouse.png 'mouse type' 'mouse' %}

The General Purpose Mouse software provides support for mouse devices in Linux virtual consoles.

{% img center /images/sysadmin/slackware/gpm.png 'general purpose mouse' 'gpm' %}

Next is the network configuration

{% img center /images/sysadmin/slackware/net.png 'network' 'net config' %}

Enter your hostname and domain

{% img center /images/sysadmin/slackware/hostname.png 'hostname' 'host' %}

{% img center /images/sysadmin/slackware/domain.png 'domain name' 'domain' %}

For simplicity, I chose the Network Manager configuration

{% img center /images/sysadmin/slackware/netconf.png 'network configuration' 'network config' %}

Confirm your choices before continuing

{% img center /images/sysadmin/slackware/netdone.png 'network setup' 'net setup' %}

I went with the default startup services. Will add more on a need-to-use basis

{% img center /images/sysadmin/slackware/startup.png 'startup services' 'startup' %}

You can try custom screen fonts if you want

{% img center /images/sysadmin/slackware/fonts.png 'custom fonts' 'screen fonts' %}

Next is the hardware clock and timezone

{% img center /images/sysadmin/slackware/hwclock.png 'hardware clock' 'clock' %}

{% img center /images/sysadmin/slackware/timezone.png 'timezone' 'timezone' %}

For the GUI, I went with XFCE

{% img center /images/sysadmin/slackware/gui.png 'gui' 'xfce' %}

You will be asked to choose a root password next

{% img center /images/sysadmin/slackware/pass.png 'root password' 'password' %}

With this, the installation setup is complete

{% img center /images/sysadmin/slackware/complete.png 'setup complete' 'setup finished' %}

Exit the wizard and reboot your brand new Slackware machine. Hit Enter when you see the splash screen, or it will boot automatically in a couple of minutes

{% img center /images/sysadmin/slackware/slacksplash.png 'lilo splash screen' 'lilo boot' %}

### Booting into your system

Slackware doesn't run the GUI automatically. You can change this by modifying the default runlevel

{% img center /images/sysadmin/slackware/textlogin.png 'text login' 'no gui' %}

I chose to manually start the GUI so I can read the random quotes that are given at login. You can load the GUI with the **startx** command

{% img center /images/sysadmin/slackware/finished.png 'slackware' 'slackware install' %}

All done! From here you can proceed to use your new distro, or customize it to your liking

``` plain
/ You will experience a strong urge to do \
\ good; but it will pass.                 /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```





