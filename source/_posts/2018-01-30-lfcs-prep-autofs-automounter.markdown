---
layout: post
title: "LFCS prep - autofs automounter"
date: 2018-01-30 12:57:31 -0500
comments: true
categories: [linux, sysadmin]
keywords: autofs, automounter, autofs linux, automounter linux, nfs, automount nfs, lfcs, rhcsa, lfcsa
description: Using autofs to mount shares automatically
---


In a [previous post](http://chousensha.github.io/blog/2017/09/10/nfs-shares-on-centos-7/) we looked at sharing folders with NFS. Now we take a step further to look at a client-only configuration that allows on demand mounting / unmounting of various filesystems. There is no need for /etc/fstab entries and resources are preserved better. The automounter is provided by the *autofs* package. After installing it, check that the autofs service has been started, before proceeding with the configuration.

<!-- more -->

The main configuration is done in the master map file, located at <code>/etc/auto.master</code>. Its format is:

``` 
<mount-point> <map-type> <options>
```

* **mount-point** = base location for the autofs filesystem to be mounted.  For indirect maps this directory will be created  (as  with  mkdir -p) and is removed when the autofs filesystem is umounted.

* **map-type** = map type used for this mount point. A map file can be given here 

* **options** = mount options 

Here are the contents of my */etc/auto.master* file:

``` 
/mnt    /etc/auto.share
```

The map file can have any name of your choosing. The auto.share file has the following format:

``` 
<mount point> <options> <location>
```

I made one with these values:

``` 
nfs-share -fstype=nfs 192.168.241.130:/var/nfs-share
```

The name refers to the autofs mount point. I didn't specify an absolute path, so the share will be mounted under the directory specified in the master map (/mnt in this case). I had a quick share served by an NFS server, and after all the above configuration, I restarted autofs and looked under /mnt:

``` 
ls /mnt/share
docs
```

Done! No need for manually adding entries to /etc/fstab and mounting them.

``` 
 _____________________________________
/ Change your thoughts and you change \
\ your world.                         /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```



