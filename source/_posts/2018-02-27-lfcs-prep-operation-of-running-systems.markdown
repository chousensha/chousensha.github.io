---
layout: post
title: "LFCS prep - Operation of running systems"
date: 2018-02-27 12:54:56 -0500
comments: true
categories: [linux, sysadmin]
keywords: linux, linux commands, linux operation, linux systems, lfcs, rhcsa, lfcsa
description: Various commands for the LFCS exam objectives
---

Various commands for the LFCS Operation of Running Systems domain, which at the time of this post can be found at https://training.linuxfoundation.org/images/pdfs/LFCS_Domains_Competencies_V2.16.pdf

<!-- more -->

{% img center /images/sysadmin/os1.png 'linux cmds' 'linux operation' %}

{% img center /images/sysadmin/os2.png 'linux cmds' 'linux operation' %}

* systemd-analyze

> Analyze system boot-up performance

```
systemd-analyze
Startup finished in 403ms (kernel) + 1.517s (initrd) + 37.584s (userspace) = 39.505s
```

#### crontab

>  maintains crontab files for individual users

> Crontab  is the program used to install, remove or list the tables used to serve the 
> cron(8) daemon.  Each user can have their own crontab, and though  these  are  files 
> in  /var/spool/, they are not intended to be edited directly

* format of crontab entry

```
cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

* run uptime as root every minute, hour, day of month, month, on weekdays

```
crontab -e
* * * * 1-5 uptime >> /root/log.txt
```

```
You have new mail in /var/spool/mail/root
[root@tron ~]# mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 4 messages 4 new
>N  1 (Cron Daemon)         Mon Oct  9 14:10  25/850   "Cron <root@tron> root"
 N  2 (Cron Daemon)         Mon Oct  9 14:11  25/850   "Cron <root@tron> root"
 N  3 (Cron Daemon)         Mon Oct  9 14:12  25/850   "Cron <root@tron> root"
 N  4 (Cron Daemon)         Mon Oct  9 14:13  25/850   "Cron <root@tron> root"
```

```
 cat log.txt
 14:29:02 up  1:03,  3 users,  load average: 0.01, 0.09, 0.12
 14:30:01 up  1:04,  3 users,  load average: 0.39, 0.24, 0.18
```

#### anacron
 
> runs commands periodically
 
> Unlike cron(8), it does not assume that the machine is  running  continuously.   
> Hence, it can be used on machines that are not running 24 hours a day to control 
> regular jobs  as  daily,  weekly, and monthly jobs
 
 
```
 cat /etc/anacrontab
 # /etc/anacrontab: configuration file for anacron
 
 # See anacron(8) and anacrontab(5) for details.
 
 SHELL=/bin/sh
 PATH=/sbin:/bin:/usr/sbin:/usr/bin
 MAILTO=root
 # the maximal random delay added to the base delay of the jobs
 RANDOM_DELAY=45
 # the jobs will be started during the following hours only
 START_HOURS_RANGE=3-22
 
 #period in days   delay in minutes   job-identifier   command
 1	5	cron.daily		nice run-parts /etc/cron.daily
 7	25	cron.weekly		nice run-parts /etc/cron.weekly
 @monthly 45	cron.monthly		nice run-parts /etc/cron.monthly
```
 
* anacron job
 
```
1 1 demo netstat -lnt > /root/conn.txt
```

Check validity of job file with <code>anacron -T</code>

#### at
 
> executes commands at a specified time
 
* schedule job and use Ctrl-D at the prompt when you're done
 
``` 
 at 19:08 oct 09
 at> netstat -lnt > /root/ports.txt
 at> <EOT>
 job 3 at Mon Oct  9 19:08:00 2017
```
 
#### atq
 
 > lists  the  user's  pending  jobs, unless the user is the superuser; in that case, 
> everybody's jobs are listed.
 
```
 [root@tron ~]# atq
 3	Mon Oct  9 19:08:00 2017 a root
```
 
#### ldd
 
> print shared library dependencies
 
> In the usual  case,  ldd  invokes  the  standard  dynamic  linker  (see ld.so(8))  
> with the LD_TRACE_LOADED_OBJECTS environment variable set to 1, which causes the 
> linker to display  the  library  dependencies.   Be aware,  > however,  that  in some
> circumstances, some versions of ldd may attempt to obtain the dependency information 
> by directly executing  the program.  Thus, you should never employ ldd on an untrusted
> executable, since this may result in the execution  of  arbitrary  code.   A  safer 
> alternative when dealing with untrusted executables is:
 
 > $ objdump -p /path/to/program | grep NEEDED
 
```
 ldd /usr/bin/bash
 	linux-vdso.so.1 =>  (0x00007fff62b20000)
 	libtinfo.so.5 => /lib64/libtinfo.so.5 (0x00007ff06afba000)
 	libdl.so.2 => /lib64/libdl.so.2 (0x00007ff06adb6000)
 	libc.so.6 => /lib64/libc.so.6 (0x00007ff06a9f4000)
 	/lib64/ld-linux-x86-64.so.2 (0x00007ff06b1f9000)
