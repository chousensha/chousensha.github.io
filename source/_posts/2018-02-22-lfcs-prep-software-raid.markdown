---
layout: post
title: "LFCS prep - Software RAID"
date: 2018-02-22 13:37:45 -0500
comments: true
categories: [linux, sysadmin]
keywords: raid, mdadm, linux raid, software raid, mdadm linux, lfcs, rhcsa, lfcsa
description: Creating RAID devices with mdadm
---

Today we'll look at creating software RAID devices with *mdadm*. For the example I will use /dev/sdb1 and /dev/sdb2

<!-- more -->

To create an array:

``` 
mdadm --create --verbose /dev/md0 --level=mirror --raid-devices=2 /dev/sdb1 /dev/sdb2
mdadm: /dev/sdb1 appears to be part of a raid array:
       level=raid0 devices=0 ctime=Thu Jan  1 09:00:00 1970
mdadm: partition table exists on /dev/sdb1 but will be lost or
       meaningless after creating array
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 102272K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

```

To view more information about the array:

``` 
cat /proc/mdstat
Personalities : [raid1] 
md0 : active raid1 sdb2[1] sdb1[0]
      102272 blocks super 1.2 [2/2] [UU]
      
unused devices: <none>
```

For more detailed information:

``` 
mdadm --detail /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Tue Feb 20 00:17:49 2018
     Raid Level : raid1
     Array Size : 102272 (99.88 MiB 104.73 MB)
  Used Dev Size : 102272 (99.88 MiB 104.73 MB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

    Update Time : Tue Feb 20 00:17:53 2018
          State : clean 
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0

           Name : rhel7:0  (local to host rhel7)
           UUID : a92e04f3:1652f291:8c68266d:6c760648
         Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       18        1      active sync   /dev/sdb2
```

Now format the RAID device:

``` 
mkfs.ext4 /dev/md0
```

You need to create the config file <code>/etc/mdadm.conf</code> and add to it the output of the below command:

``` 
mdadm --detail --scan
ARRAY /dev/md0 metadata=1.2 name=rhel7:0 UUID=a92e04f3:1652f291:8c68266d:6c760648
```


See more information about the array disks:

``` 
mdadm --examine /dev/sdb1
/dev/sdb1:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : a92e04f3:1652f291:8c68266d:6c760648
           Name : rhel7:0  (local to host rhel7)
  Creation Time : Tue Feb 20 00:17:49 2018
     Raid Level : raid1
   Raid Devices : 2

 Avail Dev Size : 204640 (99.92 MiB 104.78 MB)
     Array Size : 102272 (99.88 MiB 104.73 MB)
  Used Dev Size : 204544 (99.88 MiB 104.73 MB)
    Data Offset : 160 sectors
   Super Offset : 8 sectors
   Unused Space : before=72 sectors, after=96 sectors
          State : clean
    Device UUID : 109087ab:0f2121e6:b6e09819:ffba2648

    Update Time : Tue Feb 20 00:22:17 2018
  Bad Block Log : 512 entries available at offset 72 sectors
       Checksum : f832b421 - correct
         Events : 17


   Device Role : Active device 0
   Array State : AA ('A' == active, '.' == missing, 'R' == replacing)
```

Mount the array and check it:

``` 
df -h /dev/md0
Filesystem      Size  Used Avail Use% Mounted on
/dev/md0         93M  1.6M   85M   2% /mnt/raid
```

Other useful commands:

* assemble array

``` 
mdadm --assemble /dev/md0
```

* remove disk from array (you have to mark it as failing first)

``` 
mdadm --fail /dev/md0 /dev/sdb2
mdadm --remove /dev/md0 /dev/sdb2
```

* add disk to array

``` 
mdadm --add /dev/md0 /dev/sdb2
```

* remove array

``` 
mdadm --stop /dev/md0
mdadm --remove /dev/md0
```

``` 
 _____________________________________
/ Don't relax! It's only your tension \
\ that's holding you together.        /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

