---
layout: post
title: "LFCS prep - Storage Management"
date: 2018-03-02 13:12:06 -0500
comments: true
categories: [linux, sysadmin]
keywords: linux, linux commands, linux storage, linux storage management, linux lvm, lvm, lfcs exam, lfcs, rhcsa, lfcsa
description: Commands for managing storage on Linux
---

Various commands for the LFCS Storage Management domain, which at the time of this post can be found at https://training.linuxfoundation.org/images/pdfs/LFCS_Domains_Competencies_V2.16.pdf

<!-- more -->

{% img center /images/sysadmin/storage.png 'linux storage' 'storage objectives' %}

{% img center /images/sysadmin/storage2.png 'linux storage' 'storage objectives' %}

#### mdadm

> manage MD devices aka Linux Software RAID

#### lsblk

> list block devices

```
lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0               2:0    1    4K  0 disk
sda               8:0    0   20G  0 disk
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 19.5G  0 part
  ├─centos-root 253:0    0 17.5G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0    1G  0 disk
sr0              11:0    1 1024M  0 rom  
```

* output filesystem info

```
lsblk -f
NAME         FSTYPE      LABEL UUID                                   MOUNTPOINT
fd0                                                                   
sda                                                                   
├─sda1       xfs               ef67363c-0fb5-4680-aa53-572e5e38d8d0   /boot
└─sda2       LVM2_member       0R3HDI-jqHC-WdoO-gDZ5-ScPb-JdkI-y9sIvD
  ├─centos-root
             xfs               71dd7ac4-ba5b-4e8a-a706-a51264d85900   /
  └─centos-swap
             swap              41edd5da-8142-4fc6-9e96-e8b7a08c2ae9   [SWAP]
sdb                                                                   
sr0                                                
```

#### fdisk

> manipulate disk partition table

* list partition tables

```
fdisk -l /dev/sda

Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000e124e

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    41943039    20458496   8e  Linux LVM
```

* fdisk menu

```
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)
```

#### gdisk

> Interactive GUID partition table (GPT) manipulator

```
gdisk /dev/sdb
GPT fdisk (gdisk) version 0.8.6

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): ?
b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu
```

#### parted

> a partition manipulation program

* list all partitions

```
parted -l
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  525MB   524MB   primary  xfs          boot
 2      525MB   21.5GB  20.9GB  primary               lvm


Error: /dev/sdb: unrecognised disk label
Model: VMware, VMware Virtual S (scsi)                                    
Disk /dev/sdb: 1074MB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags:

Model: Linux device-mapper (linear) (dm)
Disk /dev/mapper/centos-swap: 2147MB
Sector size (logical/physical): 512B/512B
Partition Table: loop
Disk Flags:

Number  Start  End     Size    File system     Flags
 1      0.00B  2147MB  2147MB  linux-swap(v1)


Model: Linux device-mapper (linear) (dm)
Disk /dev/mapper/centos-root: 18.8GB
Sector size (logical/physical): 512B/512B
Partition Table: loop
Disk Flags:

Number  Start  End     Size    File system  Flags
 1      0.00B  18.8GB  18.8GB  xfs
```

#### mkfs

> build a Linux filesystem

- -t # filesystem type

* types of filesystems

```
mkfs.
mkfs.btrfs   mkfs.ext2    mkfs.ext4    mkfs.minix   mkfs.vfat    
mkfs.cramfs  mkfs.ext3    mkfs.fat     mkfs.msdos   mkfs.xfs  
```

#### tune2fs

> adjust  tunable  filesystem  parameters  on  ext2/ext3/ext4 filesystems

* set label

``` 
tune2fs -L label device
```

#### e2label

> Change the label on an ext2/ext3/ext4 filesystem

#### dumpe2fs

> dump ext2/ext3/ext4 filesystem information

#### xfs_db

> debug an XFS filesystem

#### mount

> mount a filesystem

> mount -t type device dir

- -a # mount all filesystems
- -r # mount read-only
- -o # mount options


* list all mounted filesystems