```

#### sar
 
 > Collect, report, or save system activity information
 
 * CPU activity
 
```
 sar
 Linux 3.10.0-514.26.2.el7.x86_64 (tron) 	10/05/2017 	_x86_64_	(1 CPU)
 
 03:57:29 PM       LINUX RESTART
 
 04:00:01 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
 04:10:01 PM     all      2.66      0.00      1.02      0.15      0.00     96.17
 04:20:01 PM     all      2.64      0.00      0.83      0.01      0.00     96.52
 04:30:01 PM     all      0.65      0.03      0.94      0.09      0.00     98.28
 04:40:01 PM     all      0.35      0.00      0.20      0.00      0.00     99.45
 04:50:01 PM     all      0.25      0.00      0.14      0.00      0.00     99.60
 05:00:01 PM     all      0.24      0.00      0.17      0.00      0.00     99.58
 05:10:01 PM     all      0.24      0.00      0.17      0.00      0.00     99.59
 05:20:01 PM     all      0.35      0.00      0.20      0.00      0.00     99.45
 05:30:01 PM     all      0.40      0.00      0.24      0.00      0.00     99.36
 05:40:01 PM     all      0.51      0.00      0.21      0.00      0.00     99.27
 05:50:01 PM     all      5.88      0.00      1.51      0.02      0.00     92.59
 06:00:02 PM     all      0.48      0.00      0.25      0.00      0.00     99.27
 06:10:01 PM     all      0.16      0.00      0.19      0.00      0.00     99.65
 06:20:01 PM     all      2.16      0.00      0.59      0.01      0.00     97.25
 06:30:01 PM     all      2.94      0.00      0.69      0.00      0.00     96.37
 06:40:01 PM     all      0.10      0.00      0.18      0.00      0.00     99.72
 06:50:01 PM     all      0.11      0.00      0.13      0.00      0.00     99.76
 07:00:02 PM     all      0.17      0.00      0.21      0.00      0.00     99.62
 07:10:01 PM     all      0.10      0.00      0.15      0.00      0.00     99.76
 07:20:01 PM     all      0.10      0.00      0.14      0.00      0.00     99.76
 07:30:01 PM     all      0.14      0.00      0.16      0.00      0.00     99.70
 07:40:02 PM     all      0.10      0.00      0.14      0.00      0.00     99.76
 
 07:40:02 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
 07:50:01 PM     all      0.09      0.00      0.15      0.00      0.00     99.75
 08:00:01 PM     all      0.19      0.00      0.26      0.00      0.00     99.55
 08:10:01 PM     all      0.44      0.00      0.22      0.00      0.00     99.34
 Average:        all      0.84      0.00      0.36      0.01      0.00     98.78
```
 
 * memory statistics
 
```
 sar -r
 Linux 3.10.0-514.26.2.el7.x86_64 (tron) 	10/05/2017 	_x86_64_	(1 CPU)
 
 03:57:29 PM       LINUX RESTART
 
 04:00:01 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
 04:10:01 PM    889804    994304     52.77      1004    382376   2556680     64.22    522128    294044        12
 04:20:01 PM    890844    993264     52.72      1004    382456   2556872     64.22    521256    294004         4
 04:30:01 PM    802736   1081372     57.39      1004    387280   2557724     64.24    524088    296556         0
 04:40:01 PM    802744   1081364     57.39      1004    387320   2557724     64.24    524120    296564        12
 04:50:01 PM    802712   1081396     57.40      1004    387296   2557724     64.24    524096    296556         0
 05:00:01 PM    802640   1081468     57.40      1004    387304   2582312     64.86    524252    296544         8
 05:10:01 PM    803032   1081076     57.38      1004    387288   2581776     64.85    524048    296516         0
 05:20:01 PM    798948   1085160     57.60      1004    387288   2581776     64.85    528108    296516         0
 05:30:01 PM    798808   1085300     57.60      1004    387312   2581776     64.85    528156    296524        20
 05:40:01 PM    798552   1085556     57.62      1004    387344   2581720     64.85    528424    296520         0
 05:50:01 PM    773308   1110800     58.96      1004    388048   2605656     65.45    553040    297056         0
 06:00:02 PM    773116   1110992     58.97      1004    388056   2605656     65.45    553196    297056         0
 06:10:01 PM    770812   1113296     59.09      1004    388564   2605664     65.45    554944    297556         0
 06:20:01 PM    733504   1150604     61.07      1004    388708   2642264     66.37    592036    297640         0
 06:30:01 PM    732720   1151388     61.11      1004    388852   2642544     66.37    592928    297552         0
 06:40:01 PM    732732   1151376     61.11      1004    388884   2642608     66.38    592956    297556         8
 06:50:01 PM    732640   1151468     61.11      1004    388864   2642608     66.38    592948    297556         0
 07:00:02 PM    732616   1151492     61.12      1004    388872   2642608     66.38    593060    297496         0
 07:10:01 PM    732512   1151596     61.12      1004    388880   2642608     66.38    593120    297496         0
 07:20:01 PM    732536   1151572     61.12      1004    388888   2642608     66.38    593324    297320         0
 07:30:01 PM    732452   1151656     61.12      1004    388896   2642608     66.38    593348    297320         0
 07:40:02 PM    732424   1151684     61.13      1004    388924   2642608     66.38    593364    297324         4
 
 07:40:02 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
 07:50:01 PM    732388   1151720     61.13      1004    388908   2642608     66.38    593376    297320         0
 08:00:01 PM    732444   1151664     61.13      1004    388956   2642608     66.38    594632    296136         8
 08:10:01 PM    752844   1131264     60.04      1004    388988   2621972     65.86    574028    296148         0
 Average:       772795   1111313     58.98      1004    387782   2608132     65.51    560359    296755         3
```
 
#### iostat
 
 > report CPU statistics and input/output statistics for devices and partitions.
 
```
 iostat
 Linux 3.10.0-514.26.2.el7.x86_64 (tron) 	10/05/2017 	_x86_64_	(1 CPU)
 
 avg-cpu:  %user   %nice %system %iowait  %steal   %idle
            1.41    0.01    0.76    0.20    0.00   97.61
 
 Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
 sda               2.09        54.42         6.10     452565      50723
 dm-0              1.85        50.73         5.85     421879      48655
 dm-1              0.02         0.13         0.00       1068          0
```
 
#### pidstat
 
 > Report statistics for Linux tasks
 
```
 pidstat -p 3381
 Linux 3.10.0-514.26.2.el7.x86_64 (tron) 	10/05/2017 	_x86_64_	(1 CPU)
 
 06:20:21 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
```
 
#### mpstat
 
 > Report processors related statistics
 
```
 mpstat
 Linux 3.10.0-514.26.2.el7.x86_64 (tron) 	10/05/2017 	_x86_64_	(1 CPU)
 
 06:23:38 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
 06:23:38 PM  all    1.59    0.01    0.76    0.20    0.00    0.02    0.00    0.00    0.00   97.42
```
 
#### vmstat
 
 > Report virtual memory statistics
 
```
 vmstat
 procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
  r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
  4  0      0 776148   1004 503192    0    0    70     8   79  169  2  1 97  0  0
```
 
#### tload
 
 > graphic representation of system load average
 
```
 0.01, 0.07, 0.17
