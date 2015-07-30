---
layout: post
title: "OverTheWire: Bandit"
date: 2015-06-22 20:11:18 +0300
comments: true
categories: [overthewire, wargames, linux]
keywords: overthewire, wargames, bandit
description: OverTheWire Bandit wargame
---

I've completed this and some other wargames before starting a blog, but I thought I should revisit them and do a proper walkthrough, and that would also help me organize my notes beyond one-liners that I no longer know what they were for :D So, going to start with Bandit, which is the most basic and beginner friendly of the OverTheWire wargames. You can look at each level's page for a list of commands that you may need to solve it and some additional reading material that might help in better understanding what's going on. I will also give *man* pages descriptions for the commands I'll use to complete the levels.

<!-- more -->

### Level 0

The goal of this level is for you to log into the game using SSH. The host to which you need to connect is bandit.labs.overthewire.org. The username is bandit0 and the password is bandit0. Once logged in, go to the Level 1 page to find out how to beat Level 1.

### Level 0 -> Level 1

The password for the next level is stored in a file called readme located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH to log into that level and continue the game.

``` plain
ssh bandit0@bandit.labs.overthewire.org
Welcome to the OverTheWire games machine !

Please read /README.txt for more information on how to play the levels
on this gameserver.

bandit0@melinda:~$ ls
readme
bandit0@melinda:~$ cat readme
boJ9jbbUNNfktd78OOpsqOltutMc3MY1
```

Well, this is straightforward. The required file just lies around for the reading

**Useful command(s)**

> cat - concatenate files and print on the standard output

### Level 1 -> Level 2

The password for the next level is stored in a file called - located in the home directory

Although filenames starting with dashes are legal in Linux, if you try to use some commands on them, the dash would get confused with command flags. If you try to *cat* it directly, *cat* will just wait for further input. The workaround is to feed *cat* the path to the file (can also be done just by using the current directory path)

``` plain
bandit1@melinda:~$ ls
-
bandit1@melinda:~$ pwd
/home/bandit1
bandit1@melinda:~$ cat /home/bandit1/-
CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9
```

### Level 2 -> Level 3

The password for the next level is stored in a file called spaces in this filename located in the home directory

Spaces in filenames can be interpreted wrongly on the command line, because they may look as separators for the commands instead of literal spaces that are part of a filename, and this leads to all sorts of problems. That is why using spaces in filenames are generally avoided in Linux. If you try to *cat* the file as it is, this is what happens:

``` plain
bandit2@melinda:~$ ls 
spaces in this filename
bandit2@melinda:~$ cat spaces in this filename
cat: spaces: No such file or directory
cat: in: No such file or directory
cat: this: No such file or directory
cat: filename: No such file or directory
```

To solve the issue, you can either wrap the filenames in quotes or escape the spaces with backslashes, to ensure that the name of the file is passed correctly to the command (and if you use Tab completion, the shell will automatically escape them for you :D)

``` plain
bandit2@melinda:~$ cat 'spaces in this filename'
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
bandit2@melinda:~$ cat spaces\ in\ this\ filename 
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
```

### Level 3 -> Level 4

The password for the next level is stored in a hidden file in the inhere directory.

If you do a normal *ls*, you won't see anything. To see hidden files, you use the *-a* option, which stands for *--all*:

``` plain
bandit3@melinda:~$ ls
inhere
bandit3@melinda:~$ cd inhere
bandit3@melinda:~/inhere$ ls -a
.  ..  .hidden
bandit3@melinda:~/inhere$ cat .hidden
pIwrPrtPN36QITSp3EQaw936yaFoFgAB
```

**Useful command(s)**

> ls - list directory contents

> -a, --all
> do not ignore entries starting with .

### Level 4 - Level 5

The password for the next level is stored in the only human-readable file in the inhere directory. Tip: if your terminal is messed up, try the “reset” command.

``` plain
bandit4@melinda:~/inhere$ ls
-file00  -file02  -file04  -file06  -file08
-file01  -file03  -file05  -file07  -file09
bandit4@melinda:~/inhere$ cat ./-file00
;�-i�(��z��У��ޘ�鑾
```

You can try to manually *cat* each of them, until you will reach the right one:

``` plain
bandit4@melinda:~/inhere$ cat ./-file07
koReBOKuIDDepwhWk7jZC0RTdopnAYKh
```

But what if there were hundreds of files? If you use the *file* command, you can see the difference between the files:

``` plain
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
```

Now you know which file to look in for the password!

**Useful command(s)**

> file — determine file type

### Level 5 -> Level 6

The password for the next level is stored in a file somewhere under the inhere directory and has all of the following properties: - human-readable - 1033 bytes in size - not executable

Well, clearly no manual work here, the directory is filled with other folders:

