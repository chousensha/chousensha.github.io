---
layout: post
title: "LFCS prep - Managing SELinux"
date: 2018-02-23 13:18:55 -0500
comments: true
categories: [linux, sysadmin]
keywords: selinux, linux security, lfcs, rhcsa, lfcsa
description: Securing Linu with SELinux
---

In this post we'll go over using SELinux to manage the security of a Linux system.

NSA Security-Enhanced Linux or SELinux is a mandatory access control architecture controlled through the <code>/etc/selinux/config</code> file. 

<!-- more -->

Let's take a peek at the file on my box:

``` 
cat /etc/selinux/config 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 
```

If SELinux is disabled and then enabled, all the files inside the filesystem will need to be re-labeled.

You can adjust SELinux with many commands and utilities:

### getenforce

> get the current mode of SELinux

```
getenforce
Enforcing
```

### sestatus

> SELinux status tool

```
sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
```

### setenforce

> modify the mode SELinux is running in

> Use Enforcing or 1 to put SELinux in enforcing mode.
> Use Permissive or 0 to put SELinux in permissive mode


### SELinux log files:

``` 
tail /var/log/audit/audit.log | grep AVC
type=USER_AVC msg=audit(1518200447.761:276): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='avc:  denied  { status } for auid=n/a uid=0 gid=0 path="/etc/fstab" cmdline="systemctl show --property=RequiredBy -- -.mount" scontext=system_u:system_r:initrc_t:s0 tcontext=unconfined_u:object_r:etc_t:s0 tclass=service  exe="/usr/lib/systemd/systemd" sauid=0 hostname=? addr=? terminal=?'
```

### ausearch

> a tool to query audit daemon logs

```
ausearch -m avc
----
time->Wed Aug  3 15:23:45 2016
type=SYSCALL msg=audit(1470227025.724:584): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=7efd495a23a8 a2=1c a3=7fff168b531c items=0 ppid=1 pid=8541 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="httpd" exe="/usr/sbin/httpd" subj=system_u:system_r:httpd_t:s0 key=(null)
type=AVC msg=audit(1470227025.724:584): avc:  denied  { name_bind } for  pid=8541 comm="httpd" src=8888 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket
```

### restorecon

> restore file(s) default SELinux security contexts.

### chcon

> change file SELinux security context

### semanage

> SELinux Policy Management tool

```
semanage boolean -l | grep nfs
xen_use_nfs                    (off  ,  off)  Allow xen to use nfs
mpd_use_nfs                    (off  ,  off)  Allow mpd to use nfs
virt_use_nfs                   (off  ,  off)  Allow virt to use nfs
use_nfs_home_dirs              (off  ,  off)  Allow use to nfs home dirs
ksmtuned_use_nfs               (off  ,  off)  Allow ksmtuned to use nfs
nfsd_anon_write                (off  ,  off)  Allow nfsd to anon write
git_system_use_nfs             (off  ,  off)  Allow git to system use nfs
git_cgi_use_nfs                (off  ,  off)  Allow git to cgi use nfs
logrotate_use_nfs              (off  ,  off)  Allow logrotate to use nfs
cobbler_use_nfs                (off  ,  off)  Allow cobbler to use nfs
httpd_use_nfs                  (off  ,  off)  Allow httpd to use nfs
sge_use_nfs                    (off  ,  off)  Allow sge to use nfs
sanlock_use_nfs                (off  ,  off)  Allow sanlock to use nfs
samba_share_nfs                (off  ,  off)  Allow samba to share nfs
ftpd_use_nfs                   (off  ,  off)  Allow ftpd to use nfs
openshift_use_nfs              (off  ,  off)  Allow openshift to use nfs
polipo_use_nfs                 (off  ,  off)  Allow polipo to use nfs
tmpreaper_use_nfs              (off  ,  off)  Allow tmpreaper to use nfs
nfs_export_all_rw              (on   ,   on)  Allow nfs to export all rw
nfs_export_all_ro              (on   ,   on)  Allow nfs to export all ro
```

* list ports

``` 
semanage port -l | grep ftp
ftp_data_port_t                tcp      20
ftp_port_t                     tcp      21, 989, 990
ftp_port_t                     udp      989, 990
tftp_port_t                    udp      69
```