```

#### watch
 
 > execute a program periodically, showing output fullscreen
 
#### lscpu
 
 > display information about the CPU architecture
 
```
 lscpu
 Architecture:          x86_64
 CPU op-mode(s):        32-bit, 64-bit
 Byte Order:            Little Endian
 CPU(s):                1
 On-line CPU(s) list:   0
 Thread(s) per core:    1
 Core(s) per socket:    1
 Socket(s):             1
 NUMA node(s):          1
 Vendor ID:             GenuineIntel
 CPU family:            6
 Model:                 69
 Model name:            Intel(R) Core(TM) i5-4300U CPU @ 1.90GHz
 Stepping:              1
 CPU MHz:               2493.585
 BogoMIPS:              4988.45
 Hypervisor vendor:     VMware
 Virtualization type:   full
 L1d cache:             32K
 L1i cache:             32K
 L2 cache:              256K
 L3 cache:              3072K
 NUMA node0 CPU(s):     0
```
 
 
#### uptime
 
 > Tell how long the system has been running
 
```
 uptime
  16:11:48 up 14 min,  3 users,  load average: 0.06, 0.20, 0.28
```
 
 * show since when is the system up
 
```
 uptime -s
 2017-10-05 15:57:10
```
 
#### pmap
 
 > report memory map of a process
 
 - -p # Show full path to files in the mapping column

```
 pmap -p $$
 3425:   bash
 0000000000400000    884K r-x-- /usr/bin/bash
 00000000006dc000      4K r---- /usr/bin/bash
 00000000006dd000     36K rw--- /usr/bin/bash
 00000000006e6000     24K rw---   [ anon ]
 0000000000cbc000   1572K rw---   [ anon ]
 00007f1b843d1000     48K r-x-- /usr/lib64/libnss_files-2.17.so
 00007f1b843dd000   2044K ----- /usr/lib64/libnss_files-2.17.so
 00007f1b845dc000      4K r---- /usr/lib64/libnss_files-2.17.so
 00007f1b845dd000      4K rw--- /usr/lib64/libnss_files-2.17.so
 00007f1b845de000     24K rw---   [ anon ]
 00007f1b845e4000 103588K r---- /usr/lib/locale/locale-archive
 00007f1b8ab0d000   1756K r-x-- /usr/lib64/libc-2.17.so
 00007f1b8acc4000   2044K ----- /usr/lib64/libc-2.17.so
 00007f1b8aec3000     16K r---- /usr/lib64/libc-2.17.so
 00007f1b8aec7000      8K rw--- /usr/lib64/libc-2.17.so
 00007f1b8aec9000     20K rw---   [ anon ]
 00007f1b8aece000      8K r-x-- /usr/lib64/libdl-2.17.so
 00007f1b8aed0000   2048K ----- /usr/lib64/libdl-2.17.so
 00007f1b8b0d0000      4K r---- /usr/lib64/libdl-2.17.so
 00007f1b8b0d1000      4K rw--- /usr/lib64/libdl-2.17.so
 00007f1b8b0d2000    148K r-x-- /usr/lib64/libtinfo.so.5.9
 00007f1b8b0f7000   2048K ----- /usr/lib64/libtinfo.so.5.9
 00007f1b8b2f7000     16K r---- /usr/lib64/libtinfo.so.5.9
 00007f1b8b2fb000      4K rw--- /usr/lib64/libtinfo.so.5.9
 00007f1b8b2fc000    128K r-x-- /usr/lib64/ld-2.17.so
 00007f1b8b504000     12K rw---   [ anon ]
 00007f1b8b512000      8K rw---   [ anon ]
 00007f1b8b514000     28K r--s- /usr/lib64/gconv/gconv-modules.cache
 00007f1b8b51b000      4K rw---   [ anon ]
 00007f1b8b51c000      4K r---- /usr/lib64/ld-2.17.so
 00007f1b8b51d000      4K rw--- /usr/lib64/ld-2.17.so
 00007f1b8b51e000      4K rw---   [ anon ]
 00007ffeb5ec8000    132K rw---   [ stack ]
 00007ffeb5efc000      8K r-x--   [ anon ]
 ffffffffff600000      4K r-x--   [ anon ]
  total           116692K
```
 
#### pwdx
 
 > report current working directory of a process
 
```
 pwdx $$
 3425: /root
```
 
#### free
 
 > Display amount of free and used memory in the system
 
```
 free -h
               total        used        free      shared  buff/cache   available
 Mem:           1.8G        509M        883M         10M        446M        1.1G
 Swap:          2.0G          0B        2.0G
```
 
#### nice
 
 > run a program with modified scheduling priority
 
 Niceness  values range  from  -20 (most favorable to the process) to 19 (least favorable to the process).
 
 - -n # add integer N to the niceness (default 10)
 
#### renice
 
 >  alter priority of running processes
 
 - -n # priority
 
 
#### sleep
 
 > delay for a specified amount of time
 
#### bg
 
> Resume each suspended job jobspec  in  the  background,  as  if  it  had been started
> with &
 
#### fg
 
 > Resume jobspec in the foreground, and make it the current job
 
#### jobs
 
 > list active jobs
 
```
 sleep 15 &
 [1] 9767
 [root@tron ~]# jobs
 [1]+  Running                 sleep 15 &
```
 
#### kill
 
> terminate a process
 
> Default is the TERM signal, which is similar to asking nicely to terminate. The more 
> forcefull option is KILL(9)
 
 * list signals
 
```
 kill -l
  1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
  6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
 11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
 16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
 21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
 26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
 31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
 38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
 43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
 48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
 53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
 58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
 63) SIGRTMAX-1	64) SIGRTMAX
```

#### pgrep,  pkill   
 
 > look  up  or signal processes based on name and other attributes
 
#### ps
 
 > report a snapshot of the current processes
 
 - -e # Select all processes.  Identical to -A
 
```
 ps -e
   PID TTY          TIME CMD
     1 ?        00:00:02 systemd
     2 ?        00:00:00 kthreadd
     3 ?        00:00:00 ksoftirqd/0
     5 ?        00:00:00 kworker/0:0H
     7 ?        00:00:00 migration/0
     8 ?        00:00:00 rcu_bh
     9 ?        00:00:00 rcu_sched
    10 ?        00:00:00 watchdog/0
    12 ?        00:00:00 kdevtmpfs
[...]
  3591 pts/0    00:00:00 man
  3598 pts/0    00:00:00 less
  3608 ?        00:00:00 sleep
  3614 pts/1    00:00:00 ps
