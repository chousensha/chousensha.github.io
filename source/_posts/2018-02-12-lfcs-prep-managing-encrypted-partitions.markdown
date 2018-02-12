---
layout: post
title: "LFCS prep - Managing encrypted partitions"
date: 2018-02-12 12:39:28 -0500
comments: true
categories: [linux, sysadmin]
keywords: luks, luks partitions, linux encrypt partitions, lfcs, rhcsa, lfcsa
description: Creating and mounting LUKS encrypted partitions
---

Today we'll go over creating LUKS-encrypted partitions with **cryptsetup**. LUKS (Linux Unified Key Setup) is a block device encryption format that is the standard on Linux systems. Also, because it stores all the necessary data in the partition header, it's easy to migrate partitions.

<!-- more -->

To get an overview of all the cryptographic ciphers that the system can use, look in <code>/proc/crypto</code>:

``` 
 cat /proc/crypto | grep name
name         : crc32
name         : __ghash
name         : ghash
name         : __ghash
name         : xts(aes)
name         : lrw(aes)
name         : __xts-aes-aesni
name         : __lrw-aes-aesni
name         : pcbc(aes)
name         : rfc4106(gcm(aes))
name         : __gcm-aes-aesni
name         : ctr(aes)
name         : __ctr-aes-aesni
name         : cbc(aes)
name         : __ecb-aes-aesni
name         : ecb(aes)
name         : __cbc-aes-aesni
name         : __ecb-aes-aesni
name         : __aes-aesni
name         : aes
name         : crct10dif
name         : crct10dif
name         : crc32c
name         : hmac(sha256)
name         : hmac(sha1)
name         : lzo
name         : crc32c
name         : aes
name         : sha224
name         : sha256
name         : sha1
name         : md5
name         : sha224
name         : sha256
name         : sha1
name         : aes
```

For this demo, I will be using a 200MB partition called /dev/sdb1. The below command initializes the partition as a LUKS device and you have to configure a passphrase at this step.

``` 
cryptsetup luksFormat /dev/sdb1

WARNING!
========
This will overwrite data on /dev/sdb1 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase: 
Verify passphrase: 
```

You can check out the header:

``` 
cryptsetup luksDump /dev/sdb1
LUKS header information for /dev/sdb1

Version:           1
Cipher name:       aes
Cipher mode:       xts-plain64
Hash spec:         sha256
Payload offset:    4096
MK bits:           256
MK digest:         6b 5e c2 bb 77 f9 0a 0d d4 67 4d 6f 01 a1 df 33 ce d7 94 b2 
MK salt:           42 9d d7 5f 3b 6c c1 0b 55 53 d1 fd 28 bd 73 27 
                   64 63 bb 70 02 de 33 46 0c 4f 2e 07 a9 a7 28 52 
MK iterations:     68500
UUID:              85b6a404-b24e-4c31-baf3-75cb3f041054

Key Slot 0: ENABLED
    Iterations:             547007
    Salt:                   4d 13 8b 20 74 cb 88 dd 8e dc b3 b0 2c ca 83 4b 
                              bd ab 12 2d 95 57 72 06 00 25 11 0e 43 b8 dc 81 
    Key material offset:    8
    AF stripes:                4000
Key Slot 1: DISABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

Next, create a mapping between the opened LUKS partition and the device mapper. 

``` 
cryptsetup --verbose luksOpen /dev/sdb1 hidden
Enter passphrase for /dev/sdb1: 
Key slot 0 unlocked.
Command successful.
```

View the mapping details:

``` 
ls -l /dev/mapper/hidden 
lrwxrwxrwx. 1 root root 7 Feb  9 18:29 /dev/mapper/hidden -> ../dm-2
```

If you want to have more insight into the status of the mapping:

``` 
cryptsetup -v status hidden
/dev/mapper/hidden is active.
  type:    LUKS1
  cipher:  aes-xts-plain64
  keysize: 256 bits
  device:  /dev/sdb1
  offset:  4096 sectors
  size:    405504 sectors
  mode:    read/write
Command successful.
```

Format the partition with the filesystem of your choice:

``` 
mkfs -t ext4 /dev/mapper/hidden
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=0 blocks, Stripe width=0 blocks
50800 inodes, 202752 blocks
10137 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=33816576
25 block groups
8192 blocks per group, 8192 fragments per group
2032 inodes per group
Superblock backups stored on blocks: 
    8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 
```

I made a folder for mounting the new encrypted partition and mounted it:

``` 
mount -v /dev/mapper/hidden /mnt/hidden
mount: /mnt/hidden does not contain SELinux labels.
       You just mounted an file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
mount: /dev/mapper/hidden mounted on /mnt/hidden.
```

When done working on it, you can unmount it:

``` 
umount -v /mnt/hidden
umount: /mnt/hidden (/dev/mapper/hidden) unmounted
```

And then remove the mapping:

``` 
cryptsetup --verbose luksClose hidden
Command successful.
```

To remount it, just go through the above steps again. If you want to mount it at boot, you need to take note of some UUIDS:

``` 
lsblk -o name,uuid,mountpoint
NAME          UUID                                   MOUNTPOINT
sda                                                  
├─sda1        b8e2fb99-bf4c-4632-883b-8ed5c2350b1e   /boot
└─sda2        PLxZXl-aHBf-9VPQ-XOd1-eLMH-cWOW-NhjdcP 
  ├─rhel-root 2c0268ac-c31a-462d-b5b9-41bdfbecd75d   /
  └─rhel-swap 6544d880-b634-4566-a68d-d3a58cec4cf6   [SWAP]
sdb                                                  
└─sdb1        85b6a404-b24e-4c31-baf3-75cb3f041054   
  └─hidden    ab98b274-9d34-41f0-9178-4f8c4b7e8a77   
sr0                                                  
```


Then you need to edit <code>/etc/crypttab</code>:

``` 
name device <password> <options>
```

Device refers to the block device or its UUID. The entries between tags are optional. In my case, the entry would look like this (the UUID is of /dev/sdb1)

``` 
cat /etc/crypttab 
hidden UUID=85b6a404-b24e-4c31-baf3-75cb3f041054
```

And edit /etc/fstab like you would for any other partition (UUID refers to the hidden mapping):

``` 
UUID=ab98b274-9d34-41f0-9178-4f8c4b7e8a77 /mnt/hidden ext4 defaults 0 0
```

Now you will be prompted at boot for the password. If you want to use a key for automatic unlocking, create a key file (here a random key of 4096 bytes length):

``` 
dd if=/dev/urandom of=/root/key bs=1024 count=4 
4+0 records in
4+0 records out
4096 bytes (4.1 kB) copied, 0.00100253 s, 4.1 MB/s
```

Make it only readable by root with <code>chmod 400 /root/key</code>

Add the key for the encrypted volume:

``` 
cryptsetup -v luksAddKey /dev/sdb1 /root/key
Enter any existing passphrase: 
Key slot 0 unlocked.
Key slot 0 unlocked.
Command successful.
```

Now the volume will be unlocked automatically without the need of entering a passphrase.

``` 
 _______________________________________
/ You definitely intend to start living \
\ sometime soon.                        /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