``` plain
bandit5@melinda:~/inhere$ ls
maybehere00  maybehere04  maybehere08  maybehere12  maybehere16
maybehere01  maybehere05  maybehere09  maybehere13  maybehere17
maybehere02  maybehere06  maybehere10  maybehere14  maybehere18
maybehere03  maybehere07  maybehere11  maybehere15  maybehere19
```

To find the file with the required properties, we can use..*find*! It conveniently has the exact switches for what we're looking for:

``` plain
bandit5@melinda:~/inhere$ find ! -executable -readable -size 1033c
./maybehere07/.file2
bandit5@melinda:~/inhere/maybehere07$ cat .file2
DXjZPULLxYr17uwoI01bNLQbtFemEgo7
```

**Useful command(s)**

> find - search for files in a directory hierarchy

> -executable
> Matches files which are executable  and  directories  which  are searchable  (in  a file name resolution sense).

> -readable
> Matches  files  which  are  readable.

> -size n
> File uses n units of space.
> `c'    for bytes

### Level 6 -> Level 7

The password for the next level is stored somewhere on the server and has all of the following properties: - owned by user bandit7 - owned by group bandit6 - 33 bytes in size

Again, *find* comes to the rescue!

``` plain
bandit6@melinda:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
bandit6@melinda:~$ cat /var/lib/dpkg/info/bandit7.password
HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs
```

**Useful command(s)**

> find

> -user uname

> File is owned by user uname (numeric user ID allowed).

> -group gname

> File belongs to group gname (numeric group ID allowed).

### Level 7 -> Level 8

The password for the next level is stored in the file data.txt next to the word millionth

We can use *grep* to get the line with the millionth word and our password:

``` plain
bandit7@melinda:~$ cat data.txt | grep millionth
millionth	cvX2JJa4CFALtqS87jk27qwqGhBM9plV
```

**Useful command(s)**

> grep  searches the named input FILEs (or standard input if no files are
> named, or if a single hyphen-minus (-) is given as file name) for lines
> containing  a  match to the given PATTERN.  By default, grep prints the
> matching lines.

### Level 8 -> Level 9

The password for the next level is stored in the file data.txt and is the only line of text that occurs only once

To get only the unique line(s), we will use some more pipe redirection:

``` plain
bandit8@melinda:~$ sort data.txt | uniq -u
UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR
```

**Useful command(s)**

> sort - sort lines of text files

> uniq - report or omit repeated lines
> -u, --unique
> only print unique lines

### Level 9 -> Level 10

The password for the next level is stored in the file data.txt in one of the few human-readable strings, beginning with several ‘=’ characters.

For this one we can use *strings* and *grep* for the = sign:

``` plain
bandit9@melinda:~$ strings data.txt | grep =
epr~F=K
7?YD=
?M=HqAH
/(Ne=
C=_"
I========== the6
z5Y=
`h(8=`
n\H=;
========== password
========== ism
N$=&
l/a=L)
f=C(
========== truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk
ie)=5e
```

**Useful command(s)**

> strings - print the strings of printable characters in files.

### Level 10 -> Level 11

The password for the next level is stored in the file data.txt, which contains base64 encoded data

Luckily, there is a command-line utility just for that purpose!

``` plain
bandit10@melinda:~$ base64 -d data.txt
The password is IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR
```

**Useful command(s)**

> base64 - base64 encode/decode data and print to standard output

> -d, --decode
> decode data

### Level 11 -> Level 12

The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions

If you read the ROT13 Implementation section on Wikipedia, it will actually give you a hint on how to solve this challenge and the program needed.

``` plain
bandit11@melinda:~$ cat data.txt | tr a-zA-Z n-za-mN-ZA-M
The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu
```

Because the content of the file has been rotated 13 characters, we use the *tr* command to shift it back to the original

**Useful command(s)**

> tr - translate or delete characters

> CHAR1-CHAR2
> all characters from CHAR1 to CHAR2 in ascending order

### Level 12 -> Level 13

The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work using mkdir. For example: mkdir /tmp/myname123. Then copy the datafile using cp, and rename it using mv (read the manpages!)

If you *cat* data.txt this is what you will see:

``` plain
bandit12@melinda:~$ cat data.txt
0000000: 1f8b 0808 34da 6554 0203 6461 7461 322e  ....4.eT..data2.
0000010: 6269 6e00 013f 02c0 fd42 5a68 3931 4159  bin..?...BZh91AY
0000020: 2653 5982 c194 8a00 0019 ffff dbfb adfb  &SY.............
0000030: bbab b7d7 ffea ffcd fff7 bfbf 1feb eff9  ................
0000040: faab 9fbf fef2 fefb bebf ffff b001 3b18  ..............;.
0000050: 6400 001e a000 1a00 6468 0d01 a064 d000  d.......dh...d..
0000060: 0d00 0034 00c9 a320 001a 0000 0d06 80d1  ...4... ........
0000070: a340 01b4 98d2 3d13 ca20 6803 40d1 a340  .@....=.. h.@..@
0000080: 1a00 0340 0d0d 0000 000d 0c80 6803 4d01  ...@........h.M.
0000090: a3d4 d034 07a8 0683 4d0c 4034 069e 91ea  ...4....M.@4....
00000a0: 0f50 1a1a 1ea3 40e9 ea0c 80d0 0346 87a9  .P....@......F..
00000b0: a006 8193 4340 d320 c403 2064 00c4 000c  ....C@. .. d....
00000c0: 8640 0d00 0d06 8340 0c9a 0068 0000 6468  .@.....@...h..dh
00000d0: 1854 0084 0008 38c4 7c28 66b3 bf1f 366d  .T....8.|(f...6m
00000e0: 3971 1c93 f09a 6287 0cfe 04d3 efa9 4164  9q....b.......Ad
00000f0: 0ad1 1828 6c55 75ff 6922 dedd 8cfe 5936  ...(lUu.i"....Y6
0000100: e351 7ae8 0590 6c01 0446 5f2a ba7e 8503  .Qz...l..F_*.~..
0000110: a710 a38c d8c1 9781 5249 b909 8d92 5e09  ........RI....^.
0000120: b343 32a1 9890 cc63 74f2 a3a1 f260 3afa  .C2....ct....`:.
0000130: 4f55 cc30 f7a3 5c20 d610 a588 1ab4 543c  OU.0..\ ......T<
0000140: 71b3 d052 8980 010a b270 4112 89c4 ad7a  q..R.....pA....z
0000150: 8386 125d a460 3a11 3da3 4949 a01f 9e7d  ...].`:.=.II...}
0000160: 8f5e fef5 e13a 4537 dfb3 a898 92e8 cca0  .^...:E7........
0000170: 155c fb29 d0e1 08cf 0cec 7006 b1bc 8f39  .\.)......p....9
0000180: 51bc 1b7b e1ef 161f f020 6830 b1fd d69c  Q..{..... h0....
0000190: e096 54a1 1a03 47ce c4f1 00c7 e520 2e02  ..T...G...... ..
00001a0: 5577 63ac 3dc9 0f84 200a 745d 0503 f8f4  Uwc.=... .t]....
00001b0: b9fb 1152 1c22 a410 572e 11ac cf9e 5ff6  ...R."..W....._.
00001c0: dbf4 ef68 3010 7e36 026e aa38 19fd 4c37  ...h0.~6.n.8..L7
00001d0: 392c a262 f646 8710 9231 4ee4 5200 c601  9,.b.F...1N.R...
00001e0: 529a fec3 8c89 f85d 5f12 5c2f 9073 4544  R......]_.\/.sED
00001f0: 4fed fb97 a851 f831 cd9a 69d7 e80b 12b5  O....Q.1..i.....
0000200: fb37 ba20 86e9 92a7 78c5 5092 2bac 6269  .7. ....x.P.+.bi
0000210: 01c7 09a1 fda4 ef8b 7c14 1832 a30f db92  ........|..2....
0000220: d345 a9b4 de57 8996 4dc7 8ee8 b334 02b2  .E...W..M....4..
0000230: 8dc4 a6a6 08ea c285 d28c 9f60 6779 540a  ...........`gyT.
0000240: 2b97 5e3f f82c 1800 80f1 32b0 32d1 7724  +.^?.,....2.2.w$
0000250: 5385 0908 2c19 48a0 d123 d96f 3f02 0000  S...,.H..#.o?...
```