```

 - -r #  Restrict the selection to only running processes
 
```
 ps -r
   PID TTY      STAT   TIME COMMAND
  1354 tty1     Rs+    0:08 /usr/bin/Xorg :0 -background none -noreset -audit 4 -
  3805 pts/1    R+     0:00 ps -r
```
 
 - -u # Select by effective user ID (EUID) or name
 
 * process tree
 
```
 ps --forest
   PID TTY          TIME CMD
  3508 pts/1    00:00:00 bash
  3880 pts/1    00:00:00  \_ ps
```
 
 * all processes + EUID
 
 
```
 ps -aux
 USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
 root         1  0.1  0.3 128088  6756 ?        Ss   17:03   0:02 /usr/lib/system
 root         2  0.0  0.0      0     0 ?        S    17:03   0:00 [kthreadd]
 root         3  0.0  0.0      0     0 ?        S    17:03   0:00 [ksoftirqd/0]
 root         7  0.0  0.0      0     0 ?        S    17:03   0:00 [migration/0]
 root         8  0.0  0.0      0     0 ?        S    17:03   0:00 [rcu_bh]
 root         9  0.0  0.0      0     0 ?        R    17:03   0:00 [rcu_sched]
 root        10  0.0  0.0      0     0 ?        S    17:03   0:00 [watchdog/0]
 root        12  0.0  0.0      0     0 ?        S    17:03   0:00 [kdevtmpfs]
 root        13  0.0  0.0      0     0 ?        S<   17:03   0:00 [netns]
 root        14  0.0  0.0      0     0 ?        S    17:03   0:00 [khungtaskd]
 root        15  0.0  0.0      0     0 ?        S<   17:03   0:00 [writeback]
 root        16  0.0  0.0      0     0 ?        S<   17:03   0:00 [kintegrityd]
 root        17  0.0  0.0      0     0 ?        S<   17:03   0:00 [bioset]
 root        18  0.0  0.0      0     0 ?        S<   17:03   0:00 [kblockd]
 root        19  0.0  0.0      0     0 ?        S<   17:03   0:00 [md]
 root        25  0.0  0.0      0     0 ?        S    17:04   0:00 [kswapd0]
 root        26  0.0  0.0      0     0 ?        SN   17:04   0:00 [ksmd]
 root        27  0.0  0.0      0     0 ?        SN   17:04   0:00 [khugepaged]
 root        28  0.0  0.0      0     0 ?        S    17:04   0:00 [fsnotify_mark]
 root        29  0.0  0.0      0     0 ?        S<   17:04   0:00 [crypto]
 [...]
 root       587  0.0  0.3  47700  5856 ?        Ss   17:04   0:00 /usr/lib/system
 root       625  0.0  0.0      0     0 ?        S<   17:04   0:00 [nfit]
 root       666  0.0  0.0      0     0 ?        S<   17:04   0:00 [xfs-buf/sda1]
 dbus       727  0.0  0.1  36328  3332 ?        Ssl  17:04   0:01 /bin/dbus-daemo
 root       734  0.0  0.0 201208  1208 ?        Ssl  17:04   0:00 /usr/sbin/gsspr
 root      2862  0.0  0.1 380356  3468 ?        Sl   17:06   0:00 /usr/libexec/gv
 root      3508  0.0  0.1 116688  3392 pts/1    Ss   17:07   0:00 bash
 root      3591  0.0  0.1 119548  2292 pts/0    S+   17:12   0:00 man ps
 root      3598  0.0  0.0 110248   960 pts/0    S+   17:12   0:00 less -s
 root      3718  0.0  0.0      0     0 ?        S<   17:22   0:00 [kworker/0:2H]
 root      3743  0.0  0.0      0     0 ?        S    17:24   0:00 [kworker/0:0]
 root      3819  0.0  0.0      0     0 ?        S<   17:27   0:00 [kworker/0:1H]
 root      3892  0.1  0.0      0     0 ?        S    17:29   0:00 [kworker/0:2]
 root      3955  0.0  0.0      0     0 ?        S<   17:33   0:00 [kworker/0:0H]
 root      3967  0.0  0.0 107892   604 ?        S    17:34   0:00 sleep 60
 root      3968  0.1  0.0      0     0 ?        R    17:34   0:00 [kworker/0:1]
 root      3979  0.0  0.0 153136  1876 pts/1    R+   17:35   0:00 ps -aux
```
 
 * long format, don't show flags
 
```
 ps -ly
 S   UID   PID  PPID  C PRI  NI   RSS    SZ WCHAN  TTY          TIME CMD
 S     0  3508  3438  0  80   0  3392 29172 wait   pts/1    00:00:00 bash
 R     0  4002  3508  0  80   0  1452 37232 -      pts/1    00:00:00 ps
```
 
 * get running processes
 
```
 echo $$
 3744
 ps -p $$ -F
 UID        PID  PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
 root      3697  3692  0 29172  3372   0 13:09 pts/0    00:00:00 bash
