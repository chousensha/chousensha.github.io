---
layout: post
title: "LFCS prep - Essential commands"
date: 2018-02-24 16:46:53 -0500
comments: true
categories: [linux, sysadmin]
keywords: linux, linux commands, lfcs, rhcsa, lfcsa
description: Various commands for the LFCS exam
---

Various commands for the LFCS Essential Commands domain, which at the time of this post can be found at https://training.linuxfoundation.org/images/pdfs/LFCS_Domains_Competencies_V2.16.pdf

<!-- more-->

{% img center /images/sysadmin/cmds.png 'linux cmds' 'cmd essentials' %}

#### cut

> remove sections from each line of files

* select only relevant fields of output

Here I'm looking only at users and their home directories from /etc/passwd:

```
cat /etc/passwd | cut -d: -f1,6 | tail -n 5
apache:/usr/share/httpd
nixer:/home/nixer
lfcs:/home/lfcs
squid:/var/spool/squid
tom:/home/tom
```

#### sed

> stream editor for filtering and transforming text

* substitute all occurrences in a file to something else: <code>sed 's/original/replacement/flag' filename</code>

```
cat text.txt
Peace is a lie, there is only passion.
Through passion, I gain strength.
Through strength, I gain power.
Through power, I gain victory.
Through victory, my chains are broken.
```

Changed the I to a we and redirect to a new file:

```
sed 's/I/we/g' text.txt > altered.txt
cat altered.txt
Peace is a lie, there is only passion.
Through passion, we gain strength.
Through strength, we gain power.
Through power, we gain victory.
Through victory, my chains are broken.
The Force shall free me.
```

#### diff

> compare files line by line

- compare files side by side

```
diff -y text.txt altered.txt
Peace is a lie, there is only passion.				Peace is a lie, there is only passion.
Through passion, I gain strength.			      |	Through passion, we gain strength.
Through strength, I gain power.				      |	Through strength, we gain power.
Through power, I gain victory.				      |	Through power, we gain victory.
Through victory, my chains are broken.				Through victory, my chains are broken.
The Force shall free me.					The Force shall free me.
```

#### man

> an interface to the on-line reference manuals

* search manpages by keyword

```
man -k print | head -n 5
fmtmsg (3)           - print formatted error messages
isprint (3)          - character classification routines
iswprint (3)         - test for printing wide character
netstat (8)          - Print network connections, routing tables, interface statistics, masquerade connections, and multicast memberships
abrt-action-list-dsos (1) - Prints out DSO from mapped memory regions.
```

#### umask

> display or set the file creation mask

It can be used to remove, but not add permissions

```
umask
0022
umask -S
u=rwx,g=rx,o=rx
```

#### info

> read Info documents

#### touch

> Update the access and modification times of each FILE to the current time

- r FILE ->  use this file's times instead of current time


#### whatis

> display manual page descriptions

```
whatis ip
ip (7)               - Linux IPv4 protocol implementation
ip (8)               - show / manipulate routing, devices, pol...
```

#### apropos

> search the manual page names and descriptions

```
apropos network | tail
teamnl (8)           - team network device Netlink interface tool
tracepath (8)        - traces path to a network host discoveri...
tracepath6 (8)       - traces path to a network host discoveri...
traceroute (8)       - print the route packets trace to networ...
traceroute6 (8)      - print the route packets trace to networ...
tshark (1)           - Dump and analyze network traffic
umount.nfs (8)       - unmount a Network File System
usernetctl (8)       - allow a user to manipulate a network in...
wget (1)             - The non-interactive network downloader.
wireshark (1)        - Interactively dump and analyze network ...
```

#### uniq

> report or omit repeated lines

- D # print all duplicate lines

- u # only print unique lines

- it won't detect repeated lines unless they are adjacent. Sort first

#### sort

> sort lines of text files

#### fmt

> simple optimal text formatter

- u # uniform spacing: one space between words, two after sentences

```
[root@tron ~]# cat test.txt
Test.     Next sentence.   This    doesn't   look       right
[root@tron ~]# fmt -u test.txt
Test.  Next sentence.  This doesn't look right
```

#### nl

> number lines of files

```
nl test.txt
     1	First line

     2	Second line

     3	Third line
```

#### updatedb

> creates or updates a database used by locate(1). It's usually  run daily by cron(8) to update the default database.

#### tty

> print the file name of the terminal connected to standard input

```
tty
/dev/pts/1
```

#### type

> indicate how each name would be interpreted  if used as a command name