So, if it's been repeatedly compressed, than repeatedly decompressing it should do the job (this actually took an annoying time of repetitions...am I repeating the repeat word too often? :D)

``` plain
bandit12@melinda:~$ mkdir /tmp/mystuff
bandit12@melinda:~$ cp data.txt /tmp/mystuff
bandit12@melinda:~$ cd /tmp/mystuff
```

To reverse the hexdump we will use *xxd*:

``` plain
bandit12@melinda:/tmp/mystuff$ xxd -r data.txt > newdata
```

Now let's look at it (not literally, it's full of garbage):

``` plain
bandit12@melinda:/tmp/mystuff$ file newdata
newdata: gzip compressed data, was "data2.bin", from Unix, last modified: Fri Nov 14 10:32:20 2014, max compression
```

So now we know the data file was previously a binary file and it's been compressed with *gzip*. This means we know how to decompress it. There are a couple ways to do it. If you want to use *gzip*, you have to rename the file with a *.gz* extension:

``` plain
bandit12@melinda:/tmp/mystuff$ mv newdata data2.gz
bandit12@melinda:/tmp/mystuff$ gzip -d data3.gz
bandit12@melinda:/tmp/mystuff$ file data3
data3: bzip2 compressed data, block size = 900k
```

You can also use *zcat* directly on the file without adding any extension:

