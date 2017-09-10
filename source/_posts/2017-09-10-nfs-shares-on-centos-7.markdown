---
layout: post
title: "NFS shares on CentOS 7"
date: 2017-09-10 07:45:30 -0400
comments: true
categories: [linux, sysadmin]
keywords: nfs, nfs shares, nfs centos, nfs centos 7, nfs4
description: Configure NFS shares on CentOS
---

Today we will go over an alternate way of setting up shares on CentOS 7. In an [earlier post](https://chousensha.github.io/blog/2017/06/09/quick-shares-with-samba-on-centos/) we saw how we can share stuff with Samba, which is the preferred way, especially if you have mixed environments. But today I also wanted to go through the process using the veteran: NFS! For this example, we'll be looking at the newest version, NFS4, which adds performance and security features, but also operates a little bit differently than its predecessors.

<!-- more -->

The Network File System allows clients to mount remote filesystems as if they were present locally. By default, the NFS server is listening on port 2049. In version 4, NFS requires the use of TCP.

### Server configuration

If you don't have it already, install the <code>nfs-utils</code> package. Next, start NFS with the command: <code>systemctl start nfs</code>. For the share, I created some directory and put a file in it:

```
mkdir /var/nfs-share
chmod 777 /var/nfs-share/
```

Now we have to edit the <code>/etc/exports</code> file, which contains the configuration for the shares. The format is as follows:

```
/sharepath client(options)
```

You can specify the client by hostname, domain name, IP address or network range. In this example, I will allow all hosts on my local subnet to have read write access to the share, and permit root users to retain their privileges:

``` 
/var/nfs-share 192.168.217.0/24(rw,no_root_squash)
```

Now restart the server for the configuration to take effect: <code>systemctl restart nfs</code>. You can see NFS statistics with **nfstat**:

``` 
nfsstat
Server rpc stats:
calls      badcalls   badclnt    badauth    xdrcall
91         0          0          0          0       

Server nfs v4:
null         compound     
2         2% 89       97% 

Server nfs v4 operations:
op0-unused   op1-unused   op2-future   access       close        commit       
0         0% 0         0% 0         0% 7         2% 1         0% 0         0% 
create       delegpurge   delegreturn  getattr      getfh        link         
0         0% 0         0% 1         0% 52       20% 8         3% 0         0% 
lock         lockt        locku        lookup       lookup_root  nverify      
0         0% 0         0% 0         0% 16        6% 0         0% 0         0% 
open         openattr     open_conf    open_dgrd    putfh        putpubfh     
1         0% 0         0% 0         0% 0         0% 63       25% 0         0% 
putrootfh    read         readdir      readlink     remove       rename       
4         1% 1         0% 2         0% 0         0% 0         0% 0         0% 
renew        restorefh    savefh       secinfo      setattr      setcltid     
0         0% 0         0% 0         0% 0         0% 0         0% 0         0% 
setcltidconf verify       write        rellockowner bc_ctl       bind_conn    
0         0% 0         0% 0         0% 0         0% 0         0% 0         0% 
exchange_id  create_ses   destroy_ses  free_stateid getdirdeleg  getdevinfo   
2         0% 2         0% 1         0% 0         0% 0         0% 0         0% 
getdevlist   layoutcommit layoutget    layoutreturn secinfononam sequence     
0         0% 0         0% 0         0% 0         0% 2         0% 83       33% 
set_ssv      test_stateid want_deleg   destroy_clid reclaim_comp 
0         0% 0         0% 0         0% 1         0% 2         0% 
```

To see the available exports and their options, use **exportfs**:

``` 
exportfs -v
/var/nfs-share	192.168.217.0/24(rw,wdelay,no_root_squash,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

### Client configuration

On the client, I first made a directory for the shares:

``` 
mkdir /mnt/nfs
```

To mount the share, the command looks like this:

``` 
mount -t nfs -o options server:/export /mount/directory
```

In my case, I mounted the exported share with:

``` 
mount -t nfs 192.168.217.131:/var/nfs-share /mnt/nfs/
```

Then I went to the directory and read the file:

``` 
root@kali:/mnt/nfs# ls
file.txt
root@kali:/mnt/nfs# cat file.txt 
something here
```

When done, unmount the filesystem:

``` 
umount /mnt/nfs
```

If instead of manually mounting the share, we would want it automatically mounted at boot, we'd have to edit <code>/etc/fstab</code>:

``` 
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
192.168.217.131:/var/nfs-share /mnt/nfs nfs rw 0 0
```

Now reboot the client and check that the share was mounted:

``` 
mount | grep nfs
192.168.217.131:/var/nfs-share on /mnt/nfs type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=0.0.0.0,local_lock=none,addr=192.168.217.131)
```

So, it's pretty easy and fast to set up NFS shares, but keep in mind this tutorial only scratched the surface of the available configuration options. 

``` 
 _______________________________________
/ Q: What's tiny and yellow and very,   \
| very, dangerous? A: A canary with the |
\ super-user password.                  /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```



