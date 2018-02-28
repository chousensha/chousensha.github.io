---
layout: post
title: "LFCS prep - User and group management"
date: 2018-02-28 13:26:56 -0500
comments: true
categories: [linux, sysadmin]
keywords: linux, linux commands, linux users, linux groups, lfcs, rhcsa, lfcsa
description: Commands for managing users and groups
---

Various commands for the LFCS User and Group Management domain, which at the time of this post can be found at https://training.linuxfoundation.org/images/pdfs/LFCS_Domains_Competencies_V2.16.pdf

<!-- more -->

{% img center /images/sysadmin/users.png 'user cmds' 'user and group cmds' %}

#### ulimit

> Provides control over the resources available  to the shell and to processes started by it, on systems that allow such control

```
help ulimit
ulimit: ulimit [-SHacdefilmnpqrstuvx] [limit]
    Modify shell resource limits.

    Provides control over the resources available to the shell and processes
    it creates, on systems that allow such control.

    Options:
      -S	use the `soft' resource limit
      -H	use the `hard' resource limit
      -a	all current limits are reported
      -b	the socket buffer size
      -c	the maximum size of core files created
      -d	the maximum size of a process's data segment
      -e	the maximum scheduling priority (`nice')
      -f	the maximum size of files written by the shell and its children
      -i	the maximum number of pending signals
      -l	the maximum size a process may lock into memory
      -m	the maximum resident set size
      -n	the maximum number of open file descriptors
      -p	the pipe buffer size
      -q	the maximum number of bytes in POSIX message queues
      -r	the maximum real-time scheduling priority
      -s	the maximum stack size
      -t	the maximum amount of cpu time in seconds
      -u	the maximum number of user processes
      -v	the size of virtual memory
      -x	the maximum number of file locks

    If LIMIT is given, it is the new value of the specified resource; the
    special LIMIT values `soft', `hard', and `unlimited' stand for the
    current soft limit, the current hard limit, and no limit, respectively.
    Otherwise, the current value of the specified resource is printed.  If
    no option is given, then -f is assumed.

    Values are in 1024-byte increments, except for -t, which is in seconds,
    -p, which is in increments of 512 bytes, and -u, which is an unscaled
    number of processes.

    Exit Status:
    Returns success unless an invalid option is supplied or an error occurs.
```

Config file in **/etc/security/limits.conf**

* list current limits

```
ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7238
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7238
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

#### quotacheck

> scan a filesystem for disk usage, create, check and repair quota files

#### quotaon, quotaoff

> turn filesystem quotas on and off

#### repquota

> summarize quotas for a filesystem

#### setquota

> set disk quotas

#### edquota

> edit user quotas

#### pwscore

> simple configurable tool for checking quality of a password

> password is read from stdin

> It either reports an error if the password fails any of the checks,  or it  prints out the password quality score as an integer value between 0 and 100.

#### getent

> get entries from Name Service Switch libraries

* enumerate networks

```
getent networks
default               0.0.0.0
loopback              127.0.0.0
link-local            169.254.0.0
```

#### set

> list shell attributes

```
set -o
allexport      	off
braceexpand    	on
emacs          	on
errexit        	off
errtrace       	off
functrace      	off
hashall        	on
histexpand     	on
history        	on
ignoreeof      	off
interactive-comments	on
keyword        	off
monitor        	on
noclobber      	off
noexec         	off
noglob         	off
nolog          	off
notify         	off
nounset        	off
onecmd         	off
physical       	off
pipefail       	off
posix          	off
privileged     	off
verbose        	off
vi             	off
xtrace         	off
```

- -o <option name> set option to on

- +o <option name> set option to off

Example: noclobber protects files from being accidentally overwritten with >. To set it at login, add it in .bashrc. You can still overwrite with: **>|**

#### who

> show who is logged on

```
who
root     :0           2017-09-19 12:38 (:0)
root     pts/0        2017-09-19 12:38 (:0)
root     pts/1        2017-09-19 12:40 (:0)
```

- b # time of last system boot

```
who -b
         system boot  2017-09-19 12:36
```

- q # all login names and number of users logged on

```
who -q
root root root
# users=3
```

- r # print current runlevel

```
who -r
         run-level 5  2017-09-20 13:15
```

#### id

> print real and effective user and group IDs

* print all group IDs

```
id -G madhat
1000 10
```

* print group names

```
id -Gn madhat
madhat wheel
```

#### useradd

> create a new user or update default new user information

- -c # Any text string. It is generally a short description of the login, and is currently used as the field for the user's full name.
- -e # expire date. The date on which the user account will be disabled. The date is specified in the format YYYY-MM-DD.

If not specified, useradd will use the default expiry date specified by the EXPIRE variable in /etc/default/useradd, or an empty string (no expiry) by default.

- -g # The group name or number of the user's initial login group. The group name must exist. A group number must refer to an already existing group.
- -G # A list of supplementary groups which the user is also a member of. Each group is separated from the next by a comma, with no intervening whitespace
- -m # Create the user's home directory if it does not exist
- -s # The name of the user's login shell

* create user, specify primary and supplementary groups, create home directory, specify login shell

```
useradd -c 'sample user' -g samplegroup -G users -m -s /bin/sh nixer
grep nixer /etc/passwd
nixer:x:1001:1025:sample user:/home/nixer:/bin/sh
```

* list defaults

```
useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

#### adduser

- symlink to useradd on CentOS, Perl script on Debian / Ubuntu

#### usermod

> modify a user account

- -c # new comment
- -d # new home directory
- -l # new login name
- -s # new shell

#### chsh

> change your login shell

- -s # new shell

* list shells

```
chsh -l
/bin/sh
/bin/bash
/sbin/nologin
/usr/bin/sh
/usr/bin/bash
/usr/sbin/nologin
/bin/tcsh
/bin/csh
```

#### userdel

> delete a user account and related files

#### chpasswd

> update passwords in batch mode

> The chpasswd command reads a list of user name and password pairs from
> standard input and uses this information to update a group of existing
> users. Each line is of the format:

> user_name:password

> This command is intended to be used in a large system environment where
> many accounts are created at a single time.

#### chage

> change user password expiry information

* set password expiration date and list account aging information

```
chage -E 2017-11-20 madhat
[root@tron ~]# chage -l madhat
Last password change					: never
Password expires					: never
Password inactive					: never
Account expires						: Nov 20, 2017
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
```

#### pwconv

> creates shadow from passwd and an optionally existing shadow.

#### pwunconv

> creates passwd from passwd and shadow and then removes shadow.

#### grpconv

> creates gshadow from group and an optionally existing gshadow.

#### grpunconv

> creates group from group and gshadow and then removes gshadow.

#### newgrp

> log in to a new group - change the current group ID during a login session

#### groupadd

> create a new group

- -g # GID value

#### gpasswd

> administer /etc/group and /etc/gshadow

- -a # add user to group
- -d # delete user from group
- -M # set list of group members

#### chgrp

> change group ownership

> chgrp [OPTION]... GROUP FILE

- -R # operate on files and directories recursively

``` 
 _______________________________________
/ You get along very well with everyone \
\ except animals and people.            /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