``` plain
bandit12@melinda:/tmp/mystuff$ zcat newdata > data3
bandit12@melinda:/tmp/mystuff$ file data3
data3: bzip2 compressed data, block size = 900k
```

Since we know the program used to compress it, we use the same for the reverse:

``` plain
bandit12@melinda:/tmp/mystuff$ bzip2 -d data3
bzip2: Can't guess original name for data3 -- using data3.out
bandit12@melinda:/tmp/mystuff$ file data3.out 
data3.out: gzip compressed data, was "data4.bin", from Unix, last modified: Fri Nov 14 10:32:20 2014, max compression
```

We've been through this kind of decompression before:

``` plain
bandit12@melinda:/tmp/mystuff$ zcat data3.out > data5
bandit12@melinda:/tmp/mystuff$ file data5
data5: POSIX tar archive (GNU)
```

Next we have a tar archive and we will just loop decompressions until we're done:

``` plain
bandit12@melinda:/tmp/mystuff$ tar xvf data5
data5.bin
bandit12@melinda:/tmp/mystuff$ file data5.bin
data5.bin: POSIX tar archive (GNU)
bandit12@melinda:/tmp/mystuff$ tar xvf data5.bin
data6.bin
bandit12@melinda:/tmp/mystuff$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
bandit12@melinda:/tmp/mystuff$ bzip2 -d data6.bin
bzip2: Can't guess original name for data6.bin -- using data6.bin.out
bandit12@melinda:/tmp/mystuff$ file data6.bin.out
data6.bin.out: POSIX tar archive (GNU)
bandit12@melinda:/tmp/mystuff$ tar xvf data6.bin.out
data8.bin
bandit12@melinda:/tmp/mystuff$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", from Unix, last modified: Fri Nov 14 10:32:20 2014, max compression
bandit12@melinda:/tmp/mystuff$ zcat data8.bin > data9
bandit12@melinda:/tmp/mystuff$ file data9
data9: ASCII text
bandit12@melinda:/tmp/mystuff$ cat data9
The password is 8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL
```

Finally! Had enough decompressions for one day.

**Useful command(s)**

> xxd - make a hexdump or do the reverse.

> -r | -revert

> reverse operation: convert (or patch) hexdump into binary. If not writing to stdout, xxd writes into its output file without truncating it.


> mv - move (rename) files


> gzip - compress or expand files. Whenever possible, each file is replaced by one with the extension .gz. By default, gzip keeps the original file
> name and timestamp in the compressed  file. Compressed files can be restored to their original form using gzip -d  or gunzip or zcat.

> -d --decompress --uncompress
> Decompress.


> zcat uncompresses either a list of files on the command line or its standard input and writes the uncompressed data on standard output. zcat will
> uncompress files that have the correct magic number whether they have a .gz suffix or not.


> bzip2 - a block-sorting file compressor

> bzip2 expects a list of file names to accompany the command-line flags. Each  file is replaced by a compressed version of itself, with the name
> "original_name.bz2".

> If  the  file does not end in one of the recognised endings, .bz2, .bz, .tbz2 or .tbz, bzip2 complains that it cannot guess the name  of the
> original file, and uses the original name with .out appended.

> -d --decompress
> Force  decompression.


> Tar stores and extracts files from a tape or disk archive.

> -x, --extract, --get
> extract files from an archive

> -v, --verbose
> verbosely list files processed

> -f, --file ARCHIVE
use archive file or device ARCHIVE


### Level 13 -> Level 14

The password for the next level is stored in /etc/bandit_pass/bandit14 and can only be read by user bandit14. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. Note: localhost is a hostname that refers to the machine you are working on

Ok, we have a private key:

``` plain
bandit13@melinda:~$ ls
sshkey.private
bandit13@melinda:~$ file sshkey.private
sshkey.private: PEM RSA private key
```

The description hinted that we need to use the private key to SSH as bandit14 and read the password, and also mentioned localhost. So let's ssh to localhost:

``` plain
ssh bandit14@localhost -i /home/bandit13/sshkey.private
bandit14@melinda:~$ cat /etc/bandit_pass/bandit14
4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e
```

**Useful command(s)**

> ssh — OpenSSH SSH client (remote login program)

> -i identity_file
> Selects a file from which the identity (private key) for public key authentication is read.


### Level 14 -> Level 15

The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

This level is straightforward since we have netcat available:

``` plain
bandit14@melinda:~$ nc -v localhost 30000
Connection to localhost 30000 port [tcp/*] succeeded!
4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e
Correct!
BfMYroe26WYalil77FoDi9qh59eK5xNr
```

**Useful command(s)**

> The nc (or netcat) utility is used for just about anything under the sun involving TCP, UDP, or UNIX-domain sockets.  It can open TCP connections,
> send UDP packets, listen on arbitrary TCP and UDP ports, do port scanning, and deal with both IPv4 and IPv6. 


### Level 15 -> Level 16

The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL encryption.

Helpful note: Getting “HEARTBEATING” and “Read R BLOCK”? Use -quiet and read the “CONNECTED COMMANDS” section in the manpage. Next to ‘R’ and ‘Q’, the ‘B’ command also works in this version of that command…

For this the *openssl* command line utility will come in handy:

``` plain
The openssl program is a command line tool for using the various cryptography functions of OpenSSL's crypto library from the shell. It can be used for

        o  Creation and management of private keys, public keys and parameters
        o  Public key cryptographic operations
        o  Creation of X.509 certificates, CSRs and CRLs
        o  Calculation of Message Digests
        o  Encryption and Decryption with Ciphers
        o  SSL/TLS Client and Server Tests
        o  Handling of S/MIME signed or encrypted mail
        o  Time Stamp requests, generation and verification
```

In particular, we will use the <code>s_client</code> command which is very useful for SSL servers testing and diagnostics. We use it to connect to localhost on the specified port:

``` plain
bandit15@melinda:~$ openssl s_client -quiet -connect localhost:30001
depth=0 CN = li190-250.members.linode.com
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = li190-250.members.linode.com
verify return:1
BfMYroe26WYalil77FoDi9qh59eK5xNr
Correct!
cluFn7wTiGryunymYOu4RcffSxQluehd

read:errno=0
```

Without the *-quiet* flag we would get a ton of information and instead of the password we would see some HEARTBEATING and read R BLOCK messages

**Useful command(s)**

> openssl - OpenSSL command line tool

> s_client  This implements a generic SSL/TLS client which can establish a transparent connection to a remote server speaking SSL/TLS.
> It's intended for testing purposes only and provides only rudimentary interface functionality but internally uses
> mostly all functionality of the OpenSSL ssl library.

> -connect host:port
> This specifies the host and optional port to connect to. If not specified then an attempt is made to connect to the local host on port 4433

> -quiet
> inhibit printing of session and certificate information. This implicitly turns on -ign_eof as well.


More information (along with the CONNECTED COMMANDS section) can be found on https://openssl.org/docs/apps/s_client.html#options


### Level 16 -> Level 17

The password for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

Fortunately, we have nmap installed, so scanning the port range is easy!

``` plain
bandit16@melinda:~$ nmap localhost -p31000-32000

Starting Nmap 6.40 ( http://nmap.org ) at 2015-07-27 13:56 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00087s latency).
Not shown: 996 closed ports
PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 0.12 seconds
```

Using netcat to probe all those ports, I found out that ports 31518 and 31790 use SSL from the following error: 

``` plain
ERROR
140737354045088:error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number:s3_pkt.c:350:
```

The rest of the ports just echo back what you send them. So now let's feed the password to these ports and see which one will give the answer:

``` plain
bandit16@melinda:~$ openssl s_client -quiet -connect localhost:31518
depth=0 CN = li190-250.members.linode.com
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = li190-250.members.linode.com
verify return:1
cluFn7wTiGryunymYOu4RcffSxQluehd
cluFn7wTiGryunymYOu4RcffSxQluehd
```

So, port 31518 just returns the string you give it. Must be the other one:

``` plain
bandit16@melinda:~$ openssl s_client -quiet -connect localhost:31790
depth=0 CN = li190-250.members.linode.com
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = li190-250.members.linode.com
verify return:1
cluFn7wTiGryunymYOu4RcffSxQluehd
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
-----END RSA PRIVATE KEY-----

read:errno=0
```

Instead of a password we got a SSH private key. I copied it in my <code>.ssh</code> folder, gave it right permissions to stop the WARNING: UNPROTECTED PRIVATE KEY FILE! message, and used it to log in as the next level and get the password:

``` plain
root@kali:~/.ssh# chmod 600 bandit17
root@kali:~/.ssh# ssh -i ~/.ssh/bandit17 bandit17@bandit.labs.overthewire.org
bandit17@melinda:~$ cat /etc/bandit_pass/bandit17
xLYVMN9WE5zQ5vHacb0sZEVqbrp7nBTn
```

**Useful command(s)**

> nmap - Network exploration tool and security / port scanner

> -p <port ranges>: Only scan specified ports
> Ex: -p22; -p1-65535; -p U:53,111,137,T:21-25,80,139,8080,S:9

### Level 17 -> Level 18