```
mount -l
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=926484k,nr_inodes=231621,mode=755)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,seclabel,mode=755)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,seclabel,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_prio,net_cls)
configfs on /sys/kernel/config type configfs (rw,relatime)
/dev/mapper/rhel-root on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=33,pgrp=1,timeout=300,minproto=5,maxproto=5,direct)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
mqueue on /dev/mqueue type mqueue (rw,relatime,seclabel)
nfsd on /proc/fs/nfsd type nfsd (rw,relatime)
/dev/sda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw,relatime)
tmpfs on /run/user/0 type tmpfs (rw,nosuid,nodev,relatime,seclabel,size=188412k,mode=700)
gvfsd-fuse on /run/user/0/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=0,group_id=0)
fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
```

#### umount

> unmount file systems

#### blkid

> locate/print block device attributes

```
blkid
/dev/sda1: UUID="b8e2fb99-bf4c-4632-883b-8ed5c2350b1e" TYPE="xfs"
/dev/sda2: UUID="PLxZXl-aHBf-9VPQ-XOd1-eLMH-cWOW-NhjdcP" TYPE="LVM2_member"
/dev/mapper/rhel-root: UUID="2c0268ac-c31a-462d-b5b9-41bdfbecd75d" TYPE="xfs"
/dev/mapper/rhel-swap: UUID="6544d880-b634-4566-a68d-d3a58cec4cf6" TYPE="swap"
```

#### xfs_info

> expand an XFS filesystem

```
xfs_info /dev/sda1
meta-data=/dev/sda1              isize=512    agcount=4, agsize=65536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

#### mkswap

> set up a Linux swap area

#### swapon, swapoff

> enable/disable devices and files for paging and swapping

* display swap usage

```
swapon -s
Filename				Type		Size	Used	Priority
/dev/dm-1        
```

#### pvscan

> scan all disks for physical volumes

```
pvscan
  PV /dev/sda2   VG centos          lvm2 [19.51 GiB / 40.00 MiB free]
  Total: 1 [19.51 GiB] / in use: 1 [19.51 GiB] / in no VG: 0 [0   ]
```

#### vgscan

> scan all disks for volume groups and rebuild caches

```
vgscan
  Reading volume groups from cache.
  Found volume group "centos" using metadata type lvm2
```

#### lvscan

> scan (all disks) for Logical Volumes

```
lvscan
  ACTIVE            '/dev/centos/swap' [2.00 GiB] inherit
  ACTIVE            '/dev/centos/root' [17.47 GiB] inherit
```

#### pvcreate

> initialize a disk or partition for use by LVM

#### vgcreate

> create a volume group

#### vgs

> report information about volume groups

```
vgs
  VG     #PV #LV #SN Attr   VSize  VFree
  centos   1   2   0 wz--n- 19.51g 40.00m
```

#### lvcreate

> create a logical volume in an existing volume group

#### vgextend

> add physical volumes to a volume group

#### lvextend

> extend the size of a logical volume

#### lvremove

> remove a logical volume

#### pvmove

> move physical extents

#### vgreduce

> reduce a volume group

#### pvremove

> remove a physical volume

#### getfacl

> get file access control lists

```
getfacl out.txt
# file: out.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

#### setfacl

> set file access control lists

* give user dave read and write rights on a file

``` 
setfacl -m u:dave:rw file
```

* set default ACL recursively on a directory for a group and give read and directory traversal permissions (but no execute for files)

``` 
setfacl -R -m d:g:rockstars:rX folder
```

``` 
_________________________________________
/ Q: How many IBM types does it take to   \
| change a light bulb? A: Fifteen. One to |
| do it, and fourteen to write document   |
| number                                  |
|                                         |
| GC7500439-0001, Multitasking            |
| Incandescent Source System Facility,    |
|                                         |
| of which 10% of the pages state only    |
| "This page intentionally                |
|                                         |
| left blank", and 20% of the definitions |
| are of the form "A:.....                |
|                                         |
| consists of sequences of non-blank      |
\ characters separated by blanks".        /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