```
type ls
ls is aliased to `ls --color=auto'
```

#### tree

> list contents of directories in a tree-like format

```
tree
.
├── anaconda-ks.cfg
├── Desktop
├── Documents
│   ├── archive_file
│   ├── arhive.tar.gz
│   ├── Documents
│   │   ├── archive_file
│   │   └── test.txt
│   └── test.txt
├── Downloads
├── initial-setup-ks.cfg
├── Music
├── openscap_data
│   └── eval_remediate_results.xml
├── Pictures
├── Public
├── Templates
├── test.txt
└── Videos
```

#### wc

> print newline, word, and byte counts for each file

- l # lines

```
wc -l /etc/hosts
2 /etc/hosts
```

- w # words

```
wc -w test.txt
6 test.txt
```

#### grep

> print lines matching a pattern

- -E # Interpret PATTERN as an extended regular  expression

- -i # ignore case

- -n # prefix each line of output with its line number


* list items that begin with certain PATTERN

```
yum list installed | grep ^kernel
kernel.x86_64                        3.10.0-514.el7          @anaconda/7.3      
kernel.x86_64                        3.10.0-514.26.2.el7     @rhel-7-server-rpms
kernel-devel.x86_64                  3.10.0-514.el7          @anaconda/7.3      
kernel-devel.x86_64                  3.10.0-514.26.2.el7     @rhel-7-server-rpms
kernel-headers.x86_64                3.10.0-514.26.2.el7     @rhel-7-server-rpms
kernel-tools.x86_64                  3.10.0-514.26.2.el7     @rhel-7-server-rpms
kernel-tools-libs.x86_64             3.10.0-514.26.2.el7     @rhel-7-server-rpms
```

* list items that end with certain PATTERN

```
yum list installed | grep x86_64$
NetworkManager-libreswan-gnome.x86_64
iscsi-initiator-utils-iscsiuio.x86_64
libreport-plugin-reportuploader.x86_64
libreport-rhel-anaconda-bugzilla.x86_64
libvirt-daemon-driver-interface.x86_64
libvirt-daemon-driver-nwfilter.x86_64
openlmi-indicationmanager-libs.x86_64
sane-backends-drivers-scanners.x86_64
subscription-manager-initial-setup-addon.x86_64
```

* grep something that begins and ends with a certain PATTERN, but with variable content in the middle

```
grep -E '^de......tion$' /usr/share/dict/words | head -n 5
deactivation
dealkylation
deallocation
deambulation
deammonation
```

#### find

> search for files in a directory hierarchy

- find / -perm -6000 # find SUID and SGID

- find / -size 10M -type f # find files whose size is 10 megabytes

- type d/c/l/f # directories/character devices/symlinks/files

- user <username> # owned by username

- exec <cmd> '{}' + # execute cmd on results of the find cmd

- print # print full filename to stdout, followed by newline

- name "value" # find values that start with the name

- iname # ignore case

- not -name # not <name>

- size +/- c/k/m/G # size in bytes; + =larger than, - = smaller than; c = bytes, k = kilobytes, m = megabytes, G = gigabytes

- mtime # modification time

- atime # access time

#### which

> shows the full path of (shell) commands. It does this by searching for an executable or script in the directories listed in the environment variable PATH

```
which python
/usr/bin/python
```

#### locate

> find files by name. Reads one or more databases prepared by updatedb(8) and writes file names matching at least one of the PATTERNs to  standard  output,  one  per line.

It is much faster than find. On most systems, *locate* is a link to *slocate*, which includes security features that prevent users from seeing files in directories they shouldn't be able to access

```
locate tcpdump
/usr/sbin/tcpdump
/usr/share/bash-completion/completions/tcpdump
/usr/share/doc/tcpdump-4.5.1
/usr/share/doc/systemtap-client-3.0/examples/network/tcpdumplike.meta
/usr/share/doc/systemtap-client-3.0/examples/network/tcpdumplike.stp
/usr/share/doc/tcpdump-4.5.1/CHANGES
/usr/share/doc/tcpdump-4.5.1/CREDITS
/usr/share/doc/tcpdump-4.5.1/LICENSE
/usr/share/doc/tcpdump-4.5.1/README.md
/usr/share/man/man8/tcpdump.8.gz
/usr/share/mime/application/vnd.tcpdump.pcap.xml
/var/cache/man/cat8/tcpdump.8.gz
```

#### tar

> saves many files together into a single tape or disk archive, and can restore individual files from the archive

- c # create archive

- t # list the contents of an archive

- x # extract files from archive

- v # verbosely list files processed

- z # gzip compression

- j # bzip compression

- A # append files to existing archive

- file # archive file

- p # preserve permissions

- W # attempt to verify the archive after writing it

- --exclude=PATTERN # exclude files

* create archive

```
tar cvzf arhive.tar.gz Documents/
Documents/
Documents/test.txt
Documents/archive_file
[root@tron ~]# ls
```

* list files in archive_file

```
tar tvf arhive.tar.gz
drwxr-xr-x root/root         0 2017-09-08 17:54 Documents/
-rw-r--r-- root/root        36 2017-09-08 17:53 Documents/test.txt
-rw-r--r-- root/root         0 2017-09-08 17:54 Documents/archive_file
```

* decompress gzip archive: <code>tar xzvf arhive.tar.gz</code>

``` 
 _______________________________________
/ Kindness is a language which the deaf \
| can hear and the blind can read.      |
|                                       |
\ -- Mark Twain                         /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