There are 2 files in the homedirectory: passwords.old and passwords.new. The password for the next level is in passwords.new and is the only line that has been changed between passwords.old and passwords.new

NOTE: if you have solved this level and see ‘Byebye!’ when trying to log into bandit18, this is related to the next level, bandit19

We can use the *diff* program to see the differences between the 2 files:

``` plain
bandit17@melinda:~$ ls
passwords.new  passwords.old
bandit17@melinda:~$ diff passwords.old passwords.new 
42c42
< BS8bqB1kqkinKJjuxL6k072Qq9NRwQpR
---
> kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd
```

The 42c42 string means that line 42 in the first file was changed to line 42 in the second. diff uses some characters to signify the kind of change that was found:

``` plain
a – line was added
c – line was changed
d – line was deleted
```

The number to the left of the character represents the line number in the first file, while the one to the right refers to the line number of the second file. So the correct password is kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd

**Useful command(s)**

> diff - compare files line by line


### Level 18 -> Level 19

The password for the next level is stored in a file readme in the homedirectory. Unfortunately, someone has modified .bashrc to log you out when you log in with SSH.

Upon logging in, we are instantly disconnected and get a bye message:

``` plain
Byebye !
Connection to bandit.labs.overthewire.org closed.
```

The way to run a command immediately after logging in is to add it after the ssh command:

``` plain
root@kali:~/.ssh# ssh bandit18@bandit.labs.overthewire.org cat readme
This is the OverTheWire game server. More information on http://www.overthewire.org/wargames

Please note that wargame usernames are no longer level<X>, but wargamename<X>
e.g. vortex4, semtex2, ...

Note: at this moment, blacksun is not available.

bandit18@bandit.labs.overthewire.org's password: 
IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x
```


### Level 19 -> Level 20

To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used to setuid binary.

``` plain
Run a command as another user.
  Example: ./bandit20-do id
bandit19@melinda:~$ ls -l bandit20-do 
-rwsr-x--- 1 bandit20 bandit19 7370 Nov 14  2014 bandit20-do
```

The binary is a setuid binary owned by the bandit20 user, which means we can use it to directly read he password as if we were bandit20. No exploitation needed!

``` plain
bandit19@melinda:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
GbKksEFF4yrVs6il55v6gwY5aVje5f0j
```


### Level 20 -> Level 21

There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

NOTE: To beat this level, you need to login twice: once to run the setuid command, and once to start a network daemon to which the setuid will connect.

NOTE 2: Try connecting to your own network daemon to see if it works as you think

Here we will need to use 2 connections because we need to set up a listener as well. But since we have netcat, all is well! :D

``` plain
bandit20@melinda:~$ ./suconnect 
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP. If it receives the correct password from the other side, the next password is transmitted back.
```

Have netcat listen on a port:

``` plain
bandit20@melinda:~$ nc -vnlp 7777
Listening on [0.0.0.0] (family 0, port 7777)

```

Then use the setuid program to connect from a new shell to the netcat listener. We see on the netcat side the connection has been received and if we give it the password we will receive the next one:

``` plain
Connection from [127.0.0.1] port 7777 [tcp/*] accepted (family 2, sport 41986)
GbKksEFF4yrVs6il55v6gwY5aVje5f0j
gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr
```

You can see on the other shell with the setuid binary that the password matched:

``` plain
bandit20@melinda:~$ ./suconnect 7777
Read: GbKksEFF4yrVs6il55v6gwY5aVje5f0j
Password matches, sending next password
```


### Level 21 -> Level 22

A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

If we look in <code>/etc/cron.d/</code> we see a bunch of files, we are interested in the cronjob for bandit22:

``` plain
bandit21@melinda:/etc/cron.d$ cat cronjob_bandit22
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

So cron runs a shell script as the bandit22 user. Let's see what it is:

``` plain
bandit21@melinda:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

