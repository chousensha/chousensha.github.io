---
layout: post
title: "LFCS prep - Creating custom yum repositories"
date: 2018-01-17 13:21:03 -0500
comments: true
categories: [linux, sysadmin]
keywords: yum, yum repo, yum repository, createrepo, lfcs, rhcsa, lfcsa
description: Creating yum repositories
---

Setting up a local package repository is useful when you need to save bandwidth or you won't have access to internet. In this post we'll set up a local yum repository from the DVD source, install vsftpd from it, and then host the repository over FTP.

<!-- more -->

## Making a local repository from DVD

I made a **/mnt/cdrom** directory where I can mount the DVD. Then check what you have in the drive:

``` 
blkid /dev/sr0
/dev/sr0: UUID="2016-10-19-18-32-06-00" LABEL="RHEL-7.3 Server.x86_64" TYPE="iso9660" PTTYPE="dos"
```

Mount the ISO:

``` 
mount -t iso9660 -o ro /dev/sr0 /mnt/cdrom
```

Copy the packages on the DVD to a directory:

``` 
mkdir -p /mnt/repos
cp -rpv /mnt/cdrom/Packages/ /mnt/repos/rhel7
```

Grab a cofee, this will take a while. When it's finished, create the repo from all the RPMS you copied:

``` 
createrepo /mnt/repos/rhel7
Spawning worker 0 with 4751 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
```

Next, we need to create a conf file for the repo. These files should be placed in <code>/etc/yum.repos.d</code> and be of the format *name.repo*. For a listing of the repository options that you can set, see the [RedHat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-configuring_yum_and_yum_repositories#sec-Setting_repository_Options)

I defined a repo file that looks like this:

``` 
cat /etc/yum.repos.d/rhel7-local.repo 
[rhel7-local]
name=Local RHEL7 repository
baseurl=file:///mnt/repos/rhel7
enabled=1
gpgcheck=0
```

* [name] - this is a unique repository ID, and it should contain no spaces

* name - a description of the repository

* baseurl - location of the repository data. For local files: <code>file:///path/to/local/repo</code>. For HTTP: <code>http://path/to/repo</code>. And for FTP: <code>ftp://path/to/repo</code>

* enabled - this 0 or 1 value specifies whether the repository is used by yum or not

* gpgcheck - this 0 or 1 value specifies if yum will do a GPG signature check on the packages

I cleared the yum cache and checked that my repo has been enabled:

``` 
yum clean all
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
Cleaning repos: rhel-7-server-rpms rhel-7-server-rt-beta-rpms rhel-7-server-rt-rpms rhel7-local
Cleaning up everything
yum repolist enabled | grep rhel7-local
rhel7-local                          Local RHEL7 repository               4,751
```

## Install vsftpd from local repo

I disabled the other repositories and installed vsftpd from my local one:

``` 
yum install -y vsftpd
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
Resolving Dependencies
--> Running transaction check
---> Package vsftpd.x86_64 0:3.0.2-21.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================================================
 Package                               Arch                                  Version                                        Repository                                  Size
=============================================================================================================================================================================
Installing:
 vsftpd                                x86_64                                3.0.2-21.el7                                   rhel7-local                                169 k

Transaction Summary
=============================================================================================================================================================================
Install  1 Package

Total download size: 169 k
Installed size: 348 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : vsftpd-3.0.2-21.el7.x86_64                                                                                                                                1/1 
  Verifying  : vsftpd-3.0.2-21.el7.x86_64                                                                                                                                1/1 

Installed:
  vsftpd.x86_64 0:3.0.2-21.el7                                                                                                                                               

Complete!
```

Start the FTP service and see that it's running:

``` 
systemctl start vsftpd && systemctl status vsftpd
● vsftpd.service - Vsftpd ftp daemon
   Loaded: loaded (/usr/lib/systemd/system/vsftpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2018-01-15 15:21:54 EET; 35ms ago
  Process: 6025 ExecStart=/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf (code=exited, status=0/SUCCESS)
 Main PID: 6026 (vsftpd)
   CGroup: /system.slice/vsftpd.service
           └─6026 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf

Jan 15 15:21:54 rhel7 systemd[1]: Starting Vsftpd ftp daemon...
Jan 15 15:21:54 rhel7 systemd[1]: Started Vsftpd ftp daemon.
```

If you have firewalld running, you have to allow vsftpd. First, chech the default zone:

``` 
firewall-cmd --get-default-zone
public
```

Check if FTP is allowed:

``` 
firewall-cmd --query-service=ftp
no
```

Add a rule to allow FTP:

``` 
firewall-cmd --zone=public --add-service=ftp --permanent
success
```

Restart the firewall and check again:

``` 
firewall-cmd --query-service=ftp
yes
```

Next, I copied the repository to the FTP folder in <code>/var/ftp/pub</code> and changed the ownership to ftp:

``` 
cp -r /mnt/repos/rhel7/ /var/ftp/pub/rhel7
chown -R ftp:ftp /var/ftp/pub/rhel7/
```

Check the SELinux context:

``` 
ls -lZ /var/ftp/pub/
dr-xr-xr-x. ftp ftp unconfined_u:object_r:public_content_t:s0 rhel7
```

And to see if everything works, FTP anonymously to the server and list the files:

``` 
ftp localhost
Trying ::1...
Connected to localhost (::1).
220 (vsFTPd 3.0.2)
Name (localhost:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||43989|).
150 Here comes the directory listing.
drwxr-xr-x    3 0        0              19 Jan 15 13:47 pub
226 Directory send OK.
ftp> ls pub
229 Entering Extended Passive Mode (|||35006|).
150 Here comes the directory listing.
dr-xr-xr-x    3 14       50         258048 Jan 15 13:49 rhel7
```

Now you can create another repo file like before, but this time served over FTP:

``` 
cat /etc/yum.repos.d/ftp-rhel7.repo 
[ftp-rhel7]
name=RHEL7 repository FTP
baseurl=ftp://127.0.0.1/pub/rhel7
enabled=1
gpgcheck=0
```

I disabled the local repository and only kept the FTP one:

``` 
yum repolist
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
ftp-rhel7                                                                                                                | 2.9 kB  00:00:00     
ftp-rhel7/primary_db                                                                                                     | 3.8 MB  00:00:00     
repo id                                                        repo name                                                                  status
ftp-rhel7                                                      RHEL7 repository FTP                                                       4,751
repolist: 4,751
```

Install something to see it work:

``` 
yum install -y squid
[...]
Installed:
  squid.x86_64 7:3.5.20-2.el7                                                                                                                   

Dependency Installed:
  libecap.x86_64 0:1.0.0-1.el7                          perl-Digest.noarch 0:1.17-245.el7          perl-Digest-MD5.x86_64 0:2.52-3.el7         
  squid-migration-script.x86_64 7:3.5.20-2.el7         

Complete!
```

``` 
 _____________________________________
/ Don't get stuck in a closet -- wear \
\ yourself out.                       /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```