```
 
#### pstree
 
 > display a tree of processes
 
 - -a # Show  command  line arguments
 
```
 pstree -a
 systemd --switched-root --system --deserialize 21
   ├─ModemManager
   │   └─2*[{ModemManager}]
   ├─NetworkManager --no-daemon
   │   ├─dhclient -d -q -sf /usr/libexec/nm-dhcp-helper -pf /var/run/dhclient-enp0s3.pid -lf ...
   │   └─2*[{NetworkManager}]
   ├─abrt-watch-log -F BUG: WARNING: at WARNING: CPU: INFO: possible recursive locking detected ernel BUG at list_del corruption list_add corruptiondo_IRQ: stack overfl
   ├─abrt-watch-log -F Backtrace /var/log/Xorg.0.log -- /usr/bin/abrt-dump-xorg -xD
   ├─abrtd -d -s
   ├─accounts-daemon
   │   └─2*[{accounts-daemon}]
   ├─alsactl -s -n 19 -c -E ALSA_CONFIG_PATH=/etc/alsa/alsactl.conf --initfile=/lib/alsa/init/00main rdaemon
   ├─at-spi-bus-laun
   │   ├─dbus-daemon --config-file=/etc/at-spi2/accessibility.conf --nofork --print-address 3
   │   │   └─{dbus-daemon}
   │   └─3*[{at-spi-bus-laun}]
   ├─at-spi2-registr --use-gnome-session
   │   └─2*[{at-spi2-registr}]
   ├─atd -f
   ├─auditd -n
   │   ├─audispd
   │   │   ├─sedispatch
   │   │   └─{audispd}
   │   └─{auditd}
   ├─avahi-daemon
   │   └─avahi-daemon
   ├─caribou
   │   └─2*[{caribou}]
   ├─colord
   │   └─2*[{colord}]
   ├─crond -n
   ├─cupsd -f
   ├─dbus-daemon --fork --print-pid 4 --print-address 6 --session
   │   └─{dbus-daemon}
   ├─dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
   │   └─{dbus-daemon}
   ├─dbus-launch --sh-syntax --exit-with-session
   ├─dconf-service
   │   └─2*[{dconf-service}]
   ├─dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
   │   └─dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
   ├─evolution-calen
   │   └─5*[{evolution-calen}]
   ├─evolution-sourc
   │   └─2*[{evolution-sourc}]
   ├─gconfd-2
   ├─gdm
   │   ├─Xorg :0 -background none -noreset -audit 4 -verbose -auth /run/gdm/auth-for-gdm-kJZtDT/database -seat seat0 -nolisten tcp vt1
   │   ├─gdm-session-wor
   │   │   ├─gnome-session --session gnome-classic
   │   │   │   ├─abrt-applet
   │   │   │   │   └─2*[{abrt-applet}]
   │   │   │   ├─gnome-settings-
   │   │   │   │   └─5*[{gnome-settings-}]
   │   │   │   ├─gnome-shell
   │   │   │   │   ├─ibus-daemon --xim --panel disable
   │   │   │   │   │   ├─ibus-dconf
   │   │   │   │   │   │   └─3*[{ibus-dconf}]
   │   │   │   │   │   ├─ibus-engine-sim
   │   │   │   │   │   │   └─2*[{ibus-engine-sim}]
   │   │   │   │   │   └─2*[{ibus-daemon}]
   │   │   │   │   └─6*[{gnome-shell}]
   │   │   │   ├─gnome-software --gapplication-service
   │   │   │   │   └─3*[{gnome-software}]
   │   │   │   ├─nautilus --no-default-window --force-desktop
   │   │   │   │   └─3*[{nautilus}]
   │   │   │   ├─rhsm-icon
   │   │   │   │   └─2*[{rhsm-icon}]
   │   │   │   ├─seapplet
   │   │   │   ├─ssh-agent /bin/sh -c exec -l /bin/bash -c "env GNOME_SHELL_SESSION_MODE=classic gnome-session --session gnome-classic"
   │   │   │   ├─tracker-extract
   │   │   │   │   └─13*[{tracker-extract}]
   │   │   │   ├─tracker-miner-a
   │   │   │   │   └─2*[{tracker-miner-a}]
   │   │   │   ├─tracker-miner-f
   │   │   │   │   └─3*[{tracker-miner-f}]
   │   │   │   ├─tracker-miner-u
   │   │   │   │   └─2*[{tracker-miner-u}]
   │   │   │   └─3*[{gnome-session}]
   │   │   └─2*[{gdm-session-wor}]
   │   └─3*[{gdm}]
   ├─gnome-keyring-d --daemonize --login
   │   └─4*[{gnome-keyring-d}]
   ├─gnome-shell-cal
   │   └─5*[{gnome-shell-cal}]
   ├─gnome-terminal-
   │   ├─bash
   │   │   └─man pstree
   │   │       └─less -s
   │   ├─bash
   │   │   └─pstree -a
   │   ├─gnome-pty-helpe
   │   └─3*[{gnome-terminal-}]
   ├─goa-daemon
   │   └─3*[{goa-daemon}]
   ├─goa-identity-se
   │   └─3*[{goa-identity-se}]
   ├─gsd-printer
   │   └─2*[{gsd-printer}]
   ├─gssproxy -D
   │   └─5*[{gssproxy}]
   ├─gvfs-afc-volume
   │   └─3*[{gvfs-afc-volume}]
   ├─gvfs-goa-volume
   │   └─2*[{gvfs-goa-volume}]
   ├─gvfs-gphoto2-vo
   │   └─2*[{gvfs-gphoto2-vo}]
   ├─gvfs-mtp-volume
   │   └─2*[{gvfs-mtp-volume}]
   ├─gvfs-udisks2-vo
   │   └─2*[{gvfs-udisks2-vo}]
   ├─gvfsd
   │   └─2*[{gvfsd}]
   ├─gvfsd-fuse /run/user/0/gvfs -f -o big_writes
   │   └─5*[{gvfsd-fuse}]
   ├─gvfsd-metadata
   │   └─2*[{gvfsd-metadata}]
   ├─gvfsd-trash --spawner :1.3 /org/gtk/gvfs/exec_spaw/0
   │   └─2*[{gvfsd-trash}]
   ├─ibus-x11 --kill-daemon
   │   └─2*[{ibus-x11}]
   ├─ksmtuned /usr/sbin/ksmtuned
   │   └─sleep 60
   ├─libvirtd
   │   └─15*[{libvirtd}]
   ├─lsmd -d
   ├─lvmetad -f
   ├─master -w
   │   ├─pickup -l -t unix -u
   │   └─qmgr -l -t unix -u
   ├─mcelog --ignorenodev --daemon --syslog
   ├─mission-control
   │   └─3*[{mission-control}]
   ├─packagekitd
   │   └─2*[{packagekitd}]
   ├─polkitd --no-debug
   │   └─5*[{polkitd}]
   ├─pulseaudio --start
   │   └─{pulseaudio}
   ├─rhnsd
   ├─rhsmcertd
   ├─rngd -f
   ├─rsyslogd -n
   │   └─2*[{rsyslogd}]
   ├─rtkit-daemon
   │   └─2*[{rtkit-daemon}]
   ├─smartd -n -q never
   ├─sshd -D
   ├─systemd-journal
   ├─systemd-logind
   ├─systemd-udevd
   ├─tracker-store
   │   └─7*[{tracker-store}]
   ├─tuned -Es /usr/sbin/tuned -l -P
   │   └─4*[{tuned}]
   ├─udisksd --no-debug
   │   └─4*[{udisksd}]
   ├─upowerd
   │   └─2*[{upowerd}]
   ├─vmtoolsd
   │   └─{vmtoolsd}
   ├─vmtoolsd -n vmusr
   └─wpa_supplicant -u -f /var/log/wpa_supplicant.log -c /etc/wpa_supplicant/wpa_supplicant.conf -P /var/run/wpa_supplicant.pid