So the bandit22 user decided to paste his password in a random named file in */tmp/*, but he gave everyone the permission to read it!

``` plain
bandit21@melinda:/etc/cron.d$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
Yk7owGAcWjwMVRwrTesJEwB7WVOiILLI
```

**Useful command(s)**

> cron - daemon to execute scheduled commands 

> A crontab file contains instructions to the cron(8) daemon of the general form: "run this command at this time on this date". Each user
> has their own crontab, and commands in any given crontab will be executed as the user who owns the crontab.

> The format of a cron command is very much the V7 standard, with a number of upward-compatible extensions. Each line has five time and
> date fields, followed by a command,  followed by a newline character ('\n'). The system crontab (/etc/crontab) uses the same format,
> except that the username for the command is specified after the time and date fields and before the command. The fields may be separated
> by spaces or tabs.

> Commands  are executed by cron(8) when the minute, hour, and month of year fields match the current time, and when at least one of the two
> day fields (day of month, or day of week) match the current time. cron(8) examines cron entries once every minute.

> A field may be an asterisk (*), which always stands for "first-last".



### Level 22 -> Level 23

A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

NOTE: Looking at shell scripts written by other people is a very useful skill. The script for this level is intentionally made easy to read. If you are having problems understanding what it does, try executing it to see the debug information it prints.

Again, we identify the cronjob file first to know where to look further:

``` plain
bandit22@melinda:/etc/cron.d$ cat cronjob_bandit23
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
```

This script is more involved than the last:

``` plain
bandit22@melinda:/etc/cron.d$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

So, this script assigns the current user name to the myname variable. We know that's bandit22. And then copies the password to a file in */tmp/* that we have to identify. We can do this by substitution:

myname=bandit23 (that is what the *whoami* command will return)

Then the string "I m user bandit23" is fed to md5sum to compute a MD5 hash. You can check the output on your system:

``` plain
root@kali:~# echo "I am user bandit23" | md5sum 
8ca319486bfbbc3663ea0fbe81326349  -
```

The cut command is used to print just the hash line:

``` plain
root@kali:~# echo "I am user bandit23" | md5sum | cut -d ' ' -f 1
8ca319486bfbbc3663ea0fbe81326349
```

Now we know that mytarget=8ca319486bfbbc3663ea0fbe81326349

``` plain
bandit22@melinda:/etc/cron.d$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
jc1udXuA1tiHqjIsL8yaapX5XIAI6i0n
```

**Useful command(s)**

> whoami - Print the user name associated with the current effective user ID

> echo - display a line of text

>  md5sum - compute and check MD5 message digest

> cut - remove sections from each line of files

> -d, --delimiter=DELIM
> use DELIM instead of TAB for field delimiter

> -f, --fields=LIST
> select only these fields;  also print any line that contains no delimiter character, unless the -s option is specified


### Level 23 -> Level 24

A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

NOTE: This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level!

NOTE 2: Keep in mind that your shell script is removed once executed, so you may want to keep a copy around…

The beginning is the same as last levels:

``` plain
bandit23@melinda:/etc/cron.d$ cat cronjob_bandit24
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
bandit23@melinda:/etc/cron.d$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname
echo "Executing and deleting all scripts in /var/spool/$myname:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
	echo "Handling $i"
	timeout -s 9 60 "./$i"
	rm -f "./$i"
    fi
done
```

So this script executes and then deletes all scripts in the /var/spool/bandit24 directory. We can place a script of our own in there to copy the password of the bandit24 user in a location we have access to. I made a directory in /tmp/ first:

``` plain
bandit23@melinda:/tmp$ mkdir stuff
bandit23@melinda:/tmp$ cd stuff
```

Then I made a file that will hold the password later and gave it the most liberal permissions possible:

``` plain
bandit23@melinda:/tmp/stuff$ touch readme.txt
bandit23@melinda:/tmp/stuff$ chmod 777 readme.txt
```

Next, I made a script that will be executed by cron. It just copies the bandit24 password to the file I've created earlier.


``` plain
bandit23@melinda:/tmp/stuff$ cat > exeme.sh
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/stuff/readme.txt
```

I gave my script super permissive rights, then copied it to */var/spool/bandit24* to be executed:

``` plain
bandit23@melinda:/tmp/stuff$ chmod 777 exeme.sh
bandit23@melinda:/tmp/stuff$ cp exeme.sh /var/spool/bandit24/
```

Waited a bit, then profit:

``` plain
bandit23@melinda:/tmp/stuff$ cat readme.txt
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ
```


### Level 24 -> Level 25

A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinaties, called brute-forcing.

``` plain
bandit24@melinda:~$ nc -vv localhost 30002
Connection to localhost 30002 port [tcp/*] succeeded!
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
```

Meh, this means we'll have to bruteforce the pincode and try until the daemon says it's correct. I wrote a quick Python script for that:

``` python
#!/usr/bin/python

import itertools
import socket



PIN_CHARS = '0123456789' # digits that can be in a pin
PIN_LEN = 4 # pin length

wrong = 'Wrong!' # part of the msg received for wrong data
fail = 'Fail!' # msg received if data doesn't respect constraints
normal = 'I am the pincode checker' # normal msg
host = '127.0.0.1'
port = 30002

def computePins(iterables, r):
    """
    Build a list of all possible pin combinations that meet the constraints
    """
    pins = []
    for pin in itertools.product(iterables, repeat = r):
        pins.append(''.join(pin))
    return pins

pinlist = computePins(PIN_CHARS, PIN_LEN)



s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((host, int(port)))


for pin in pinlist:
    msg = 'UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ %s\n' % pin
    reply = s.recv(1024)
    s.send(msg)
    if wrong not in reply and fail not in reply and normal not in reply:
        print msg, reply
        break
```

