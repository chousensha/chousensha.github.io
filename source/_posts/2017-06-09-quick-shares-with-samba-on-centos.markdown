---
layout: post
title: "Quick shares with Samba on CentOS"
date: 2017-06-09 05:47:05 -0400
comments: true
categories: [linux, sysadmin]
keywords: samba, samba centos, centos samba, samba linux, samba shares, samba sharing, samba tutorial, samba server
description: Configure Samba shares on CentOS
---


The interwebz is thundering with doomsday predictions about the [Samba CVE-2017-7494 exploit](https://blog.qualys.com/securitylabs/2017/05/26/samba-vulnerability-cve-2017-7494), and here I am, deciding that now is the best time to make a post on setting up Samba shares! xD

On a related note, if for some reason you can't patch the vulnerability yet, there is a workaround (with some drawbacks). Edit the global section in smb.conf and add the line <code>nt pipe support = no</code>.

<!-- more -->

Back to the matter at hand. First, let's verify if Samba is installed on the CentOS system:

```
rpm -q samba
package samba is not installed
```

Since it's not installed on my machine, I installed it with <code>yum install samba</code>, and then ran the previous command again, to check the version:

```
rpm -q samba
samba-4.4.4-14.el7_3.x86_64
```

Now, let's start Samba and see it running:

```
service smb start
Redirecting to /bin/systemctl start  smb.service
service smb status
Redirecting to /bin/systemctl status  smb.service
● smb.service - Samba SMB Daemon
   Loaded: loaded (/usr/lib/systemd/system/smb.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2017-06-08 10:56:54 EEST; 5s ago
 Main PID: 61874 (smbd)
   Status: "smbd: ready to serve connections..."
   CGroup: /system.slice/smb.service
           ├─61874 /usr/sbin/smbd
           ├─61875 /usr/sbin/smbd
           ├─61876 /usr/sbin/smbd
           └─61879 /usr/sbin/smbd

Jun 08 10:56:52 localhost.localdomain systemd[1]: Starting Samba SMB Daemon...
Jun 08 10:56:54 localhost.localdomain smbd[61874]: [2017/06/08 10:56:54.513901,  0] ../lib/util/become_daemon.c:124(daemon_ready)
Jun 08 10:56:54 localhost.localdomain smbd[61874]:   STATUS=daemon 'smbd' finished starting up and ready to serve connections
Jun 08 10:56:54 localhost.localdomain systemd[1]: Started Samba SMB Daemon.
```

## Samba daemons

The Samba functionality is contained within 3 daemons:

* **smbd** - file sharing, printing services, authentication. Default ports are 139 and 445

* **nmbd** - NetBIOS name service requests and browsing protocols

* **winbindd** - used for Windows domains membership

## Samba configuration

The Samba configuration file is <code>/etc/samba/smb.conf</code>. Here is how a fresh config file looks after installation:

```
# See smb.conf.example for a more detailed config file or
# read the smb.conf manpage.
# Run 'testparm' to verify the config is correct after
# you modified it.

[global]
	workgroup = SAMBA
	security = user

	passdb backend = tdbsam

	printing = cups
	printcap name = cups
	load printers = yes
	cups options = raw

[homes]
	comment = Home Directories
	valid users = %S, %D%w%S
	browseable = No
	read only = No
	inherit acls = Yes

[printers]
	comment = All Printers
	path = /var/tmp
	printable = Yes
	create mask = 0600
	browseable = No

[print$]
	comment = Printer Drivers
	path = /var/lib/samba/drivers
	write list = root
	create mask = 0664
	directory mask = 0775
```

For much more detailed information and examples, see the [smb.conf.example file](/downloads/code/smb.conf.example) 


## Create Samba share

In this example, let's create a share that users can also write to. First, create the directory that you will share: <code>mkdir -p /srv/samba/myshare</code>. I placed a text file with some random stuff inside. Then I gave full access to the path and its subfolders with <code>chmod -R 777 /srv/samba</code>

Next, we need to create a Samba user, but this account is not the same as a user account on the system. We have to make a user account on the system before assigning it to Samba:

```
adduser smbuser -s /sbin/nologin
```

Here I created a user just for Samba, with no login shell. Attempting to login will give the user a message that they are not allowed to login. If you prefer that the user is disconnected with no message, you can specify <code>/bin/false</code> instead.

Then, I gave the user account a description, which you can find inside */etc/passwd*:

```
usermod -c 'Samba access is allowed for this user' smbuser
[root@localhost ~]# cat /etc/passwd | grep smbuser
smbuser:x:1001:1001:Samba access is allowed for this user:/home/smbuser:/sbin/nologin
```

Have to give the user account a password:

```
[root@localhost ~]# passwd smbuser
Changing password for user smbuser.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

Next, we create a Samba user, by using the previously created account:

```
[root@localhost ~]# smbpasswd -a smbuser
New SMB password:
Retype new SMB password:
Added user smbuser.
```

To be safe, check that the Samba user was created:

```
[root@localhost ~]# pdbedit -L
smbuser:1001:Samba access is allowed for this user
```

We have the share location and the user, now we need to edit the smb.conf file with the relevant information:

```
[global]
	# workgroup name or Windows NT domain name
	workgroup = SAMBA
	# default: user and password authentication
	security = user
	# optional comment for Windows
	server string = Samba File Server
	# default backend for user information
	passdb backend = tdbsam

[sharename]
	path = /srv/samba/myshare
	comment = Only authorized users
	# allow these users
	valid users = smbuser
	# same can be achieved with writable = yes
	read only = no  
	# allow subnet range
	allow hosts = 192.168.217.
	# deny access
	invalid users = root  
```

I used comments for easier understanding, but for performance reasons, you might want to keep your file to minimum size, by removing all those comment lines. You can do that by keeping a configuration file with all the additional remarks, while using a smb.conf with only the required configuration. All the comments will be stripped from the config file:

```
[root@localhost samba]# testparm -s smb.conf.old > smb.conf
Load smb config files from smb.conf.old
rlimit_max: increasing rlimit_max (1024) to minimum Windows limit (16384)
Processing section "[sharename]"
Loaded services file OK.
Server role: ROLE_STANDALONE
```

We checked that our config file is valid, so now it's time to test it. Restart Samba for the configurations to take effect with <code>service smb restart</code>. And now let's access the share! From another machine, I used **smbclient** to list the available services on the Samba server:

```
smbclient -L 192.168.217.131 -U smbuser
WARNING: The "syslog" option is deprecated
Enter smbuser's password:
Domain=[SAMBA] OS=[Windows 6.1] Server=[Samba 4.4.4]

	Sharename       Type      Comment
	---------       ----      -------
	sharename       Disk      Only authorized users
	IPC$            IPC       IPC Service (Samba File Server)
Domain=[SAMBA] OS=[Windows 6.1] Server=[Samba 4.4.4]

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
```

There is one last step that you need to accomplish if you have SELinux enabled. You have to label the directory you're sharing with the **samba_share_t** label:

```
chcon -R -t samba_share_t /srv/samba
```

Now the /srv/samba directory and everything it contains is labeled correctly, and SELinux won't interfere. View the security context of the path with:

```
ls -ldZ /srv/samba/
drwxr-xr-x. root root unconfined_u:object_r:samba_share_t:s0 /srv/samba/
```

Changes made with chcon are temporary. To survive a relabel or running *restorerecon*, make the changes permanent with:

```
semanage fcontext -a -t samba_share_t "path(/.*)?"
```

Then apply them with <code>restorecon -R -v /path</code>.

Finally, to connect to a share, use the syntax: <code>smbclient  //host/sharename -U username</code> (in the below examle, the name of my share is sharename, because laziness):

```
smbclient  //192.168.217.131/sharename -U smbuser
WARNING: The "syslog" option is deprecated
Enter smbuser's password:
Domain=[SAMBA] OS=[Windows 6.1] Server=[Samba 4.4.4]
smb: \> ls
      .                                   D        0  Thu Jun  8 09:39:42 2017
      ..                                  D        0  Thu Jun  8 09:26:38 2017
      read.txt                            N       11  Thu Jun  8 09:39:42 2017

    		18307072 blocks of size 1024. 13091600 blocks available
```

List available commands:

```
    smb: \> ?
    ?              allinfo        altname        archive        backup         
    blocksize      cancel         case_sensitive cd             chmod          
    chown          close          del            dir            du             
    echo           exit           get            getfacl        geteas         
    hardlink       help           history        iosize         lcd            
    link           lock           lowercase      ls             l              
    mask           md             mget           mkdir          more           
    mput           newer          notify         open           posix          
    posix_encrypt  posix_open     posix_mkdir    posix_rmdir    posix_unlink   
    posix_whoami   print          prompt         put            pwd            
    q              queue          quit           readlink       rd             
    recurse        reget          rename         reput          rm             
    rmdir          showacls       setea          setmode        scopy          
    stat           symlink        tar            tarmode        timeout        
    translate      unlock         volume         vuid           wdel           
    logon          listconnect    showconnect    tcon           tdis           
    tid            logoff         ..             !              
```

Download file:

```
smb: \> get read.txt
getting file \read.txt of size 11 as read.txt (3.6 KiloBytes/sec) (average 3.6 KiloBytes/sec)
```

Delete file:

```
smb: \> del read.txt
```

Upload file:

```
smb: \> put test.png
putting file test.png as \test.png (0.2 kb/s) (average 0.2 kb/s)
smb: \> ls
  .                                   D        0  Thu Jun  8 13:24:14 2017
  ..                                  D        0  Thu Jun  8 09:26:38 2017
  test.png                            A       35  Thu Jun  8 13:22:09 2017

		18307072 blocks of size 1024. 13091480 blocks available
```

From a Windows system, you can run <code>\\192.168.217.131\sharename</code> to connect to the share, or use the *net use* command.

View shares:

```
C:\Documents and Settings\admin>net use
New connections will be remembered.


Status       Local     Remote                    Network

-------------------------------------------------------------------------------
OK                     \\192.168.217.131\sharename
                                                 Microsoft Windows Network
The command completed successfully.
```

Connect to shares:

```
C:\Documents and Settings\admin>net use S: \\192.168.217.131\sharename
The command completed successfully.


C:\Documents and Settings\admin>s:

S:\>dir
 Volume in drive S is sharename
 Volume Serial Number is DCCC-194F

 Directory of S:\

06/09/2017  02:24 AM    <DIR>          .
06/08/2017  10:26 PM    <DIR>          ..
06/09/2017  02:22 AM                35 test.png
               1 File(s)             35 bytes
               2 Dir(s)  13,405,708,288 bytes free
```

### Other useful options

You can drill down into the smb.conf file and customize it to your liking. Here are a few options:

- read list = user1, user2 - set read only users on a writable share

- write list = user1, user2 - set write access for users on a read only share

- deny hosts  = ip - deny access to the specified IPs

- hide unreadable = yes - don't let users see files they don't have access to

- browseable = no - hide shares from Windows network

Key takeaways:

* server and share security levels are deprecated, so best to avoid them

* specifying a share in the smb.conf file is not enough. Ensure that you have created the path and gave it sufficient permissins

* Samba users need to already exist on the system

* you can have both a well documented config file and a minimal size one for performance, by using <code>testparm -s</code>

* if you use SELinux, don't forget to label your share with <code>samba_share_t</code>

Learn more:

* [Samba section in Paul Cobbaut's Linux Servers course](http://linux-training.be/linuxsrv.pdf)

* [smb.conf manpage](https://www.samba.org/samba/docs/man/manpages-3/smb.conf.5.html) - The configuration file for the Samba suite

* [CentOS Samba guide](https://www.centos.org/docs/5/html/Deployment_Guide-en-US/ch-samba.html)

* [smbpasswd man page](smbpasswd - The Samba encrypted password file) - The Samba encrypted password file

* [pdbedit manpage](https://www.samba.org/samba/docs/man/manpages/pdbedit.8.html) - manage the SAM database (Database of Samba Users)

* [testparm manpage](https://www.samba.org/samba/docs/man/manpages-3/testparm.1.html) - check an smb.conf configuration file for internal correctness

* [smbclient manpage](https://www.samba.org/samba/docs/man/manpages-3/smbclient.1.html) - ftp-like client to access SMB/CIFS resources on servers

``` 
/ Nothing so needs reforming as other \
| people's habits.                    |
|                                     |
\ -- Mark Twain                       /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

