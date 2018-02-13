---
layout: post
title: "LFCS prep - Configuring SGID directories"
date: 2018-02-13 12:39:42 -0500
comments: true
categories: [linux, sysadmin]
keywords: set group id, sgid, setgid, setgid directory, lfcs, rhcsa, lfcsa
description: Configure SGID directories
---

In this post we'll see how to configure SGID directories to share files between members of the same group.

<!-- more -->

I created a group called saiyans with a GID of 9000 to separate it from the rest of the groups:

``` 
groupadd -g 9000 saiyans
```

I made a directory that will be used to share files within the group:

``` 
mkdir /opt/invasion_plans
```

Changed the ownership of the directory to the nobody user and the saiyans group.

``` 
chown nobody:saiyans /opt/invasion_plans/
ls -ld /opt/invasion_plans/
drwxr-xr-x. 2 nobody saiyans 6 Feb 13 12:57 /opt/invasion_plans/
```

Set Group ID:

``` 
chmod g+s /opt/invasion_plans/
ls -ld /opt/invasion_plans/
drwxr-sr-x. 2 nobody saiyans 6 Feb 13 12:57 /opt/invasion_plans/
```

There are no write permissions yet, so have to assign them:

``` 
chmod g+w /opt/invasion_plans/
ls -ld /opt/invasion_plans/
drwxrwsr-x. 2 nobody saiyans 6 Feb 13 12:57 /opt/invasion_plans/
```

Removed the permissions for the other users:

``` 
chmod o-rwx /opt/invasion_plans/
ls -ld /opt/invasion_plans/
drwxrws---. 2 nobody saiyans 6 Feb 13 12:57 /opt/invasion_plans/
```

Now I added 2 users to the saiyans group:

``` 
useradd -G saiyans kakarot 
useradd -G saiyans vegeta
cat /etc/group | grep saiyans
saiyans:x:9000:kakarot,vegeta
```

And tested it:

``` 
su - kakarot
echo kamehameha > /opt/invasion_plans/boom
su - vegeta
echo 'final flash' > /opt/invasion_plans/bang
[kakarot@rhel7 ~]$ cat /opt/invasion_plans/bang 
final flash
[vegeta@rhel7 ~]$ cat /opt/invasion_plans/boom 
kamehameha
```

In case users need to be restricted from deleting files they didn't create, you can set the sticky bit with <code>chmod +t /opt/invasion_plans</code>

``` 
 ________________________________________
/ Your lucky number is 3552664958674928. \
\ Watch for it everywhere.               /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```