* change label

``` 
semanage fcontext --add -t <type> file
```

You need to call restorecon after changing the context

### getsebool

> get SELinux boolean value(s)

```
getsebool -a | grep ftp
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off
```

### setsebool

> set SELinux boolean value

```
setseebol <boolean> on/off
```

- -P # make the change persistent to reboots


### seinfo

> SELinux policy query tool

Install the *setools* package to get this utility

```
seinfo

Statistics for policy file: /sys/fs/selinux/policy
Policy Version & Type: v.28 (binary, mls)

   Classes:            91    Permissions:       256
   Sensitivities:       1    Categories:       1024
   Types:            4729    Attributes:        251
   Users:               8    Roles:              14
   Booleans:          301    Cond. Expr.:       350
   Allow:          101266    Neverallow:          0
   Auditallow:        157    Dontaudit:        8030
   Type_trans:      17756    Type_change:        74
   Type_member:        35    Role allow:         39
   Role_trans:        416    Range_trans:      5697
   Constraints:       109    Validatetrans:       0
   Initial SIDs:       27    Fs_use:             28
   Genfscon:          105    Portcon:           597
   Netifcon:            0    Nodecon:             0
   Permissives:         6    Polcap:              2
```

### sealert

> sealert is the user interface component (either GUI or command line) to
> the  setroubleshoot  system. setroubleshoot is used to diagnose SELinux
> denials and attempts  to  provide  user  friendly  explanations  for  a
> SELinux  denial (e.g. AVC) and recommendations for how one might adjust
> the system to prevent the denial in the future.

* -l # Lookup alert by id, if id is wildcard * then return all alerts

* -a # Scan a log file, analyze its AVC's

``` 
sealert -a /var/log/audit/audit.log
100% done
found 4 alerts in /var/log/audit/audit.log
--------------------------------------------------------------------------------

SELinux is preventing /usr/bin/pgrep from getattr access on the filesystem /sys.

*****  Plugin catchall (100. confidence) suggests   **************************

If you believe that pgrep should be allowed getattr access on the sys filesystem by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# ausearch -c 'pgrep' --raw | audit2allow -M my-pgrep
# semodule -i my-pgrep.pp


Additional Information:
Source Context                system_u:system_r:ksmtuned_t:s0
Target Context                system_u:object_r:sysfs_t:s0
Target Objects                /sys [ filesystem ]
Source                        pgrep
Source Path                   /usr/bin/pgrep
Port                          <Unknown>
Host                          <Unknown>
Source RPM Packages           procps-ng-3.3.10-10.el7.x86_64
Target RPM Packages           filesystem-3.2-21.el7.x86_64
Policy RPM                    selinux-policy-3.13.1-102.el7_3.16.noarch
Selinux Enabled               True
Policy Type                   targeted
Enforcing Mode                Enforcing
Host Name                     rhel7
Platform                      Linux rhel7 3.10.0-514.26.2.el7.x86_64 #1 SMP Fri
                              Jun 30 05:26:04 UTC 2017 x86_64 x86_64
Alert Count                   9
First Seen                    2018-02-10 02:25:54 JST
Last Seen                     2018-02-10 03:19:45 JST
Local ID                      d5a68144-c84d-44c4-bde0-31380fd5bb60

Raw Audit Messages
type=AVC msg=audit(1518200385.729:205): avc:  denied  { getattr } for  pid=4616 comm="pgrep" name="/" dev="sysfs" ino=1 scontext=system_u:system_r:ksmtuned_t:s0 tcontext=system_u:object_r:sysfs_t:s0 tclass=filesystem


type=SYSCALL msg=audit(1518200385.729:205): arch=x86_64 syscall=statfs success=no exit=EACCES a0=7f9407f4e013 a1=7fffcd705e80 a2=fffffffffff476d8 a3=7fffcd705b90 items=0 ppid=4615 pid=4616 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm=pgrep exe=/usr/bin/pgrep subj=system_u:system_r:ksmtuned_t:s0 key=(null)

Hash: pgrep,ksmtuned_t,sysfs_t,filesystem,getattr
[...]
```

``` 
 __________________________________
/ Someone is speaking well of you. \
|                                  |
\ How unusual!                     /
 ----------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```