And running it:

``` plain
bandit24@melinda:/tmp/b24mine$ ./bandit24.py 
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 5670
Correct!
The password of user bandit25 is uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG
```


### Level 25 -> Level 26

Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not /bin/bash, but something else. Find out what it is, how it works and how to break out of it.

After logging in I found a SSH private key for level 26 just lying around:

``` plain
bandit25@melinda:~$ ls
bandit26.sshkey
```

But when using it to log in, the connection closes immediately, after showing this:

``` plain
  _                     _ _ _   ___   __  
 | |                   | (_) | |__ \ / /  
 | |__   __ _ _ __   __| |_| |_   ) / /_  
 | '_ \ / _` | '_ \ / _` | | __| / / '_ \ 
 | |_) | (_| | | | | (_| | | |_ / /| (_) |
 |_.__/ \__,_|_| |_|\__,_|_|\__|____\___/ 
Connection to bandit.labs.overthewire.org closed.
```

After reading a bit about how to find out another user's shell, what worked was looking in <code>/etc/passwd</code> for that specific user and checking the shell field (last one):

``` plain
cat /etc/passwd | grep bandit26
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
```

Cool, we have something! Let's look at it:

``` plain
bandit25@melinda:~$ cat /usr/bin/showtext
#!/bin/sh

more ~/text.txt
exit 0
```

So this shell uses *more* to display the contents of the text.txt file. Since *more* only displays one screen at a time, we want to see what else is contained in that file, but we don't have access to it, nor can we do anything about the shell. So the only way to proceed is to thoroughly read the *more* manpage and see if we can find something useful..

So, apparently it's possible to start an editor from more, and that rang a bell because *vi* was listed among the commands that might be needed to solve this level. And then maybe we can see inside the file some more (see what I did there? xD)

Since I wasn't sure how to proceed on the bandit system, I tested it on my own box first, by using *more* on a file large enough to activate it:

{% img center /images/overthewire/bandit/more.png 'more' 'more' %}

And pressing v dropped me into Vim!

{% img center /images/overthewire/bandit/vi.png 'vi' 'vi' %}

That gives a way to view the file beyond the first screen. But in the login shell, there was no interaction, I couldn't get *more* to step in..wouldn't even have known about it without checking the shell. But when I played around on my box some more, if using *more* on a very small file, it would just output its contents, the same way as *cat*, so there would be no indication that *more* was used to display it. Just like via SSH! So it means that text.txt file doesn't really have anything than that bandit ASCII art. But I wanted to check, and wasn't sure how to enter into the *more* prompt by logging in...it turns out that can be done by making the terminal window small!

{% img center /images/overthewire/bandit/small_more.png 'small login for more' 'small is more' %}

{% img center /images/overthewire/bandit/more_shell.png 'more' 'more' %}

Bingo! From here we can drop in *vi*! But as expected, there is nothing else in the file. I admit I don't use *vi* and the few times I needed to use it for something I had to follow instructions by looking on the internet, so it took me a little more reading before I stumbled upon [this very useful SO post](http://stackoverflow.com/questions/1169805/how-to-load-another-files-content-to-current-file-in-vim). It's possible to read another file by inserting it into the current file! You can do this by typing <code>:r newfile</code>:

{% img center /images/overthewire/bandit/vi_read.png 'vi read' 'vi read' %}

Next I had to scroll through some warnings about multiple swap files, and at the end I saw the password inserted where the cursor was! 

{% img center /images/overthewire/bandit/bandit26_pass.png 'bandit 26 password' 'bandit 26' %}

So the password is *5czgV9L3Xx8JPOyRbXh6lQbmIOWvPT6Z*, and this was a hell of a fun level to complete! :D

**Useful command(s)**

> The /etc/passwd file is a text file that describes user login accounts for the system.

>  Each line of the file describes a single user, and contains seven colon-separated fields:

> name:password:UID:GID:GECOS:directory:shell

> shell       This is the program to run at login


> more is a filter for paging through text one screenful at a time. This version is especially primitive.

> Interactive commands for more are based on vi(1). 

> v           Start up an editor at current line. The editor is taken from the environment variable VISUAL if defined, or EDITOR if VISUAL is
> not defined, or defaults to "vi" if neither VISUAL nor EDITOR is defined.


### Level 26 -> Level 27

At this moment, level 27 does not exist yet.


Ok, coming back to this was lots of fun, and 2 more levels have been added since I solved it the first time, so perhaps more will be added in the future as well. The bandit wargame has been one of my favorites, and level 26 was really interesting!


{% img center /images/overthewire/bandit/cookie.png 'fortune cookie' 'cookie' %}
