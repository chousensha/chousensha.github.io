---
layout: post
title: "Getting Pure-FTPd up and running"
date: 2014-06-09 23:20:06 +0300
comments: true
categories: [sysadmin, linux]
keywords: ftp, pure-ftpd, server, install pure-ftpd, pure-ftpd config, pure-ftpd tutorial, ftp tutorial, pure ftpd, pure ftpd config, pure ftpd commands
description: Setting up a Pure-FTPd server
---


In this tutorial I'll present the steps I took to set up Pure-FTPd on a Backtrack system. There might be slight differences for other distributions, particularly where file names and locations are concerned.
<!-- more -->
First, a brief description of Pure-FTPd, from the official documentation:

> Pure-FTPd is a fast, production-quality, standard-conformant FTP server, based upon 
> Troll-FTPd.

> Features include chroot()ed and/or virtual chroot()ed home directories, virtual 
> domains, built-in 'ls', anti warez system, configurable ports for passive 
> downloads, FXP protocol, bandwidth throttling, ratios, LDAP / MySQL / PostgreSQL-
> based authentication, fortune files, Apache-like log files, fast standalone mode, 
> text / HTML / XML real-time status report, virtual users, virtual quotas, privilege > separation, SSL/TLS and more.

Something to keep in mind is that, unlike other FTP servers, Pure-FTPd is controlled through command line arguments, rather than a configuration file. The latter is possible, but I won't cover it here.

Before going into the commands, I want to explain the concept of **virtual users**. These are FTP-only accounts that don't have to exit on the system, since they're used only for FTP. They have to be associated with a system user, so a good practice which I'll follow here is to create a dedicated system account just for FTP. Thousands of virtual users can share the same system user, as long as they all are chrooted and they have their own home directory.

Now, on to getting the server ready to..serve. First, create a group for FTP activities:

``` plain
groupadd ftpgroup
```

Now, create a system user that will be used for FTP only:

``` plain
useradd -g ftpgroup -d /dev/null -s /etc ftpuser
```

The name of the user is ftpuser. The -g flag associates it with the previously created group. Since I don't want the user to have a home directory or a login shell for security reasons, the -d option nukes the home directory by assigning it to /dev/null, and -s option sets /etc as a  fake shell.

Next, make a directory for FTP users:

``` plain
mkdir /home/ftpusers
```

Now we want to create our first virtual user. First, make a home directory for that user (note that there is a switch to automate this, but I use the manual approach here):

``` plain
mkdir /home/ftpusers/testuser
```

The **pure-pw** utility is what we'll use for managing virtual users. We could manually edit files instead, but who would want that when there's a nice, clean way?

``` plain
pure-pw useradd testuser -u ftpuser -d /home/ftpusers/testuser
```

This creates a virtual user named testuser, with the same UID as ftpuser, and that is chrooted to its home directory.

Next step is to commit all the virtual users changes to a PureDB file. Without committing changes to the database, the accounts won't be activated, and unusable.

``` plain
pure-pw mkdb
```

We then have to set up some symbolic links to add PureDB to the authentication methods:

``` plain
ln -s /etc/pure-ftpd/pureftpd.passwd /etc/pureftpd.passwd
ln -s /etc/pure-ftpd/pureftpd.pdb /etc/pureftpd.pdb
ln -s /etc/pure-ftpd/conf/PureDB /etc/pure-ftpd/auth/PureDB
```

If you don't need Unix and PAM authentication, you can do the following:

``` plain
echo no > /etc/pure-ftpd/conf/UnixAuthentication
echo no > /etc/pure-ftpd/conf/PAMAuthentication
```

Lastly, change the permissions of /home/ftpusers and its subdirectories to have ftpuser as owner and ftpgroup as group:

``` plain
chown -R ftpuser /home/ftpusers
chgrp -R ftpgroup /home/ftpusers
```

From here on there are a lot of options which you can use to customize your FTP server to suit your needs. As always, a good place to start looking for more advanced options is in the man pages and official documentation.

Cookie:

> You don't become a failure until you're satisfied with being one.

