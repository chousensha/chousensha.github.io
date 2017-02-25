---
layout: post
title: "Setting up MySQL and PHP on Apache"
date: 2017-02-25 03:58:37 -0500
comments: true
categories: [linux, sysadmin]
keywords: mysql, php, apache, lamp, lamp stack, sql server, mysql centos, mysql linux, php linux, php centos, install mysql, install php
description: Installing and configuring MySQL server and PHP on CentOS 7
---



It's been a while since my last post, but a full time job + the Cisco Cybersecurity scholarship are really eating into my time. But I will try to sneak a post here and there, whenever I can!

In a [previous post](http://chousensha.github.io/blog/2016/08/05/getting-started-with-apache/), I made a tutorial about setting up Apache on CentOS. The next step is to fire a MySQL server and put some databases on that web server!

<!-- more -->

## Installing MySQL server

First, we need to visit the [MySQL community repository](https://dev.mysql.com/downloads/repo/yum/) to download the server package:

``` plain
wget https://repo.mysql.com/mysql57-community-release-el7-9.noarch.rpm
--2017-01-19 10:06:02--  https://repo.mysql.com/mysql57-community-release-el7-9.noarch.rpm
Resolving repo.mysql.com (repo.mysql.com)... 104.87.9.47
Connecting to repo.mysql.com (repo.mysql.com)|104.87.9.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9224 (9.0K) [application/x-redhat-package-manager]
Saving to: ‘mysql57-community-release-el7-9.noarch.rpm’

100%[===========================================================================================================>] 9,224       --.-K/s   in 0s      

2017-01-19 10:06:02 (137 MB/s) - ‘mysql57-community-release-el7-9.noarch.rpm’ saved [9224/9224]
```

Next we install the package:

``` plain
rpm -ivh mysql57-community-release-el7-9.noarch.rpm 
warning: mysql57-community-release-el7-9.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:mysql57-community-release-el7-9  ################################# [100%]
```

After running <code>yum update</code> and waiting for a while, you can install MySQL server from the newly added repository with <code>yum install mysql-server</code>.

After the installation, a new user has been created in <code>/etc/passwd</code>, and a group in <code>/etc/group</code>:

``` plain
cat /etc/passwd | grep mysql
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/false

cat /etc/group | grep mysql
mysql:x:27:
```

## Configuration

Next, start the service with <code>systemctl start mysqld</code>. The next step is to run the <code>mysql_secure_installation</code> binary to make some security configurations to your server, but for that you need the temporary root password that was generated during installation:

``` plain
grep 'password' /var/log/mysqld.log
2017-01-19T08:53:45.638708Z 1 [Note] A temporary password is generated for root@localhost: hoPAejdrk6_a
```

After running <code>mysql_secure_installation</code>, you will be asked to change the root password in accordance with a policy that requires 12 characters, with a mix of uppercase, lowercase, numbers and special characters. Then you will be prompted to answer some questions regarding the removal of anonymous users, test databases, and disallowing remote root login.

To access your MySQL instance via the command line, log in as root and give your password:

``` plain
mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 16
Server version: 5.7.17 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

Let's look at what databases are available:

``` plain
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

I created a new database to play with:

``` plain
mysql> create database vip;
Query OK, 1 row affected (0.00 sec)
```

Now change the current DB to point the one you created:

``` plain
mysql> use vip;
Database changed
```

### Backup and restoration

Before throwing some data into it, let's first see how to back up data and restore it. For this, we have the **mysqldump** utility. To make a backup of my newly created DB, I typed: <code>mysqldump -u root -p vip > backup.sql</code>.

Then I deleted the DB with the <code>drop database vip;</code> command. For some reason though, I could not restore the DB without first creating a DB with the same name. The restore command is <code>mysql -u root -p vip < backup.sql</code>.

### Adding data to DB

The database is all well and good, but we need to put a table in there!

``` sql
create table users (
name VARCHAR(10)
);
```

I made a *users* table with one column for the *name*. Let's confirm this:

``` plain
mysql> show tables;
+---------------+
| Tables_in_vip |
+---------------+
| users         |
+---------------+
1 row in set (0.00 sec)
```

Next, I made some users:

``` plain
mysql> insert into users values ('root');
Query OK, 1 row affected (0.04 sec)

mysql> insert into users values ('guest');
Query OK, 1 row affected (0.00 sec)

mysql> select * from users;
+-------+
| name  |
+-------+
| root  |
| guest |
+-------+
2 rows in set (0.00 sec)
```

Let's see the DB in action by integrating it with some PHP!

## Installing PHP 

Installing PHP on CentOS can be done with the following command: <code>yum install php php-mysql</code>

The next step is to write some PHP code to connect to the DB. But this is not Debian, so nothing just works! There is the pesky SELinux to take into account, and it's not letting Apache to reach the MySQL DB. Use **getsebool** to see the boolean values for the web server:

``` plain
getsebool -a | grep httpd
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_network_connect --> off
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> off
httpd_can_network_memcache --> off
httpd_can_network_relay --> off
httpd_can_sendmail --> off
httpd_dbus_avahi --> off
httpd_dbus_sssd --> off
httpd_dontaudit_search_dirs --> off
httpd_enable_cgi --> on
httpd_enable_ftp_server --> off
httpd_enable_homedirs --> off
httpd_execmem --> off
httpd_graceful_shutdown --> on
httpd_manage_ipa --> off
httpd_mod_auth_ntlm_winbind --> off
httpd_mod_auth_pam --> off
httpd_read_user_content --> off
httpd_run_ipa --> off
httpd_run_preupgrade --> off
httpd_run_stickshift --> off
httpd_serve_cobbler_files --> off
httpd_setrlimit --> off
httpd_ssi_exec --> off
httpd_sys_script_anon_write --> off
httpd_tmp_exec --> off
httpd_tty_comm --> off
httpd_unified --> off
httpd_use_cifs --> off
httpd_use_fusefs --> off
httpd_use_gpg --> off
httpd_use_nfs --> off
httpd_use_openstack --> off
httpd_use_sasl --> off
httpd_verify_dns --> off
```

As you can see, by default Apache is very limited in what it can do. Let's change that and allow it to connect to the DB server, and make the change persist across reboots:

``` plain
setsebool -P httpd_can_network_connect_db 1
```

If you check again, you will see that the <code>httpd_can_network_connect_db</code> boolean is now set to on. I've used the code from the [PHP mysqli_connect() documentation](http://php.net/manual/en/function.mysqli-connect.php) to check the connection status, and finally, it went from failure to success.

``` php

<?php
$link = mysqli_connect("127.0.0.1", "my_user", "my_password", "my_db");

if (!$link) {
    echo "Error: Unable to connect to MySQL." . PHP_EOL;
    echo "Debugging errno: " . mysqli_connect_errno() . PHP_EOL;
    echo "Debugging error: " . mysqli_connect_error() . PHP_EOL;
    exit;
}

echo "Success: A proper connection to MySQL was made! The my_db database is great." . PHP_EOL;
echo "Host information: " . mysqli_get_host_info($link) . PHP_EOL;

mysqli_close($link);
?>
```

Next, I made a really basic, vulnerable page just to see if things are working. I placed the following PHP file in my web server directory:

``` php

<html>


<form action="reallyproform.php" method="post">
   <p>Enter a name in this box to see if you know anyone in the VIP area: <input type="text" name="username" /></p>
   <input type="submit" name="submit" value="Submit" />
</form>

</body>

</html> 

<?php
$link = mysqli_connect("127.0.0.1", "username", "password", "database name");

$username = $_POST['username'];

$query = "SELECT * FROM users WHERE name = '".$username."'";
$result = mysqli_query($link,$query); 

if(mysqli_num_rows($result)>=1)
           {
            echo"Yes you are. Proceed";
           }
else
{
echo "Are you sure you're supposed to be here?";
}

mysqli_close($link);
?>

```

{% img center /images/sysadmin/page.png 'Sample PHP page' 'PHP form' %}

If you enter a valid username, you will just get a message, but this was just an example to show a working connection between the PHP code hosted on the server and the MySQL database.

### phpMyAdmin setup

Chances are, you won't feel an urge to always interact with your DB via the command line. The last step in this post is to install and configure phpMyAdmin in order to do all the SQL operations from a web interface.

First, install it with the following command: 

``` plain
yum install phpmyadmin
```

Restart Apache and go to http://127.0.0.1/phpmyadmin to see your new phpMyAdmin interface.

``` plain
/ You plan things that you do not even \
| attempt because of your extreme      |
\ caution.                             /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```