```

#### runlevel
 
> Print previous and current SysV runlevel
 
> If a runlevel cannot be determined, N is printed instead
 
```
 runlevel
 N 5
```

#### init / telinit
 
 - legacy
 
```
 init [OPTIONS...] {COMMAND}
 
 Send control commands to the init daemon.
 
      --help      Show this help
      --no-wall   Don't send wall message before halt/power-off/reboot
 
 Commands:
   0              Power-off the machine
   6              Reboot the machine
   2, 3, 4, 5     Start runlevelX.target unit
   1, s, S        Enter rescue mode
   q, Q           Reload init daemon configuration
   u, U           Reexecute init daemon
```
 
#### mkfifo
 
 > make FIFOs (named pipes)
 
```
 mkfifo /tmp/pipe
 [root@tron ~]# ls -l /tmp/pipe
 prw-r--r--. 1 root root 0 Sep 27 13:32 /tmp/pipe
```
 
#### script
 
 > make typescript of terminal session
 
```
 script
 Script started, file is typescript
 [root@tron ~]# ls
 anaconda-ks.cfg  Downloads             Music          Templates
 Desktop          hashes.txt            openscap_data  test.txt
 dns.conf         initial-setup-ks.cfg  Pictures       typescript
 Documents        man.txt               Public         Videos
 [root@tron ~]# rm -f dns.conf hashes.txt
 [root@tron ~]# exit
 exit
 Script done, file is typescript
 [root@tron ~]# cat typescript
 Script started on Wed 27 Sep 2017 01:34:50 PM EEST
 [root@tron ~]# ls
 anaconda-ks.cfg  Downloads             Music          Templates
 Desktop          hashes.txt            openscap_data  test.txt
 dns.conf         initial-setup-ks.cfg  Pictures       typescript
 Documents        man.txt               Public         Videos
 [root@tron ~]# rm -f dns.conf hashes.txt
 [root@tron ~]# exit
 exit
 
 Script done on Wed 27 Sep 2017 01:35:18 PM EEST
```

#### stat
 
 > display file or file system status
 
```
 stat hashes.txt
   File: ‘hashes.txt’
   Size: 43        	Blocks: 8          IO Block: 4096   regular file
 Device: fd00h/64768d	Inode: 33859054    Links: 1
 Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
 Context: unconfined_u:object_r:admin_home_t:s0
 Access: 2017-09-26 11:27:18.298105601 +0300
 Modify: 2017-09-22 14:14:29.035069557 +0300
 Change: 2017-09-22 14:14:29.035069557 +0300
  Birth: -
```

#### lastlog
 
 > reports the most recent login of all users or of a given user
 
```
 lastlog
 Username         Port     From             Latest
 root             :0                        Sat Oct  7 14:45:28 +0300 2017
 bin                                        **Never logged in**
 daemon                                     **Never logged in**
 adm                                        **Never logged in**
 lp                                         **Never logged in**
 sync                                       **Never logged in**
 shutdown                                   **Never logged in**
 halt                                       **Never logged in**
 mail                                       **Never logged in**
 operator                                   **Never logged in**
 games                                      **Never logged in**
 ftp                                        **Never logged in**
 nobody                                     **Never logged in**
 systemd-bus-proxy                           **Never logged in**
 systemd-network                            **Never logged in**
 dbus                                       **Never logged in**
 polkitd                                    **Never logged in**
 abrt                                       **Never logged in**
 unbound                                    **Never logged in**
 tss                                        **Never logged in**
 colord                                     **Never logged in**
 usbmuxd                                    **Never logged in**
 geoclue                                    **Never logged in**
 saslauth                                   **Never logged in**
 libstoragemgmt                             **Never logged in**
 rpc                                        **Never logged in**
 rtkit                                      **Never logged in**
 chrony                                     **Never logged in**
 radvd                                      **Never logged in**
 qemu                                       **Never logged in**
 ntp                                        **Never logged in**
 rpcuser                                    **Never logged in**
 nfsnobody                                  **Never logged in**
 avahi-autoipd                              **Never logged in**
 setroubleshoot                             **Never logged in**
 sssd                                       **Never logged in**
 pulse                                      **Never logged in**
 gdm              :0                        Sat Oct  7 14:42:55 +0300 2017
 gnome-initial-setup :0                        Tue Apr 26 15:36:08 +0300 2016
 sshd                                       **Never logged in**
 avahi                                      **Never logged in**
 postfix                                    **Never logged in**
 tcpdump                                    **Never logged in**
 nixhat           :0                        Sat Aug 19 20:00:48 +0300 2017
 apache                                     **Never logged in**
 smbuser                                    **Never logged in**
 squid                                      **Never logged in**
```
 
 * see last time a user logged in
 
```
 lastlog -u root
 Username         Port     From             Latest
 root             :0                        Sat Oct  7 14:45:28 +0300 2017
