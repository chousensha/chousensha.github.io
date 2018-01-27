---
layout: post
title: "LFCS prep - LVM storage management"
date: 2018-01-27 09:56:20 -0500
comments: true
categories: [linux, sysadmin]
keywords: lvm, storage, lvm storage, lvm linux, lvm tutorial lfcs, rhcsa, lfcsa
description: Introduction to Logical Volume Management
---

Today we'll look at logical volume management tasks. LVM improves the traditional partition to disk approach, adding flexibility (extending and reducing space), snapshot capabilities and redundancy.

<!-- more -->

First, some terminology is in order:

* physical volumes (PV) are the physical devices that are mapped to be used inside LVM. Their storace unit is called a physical extent (PE)

* volume groups (VG) are storage pools made from PVs

* logical volumes (LV) are made up of the VG space 

## Creating PVs

I am now going to create 2 PVs of 100 MB each. You first need to create 2 LVM partitions. If you look inside fdisk, you see the partition code for LVM is **8e**:

``` 
8e  Linux LVM
```

So before writing the partition in fdisk, use <code>t</code> to change it to Linux LVM. Then verify that the new partitions were created:

``` 
lsblk | grep sdb
sdb             8:16   0    1G  0 disk 
├─sdb1          8:17   0  100M  0 part 
└─sdb2          8:18   0  100M  0 part 
```

Now the PVs can be created:

``` 
pvcreate /dev/sdb1 /dev/sdb2
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdb2" successfully created.
```

## Creating VGs

Now you can create a VG from the 2 PVs:

``` 
vgcreate vg-zero /dev/sdb1 /dev/sdb2
  Volume group "vg-zero" successfully created
```

Let's look at our existing PVs now:

``` 
pvs
  PV         VG      Fmt  Attr PSize  PFree 
  /dev/sda2  rhel    lvm2 a--  29.00g  4.00m
  /dev/sdb1  vg-zero lvm2 a--  96.00m 96.00m
  /dev/sdb2  vg-zero lvm2 a--  96.00m 96.00m
```

And check an individual PV in more detail:

``` 
pvdisplay /dev/sdb1
  --- Physical volume ---
  PV Name               /dev/sdb1
  VG Name               vg-zero
  PV Size               100.00 MiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              24
  Free PE               24
  Allocated PE          0
  PV UUID               ViTFfu-jBZE-47Ie-d6qB-WTDt-fpNf-bPC00Y
```

Now let's check our VGs:

``` 
vgs
  VG      #PV #LV #SN Attr   VSize   VFree  
  rhel      1   2   0 wz--n-  29.00g   4.00m
  vg-zero   2   0   0 wz--n- 192.00m 192.00m
```

And in more detail:

``` 
vgdisplay vg-zero 
  --- Volume group ---
  VG Name               vg-zero
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               192.00 MiB
  PE Size               4.00 MiB
  Total PE              48
  Alloc PE / Size       0 / 0   
  Free  PE / Size       48 / 192.00 MiB
  VG UUID               gUf1XI-znJt-MM2T-V721-PNbe-uxcr-m3bVTe
```

## Creating LVs

Next, I created a LV named inventory with a size of 128 MB, from the vg-zero VG:

``` 
lvcreate -n inventory -L 128MB vg-zero 
  Logical volume "inventory" created.
```

Look at the existing LVs:

``` 
lvs
  LV        VG      Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root      rhel    -wi-ao----  26.99g                                                    
  swap      rhel    -wi-ao----   2.00g                                                    
  inventory vg-zero -wi-a----- 128.00m     
```

And check our newly created one:

``` 
lvdisplay /dev/vg-zero/inventory 
  --- Logical volume ---
  LV Path                /dev/vg-zero/inventory
  LV Name                inventory
  VG Name                vg-zero
  LV UUID                jE4XRJ-ddlX-nM8s-NVSr-4cfK-F9O0-JcKecs
  LV Write Access        read/write
  LV Creation host, time rhel7, 2018-01-26 15:36:04 +0200
  LV Status              available
  # open                 0
  LV Size                128.00 MiB
  Current LE             32
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
```

Before using the new LV, you need to create a filesystem on top of it. I went with XFS:

``` 
mkfs.xfs /dev/vg-zero/inventory 
meta-data=/dev/vg-zero/inventory isize=512    agcount=4, agsize=8192 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=32768, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=855, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

Now prepare a location to mount it:

``` 
mkdir /mnt/inventory
```

I want it to mount at boot, so I added the following to <code>/etc/fstab</code>:

``` 
/dev/vg-zero/inventory     /mnt/inventory                xfs    defaults        0 0
```

Let's break down the **/etc/fstab** fields a bit:

* the first field represents the device, which can be specified by name, UUID, or label

* the second field is the mount point of the device

* the third is the filesystem type

* fourth is for the mount options. The *defaults* option specifies the usage of default options: rw, suid, dev, exec, auto, nouser, async

* the fifth is a backup flag for the dump utility. Use 1 to enable it and 0 to disable it

* sixth represents the fsck automatic check at boot. Use 0 to disable it, 1 for the root filesystem, and 2 for other filesystems that might need the automatic checking

It's a good practice to mount the devices you added to fstab before a reboot to ensure there are no errors: <code>mount -a</code>

## Extending / reducing VGs

To add more space to a VG, you can add more PVs to it. Let's see its current space:

``` 
vgdisplay vg-zero | grep Free
  Free  PE / Size       16 / 64.00 MiB
```

I created a new PV on /dev/sdb3 and added it to the VG:

``` 
vgextend vg-zero /dev/sdb3
  Volume group "vg-zero" successfully extended
```

The storage size has increased now:

``` 
vgdisplay vg-zero | grep Free
  Free  PE / Size       28 / 112.00 MiB
```

And to reduce the space of a VG, remove one or more PVs from it:

``` 
vgreduce vg-zero /dev/sdb3
  Removed "/dev/sdb3" from volume group "vg-zero"
```

To migrate the PEs used on a PV to other PVs in the VG, use **pvmove**:

``` 
pvmove /dev/sdb1
  /dev/sdb1: Moved: 0.00%
  /dev/sdb1: Moved: 100.00%
```

## Resizing LVs

You can use *lvresize* for extending or shrinking LVs, or *lvextend* for extending only. There needs to be enough free space in the VG for growing the LV.

``` 
lvextend -L +50M /dev/vg-zero/inventory 
  Rounding size to boundary between physical extents: 52.00 MiB.
  Size of logical volume vg-zero/inventory changed from 128.00 MiB (32 extents) to 180.00 MiB (45 extents).
  Logical volume vg-zero/inventory successfully resized.
```

You also need to grow the filesystem to occupy the newly extended LV. Since I picked XFS, I'll use *xfs_growfs* for this:

``` 
xfs_growfs /mnt/inventory/
meta-data=/dev/mapper/vg--zero-inventory isize=512    agcount=4, agsize=8192 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=32768, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=855, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 32768 to 46080
```

This was just for demo, in practice you want to resize both the LV and the filesystem with a single command:

``` 
lvextend -L 190M -r /dev/vg-zero/inventory 
  Rounding size to boundary between physical extents: 192.00 MiB.
  Size of logical volume vg-zero/inventory changed from 180.00 MiB (45 extents) to 192.00 MiB (48 extents).
  Logical volume vg-zero/inventory successfully resized.
meta-data=/dev/mapper/vg--zero-inventory isize=512    agcount=6, agsize=8192 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=46080, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=855, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 46080 to 49152
```

This time I specified an absolute size instead of adding a certain amount.

## Removing LVs

When you're done, you backed up your data or no longer need it, you can delete the LV. Unmount it first:

``` 
umount -v /mnt/inventory/
umount: /mnt/inventory (/dev/mapper/vg--zero-inventory) unmounted
```

Remove the LV:

``` 
lvremove /dev/vg-zero/inventory 
Do you really want to remove active logical volume vg-zero/inventory? [y/n]: y
  Logical volume "inventory" successfully removed
```

Remove the VG:

``` 
vgremove vg-zero 
  Volume group "vg-zero" successfully removed
```

Finally, also remove the PVs:

``` 
pvremove /dev/sdb1 /dev/sdb2
  Labels on physical volume "/dev/sdb1" successfully wiped.
  Labels on physical volume "/dev/sdb2" successfully wiped.
```

And don't forget to delete any remaining fstab entries.

``` 
 ___________________________________
/ That secret you've been guarding, \
\ isn't.                            /
 -----------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