```

#### last
 
> show listing of last logged in users
 
> Last searches back through the file /var/log/wtmp (or the  file  designated  by  the 
> f flag) and displays a list of all users logged in (and out) since that file was 
> created.
 
```
 last -n 10
 root     pts/1        :0               Sat Oct  7 14:53   still logged in   
 root     pts/0        :0               Sat Oct  7 14:45   still logged in   
 root     :0           :0               Sat Oct  7 14:45   still logged in   
 (unknown :0           :0               Sat Oct  7 14:42 - 14:45  (00:02)    
 reboot   system boot  3.10.0-514.21.1. Sat Oct  7 14:41 - 14:54  (00:13)    
 root     pts/1        :0               Fri Oct  6 21:48 - 23:19  (01:30)    
 root     pts/0        :0               Fri Oct  6 21:47 - 23:19  (01:32)    
 root     :0           :0               Fri Oct  6 21:43 - 23:19  (01:36)    
 (unknown :0           :0               Fri Oct  6 21:37 - 21:43  (00:05)    
 reboot   system boot  3.10.0-514.21.1. Fri Oct  6 21:36 - 23:19  (01:43)    
 
 wtmp begins Tue Apr 26 15:32:43 2016
```
 
#### lastb
 
 > shows a log of the file /var/log/btmp, which contains all the bad login attempts
 
```
 lastb
 root     :0           :0               Sat Oct  7 14:57 - 14:57  (00:00)    
 root     :0           :0               Sat Oct  7 14:57 - 14:57  (00:00)    
 nixhat   :0           :0               Sat Oct  7 14:57 - 14:57  (00:00)    
 
 btmp begins Sat Oct  7 14:57:26 2017
```
 
#### logrotate
 
 > rotates, compresses, and mails system logs
 
 Config file: **/etc/logrotate.conf**
 
#### journalctl
 
 > Query the systemd journal
 
```
 journalctl -u httpd.service
 -- Logs begin at Sat 2017-10-07 14:41:21 EEST, end at Sat 2017-10-07 15:34:36 EE
 Oct 07 14:42:14 localhost.localdomain systemd[1]: Starting The Apache HTTP Serve
 Oct 07 14:42:23 localhost.localdomain httpd[1099]: AH00558: httpd: Could not rel
 Oct 07 14:42:24 localhost.localdomain systemd[1]: Started The Apache HTTP Server
 Oct 07 15:11:02 localhost.localdomain httpd[6068]: AH00558: httpd: Could not rel
 Oct 07 15:11:02 localhost.localdomain systemd[1]: Reloaded The Apache HTTP Serve
```
 
 * show boot log
 
```
 journalctl -b | tail
 Jan 25 18:28:01 rhel7 systemd[1]: Created slice user-993.slice.
 Jan 25 18:28:01 rhel7 systemd[1]: Starting user-993.slice.
 Jan 25 18:28:01 rhel7 systemd[1]: Started Session 45 of user pcp.
 Jan 25 18:28:01 rhel7 systemd[1]: Starting Session 45 of user pcp.
 Jan 25 18:28:01 rhel7 CROND[6820]: (pcp) CMD ( /usr/libexec/pcp/bin/pmie_check -C)
 Jan 25 18:28:01 rhel7 systemd[1]: Removed slice user-993.slice.
 Jan 25 18:28:01 rhel7 systemd[1]: Stopping user-993.slice.
 Jan 25 18:30:01 rhel7 systemd[1]: Started Session 46 of user root.
 Jan 25 18:30:01 rhel7 systemd[1]: Starting Session 46 of user root.
 Jan 25 18:30:01 rhel7 CROND[6978]: (root) CMD (/usr/lib64/sa/sa1 1 1)
```
 
 * show error events
 
```
 journalctl -p err
 -- Logs begin at Thu 2018-01-25 14:28:07 EET, end at Thu 2018-01-25 18:30:01 EET. --
 Jan 25 14:28:10 rhel7 kernel: piix4_smbus 0000:00:07.3: Host SMBus controller not enabled!
 Jan 25 14:28:11 rhel7 kernel: intel_rapl: no valid rapl domains found in package 0
 Jan 25 14:28:35 rhel7 cntlm[1121]: Error creating a new PID file
 Jan 25 14:28:45 rhel7 spice-vdagent[2501]: Cannot access vdagent virtio channel /dev/virtio-ports/com.redhat.spice.0
 Jan 25 14:28:57 rhel7 udisksd[2652]: Error probing device: Error sending ATA command IDENTIFY PACKET DEVICE to /dev/sr0: ATA command failed: error=0x01 count=0x02 status=0x5
 Jan 25 14:47:48 rhel7 gdm-launch-environment][2380]: pam_systemd(gdm-launch-environment:session): Failed to release session: Interrupted system call
 Jan 25 14:47:50 rhel7 spice-vdagent[3306]: Cannot access vdagent virtio channel /dev/virtio-ports/com.redhat.spice.0
 Jan 25 14:48:15 rhel7 pulseaudio[3318]: GetManagedObjects() failed: org.freedesktop.DBus.Error.NoReply: Did not receive a reply. Possible causes include: the remote applicat
 Jan 25 14:53:38 rhel7 kernel: ata4: exception Emask 0x10 SAct 0x0 SErr 0x4010000 action 0xe frozen
 Jan 25 14:53:38 rhel7 kernel: ata4: irq_stat 0x00400040, connection status changed
 Jan 25 14:53:38 rhel7 kernel: ata4: SError: { PHYRdyChg DevExch }
```

#### rpm
 
> RPM Package Manager
 
* count all installed packages
 
```
rpqm -qa | wc -l
1366
```
 
* get info about installed package
 
```
 rpm -qi samba
 Name        : samba
 Epoch       : 0
 Version     : 4.4.4
 Release     : 14.el7_3
 Architecture: x86_64
 Install Date: Thu 08 Jun 2017 10:55:00 AM EEST
 Group       : System Environment/Daemons
 Size        : 1869228
 License     : GPLv3+ and LGPLv3+
 Signature   : RSA/SHA256, Thu 25 May 2017 04:16:22 PM EEST, Key ID 24c6a8a7f4a80eb5
 Source RPM  : samba-4.4.4-14.el7_3.src.rpm
 Build Date  : Thu 25 May 2017 02:35:40 PM EEST
 Build Host  : c1bm.rdu2.centos.org
 Relocations : (not relocatable)
 Packager    : CentOS BuildSystem <http://bugs.centos.org>
 Vendor      : CentOS
 URL         : http://www.samba.org/
 Summary     : Server and Client software to interoperate with Windows machines
 Description :
 Samba is the standard Windows interoperability suite of programs for Linux and
 Unix.
```
 
* list files in package
 
```
 rpm -ql lynx
 /etc/lynx-site.cfg
 /etc/lynx.cfg
 /etc/lynx.lss
 /usr/bin/lynx
 /usr/share/doc/lynx-2.8.8
 [...]
 /usr/share/locale/zh_TW/LC_MESSAGES/lynx.mo
 /usr/share/man/man1/lynx.1.gz
```
 
* list config files
 
```
 rpm -qc name
```
 
* install RPM package
 
```
 rpm -i pkgname
```
 
* upgrade package
 
```
 rpm -U pkgname
```
 
* remove package
 
```
 rpm -e pkgname
```
 
* see where file comes from
 
```
 rpm -qf /etc/httpd/conf
 httpd-2.4.6-45.el7.centos.4.x86_64
```
 
* verify package
 
```
rpm -V pgkname
```
 
* list documentation files
 
```
rpm -qd cmdname
```

#### yum
 
> Yellowdog Updater Modified
 
* install automatically
 
```
 yum install -y pkgname
```
 
* get info about package
 
```
 yum info nmap
 [...]
 Available Packages
 Name        : nmap
 Arch        : x86_64
 Epoch       : 2
 Version     : 6.40
 Release     : 7.el7
 Size        : 4.0 M
 Repo        : base/7/x86_64
 Summary     : Network exploration tool and security scanner
 URL         : http://nmap.org/
 License     : GPLv2 and LGPLv2+ and GPLv2+ and BSD
 Description : Nmap is a utility for network exploration or security auditing.
             : It supports ping scanning (determine which hosts are up), many
             : port scanning techniques (determine what services the hosts are
             : offering), and TCP/IP fingerprinting (remote host operating system
             : identification). Nmap also offers flexible target and port
             : specification, decoy scanning, determination of TCP sequence
             : predictability characteristics, reverse-identd scanning, and more.
             : In addition to the classic command-line nmap executable, the Nmap
             : suite includes a flexible data transfer, redirection, and
             : debugging tool (netcat utility ncat), a utility for comparing scan
             : results (ndiff), and a packet generation and response analysis
             : tool (nping).
```

* remove pkg
 
```
 yum remove pkgname
```
 
* list dependencies
 
```
 yum deplist cowsay
 [...]
 package: cowsay.noarch 3.04-4.el7
   dependency: /bin/sh
    provider: bash.x86_64 4.2.46-29.el7_4
   dependency: /usr/bin/perl
    provider: perl.x86_64 4:5.16.3-292.el7
   dependency: perl(Cwd)
    provider: perl-PathTools.x86_64 3.40-5.el7
   dependency: perl(File::Basename)
    provider: perl.x86_64 4:5.16.3-292.el7
   dependency: perl(Getopt::Std)
    provider: perl.x86_64 4:5.16.3-292.el7
   dependency: perl(Text::Tabs)
    provider: perl.x86_64 4:5.16.3-292.el7
   dependency: perl(Text::Wrap)
    provider: perl.x86_64 4:5.16.3-292.el7
   dependency: perl(strict)
    provider: perl.x86_64 4:5.16.3-292.el7
   dependency: perl-Encode
    provider: perl-Encode.x86_64 2.51-7.el7
```

* list installed packages
 
```
 yum list installed
 [...]
 yum.noarch                             3.4.3-150.el7.centos            @base    
 yum-langpacks.noarch                   0.4.2-7.el7                     @base    
 yum-metadata-parser.x86_64             1.1.4-10.el7                    @anaconda
 yum-plugin-fastestmirror.noarch        1.1.31-40.el7                   @base    
 yum-utils.noarch                       1.1.31-40.el7                   @base    
 zenity.x86_64                          3.8.0-5.el7                     @anaconda
 zip.x86_64                             3.0-11.el7                      @base    
 zlib.x86_64                            1.2.7-17.el7                    @base    
```
 
* update all packages
 
```
 yum update
```
 
* check if updates are available, without updating
 
```
 yum check-update
```
 
* list info about available packages
 
```
 yum list python
 [...]
 Installed Packages
 python.x86_64                         2.7.5-48.el7                         @base
 Available Packages
 python.x86_64                         2.7.5-58.el7                         base
 [root@localhost ~]#
```

* find out which package provides some feature or file
 
```
 yum provides scapy
 scapy-2.3.3-1.el7.noarch : Interactive packet manipulation tool and network
                          : scanner
 Repo        : epel
```
 
* find packages when you know something about  the package but aren't sure of its name.
 
```
 yum search name
```

* list package summary
 
```
 yum info scapy
 Available Packages
 Name        : scapy
 Arch        : noarch
 Version     : 2.3.3
 Release     : 1.el7
 Size        : 983 k
 Repo        : epel/x86_64
 Summary     : Interactive packet manipulation tool and network scanner
 URL         : http://www.secdev.org/projects/scapy/
 License     : GPLv2
 Description : Scapy is a powerful interactive packet manipulation program built
             : on top of the Python interpreter. It can be used to forge or
             : decode packets of a wide number of protocols, send them over the
             : wire, capture them, match requests and replies, and much more.
```
 
* find which package provides an utility
 
```
 yum provides sealert
 Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
 setroubleshoot-server-3.2.27.2-3.el7.x86_64 : SELinux troubleshoot server
 Repo        : rhel7-local
 Matched from:
 Filename    : /usr/bin/sealert
```

* perform cleaning operations
 
```
 yum clean
 all           dbcache       headers       packages      
 cache         expire-cache  metadata   
```
 
* Produces a list of all dependencies and  what  packages  provide those  dependencies  for the given packages.
 
``` 
yum deplist scapy
 package: scapy.noarch 2.3.3-1.el7
   dependency: /usr/bin/python
    provider: python.x86_64 2.7.5-58.el7
   dependency: python >= 2.5
    provider: python.x86_64 2.7.5-58.el7
   dependency: python(abi) = 2.7
    provider: python.x86_64 2.7.5-58.el7
```

* list configured repositories
 
```
 yum repolist
 repo id               repo name                                           status
 base/7/x86_64         CentOS-7 - Base                                      9,591
 *epel/x86_64          Extra Packages for Enterprise Linux 7 - x86_64      11,985
 extras/7/x86_64       CentOS-7 - Extras                                      227
 nux-dextop/x86_64     Nux.Ro RPMs for general desktop use                  1,599
 updates/7/x86_64      CentOS-7 - Updates                                     731
 repolist: 24,133
```
 
* see past transactions
 
```
 yum history
```
 
* list available kernels
 
```
 yum list kernel
 Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-
               : manager
 Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
 Installed Packages
 kernel.x86_64              3.10.0-514.el7                    @anaconda/7.3      
 kernel.x86_64              3.10.0-514.26.2.el7               @rhel-7-server-rpms
 Available Packages
 kernel.x86_64              3.10.0-693.2.2.el7                rhel-7-server-rpms
```

* install new kernel and set it as default boot choice

``` 
yum --enablerepo=kernelrepo install kernelname
```

Set <code>GRUB_DEFAULT=0</code> to make it default, 1 otherwise 


``` 
 _____________________________________
/ Fine day for friends. So-so day for \
\ you.                                /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


