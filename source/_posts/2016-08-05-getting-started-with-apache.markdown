---
layout: post
title: "Getting started with Apache"
date: 2016-08-05 07:26:16 -0400
comments: true
categories: [linux, sysadmin]
keywords: apache, apache web server, apache server, apache 2.4, apache on centos, install apache, configure apache, apache linux, apache centos, apache tutorial
description: Installing and configuring Apache 2.4 on CentOS 7
---

In this post I will go over the installation and configuration of an Apache web server on CentOS.

<!-- more -->

# Installation

First, let's verify whether Apache is already installed. Note that on CentOS, the package (and process name) for Apache is called **httpd**.

``` plain
rpm -q httpd
package httpd is not installed
ls -l /var/www
ls: cannot access /var/www: No such file or directory
```

Apache is not installed, so let's install it with <code>yum install httpd</code>. Now verify that it has been installed:

``` plain
rpm -q httpd
httpd-2.4.6-40.el7.centos.4.x86_64
ls -l /var/www
total 0
drwxr-xr-x. 2 root root 6 Jul 18 18:30 cgi-bin
drwxr-xr-x. 2 root root 6 Jul 18 18:30 html
```

Remember that the directory for Apache configuration is <code>/etc/httpd</code>:

``` plain
[root@localhost ~]# ls /etc/httpd
conf  conf.d  conf.modules.d  logs  modules  run
```

And the Apache configuration file is <code>/etc/httpd/conf/httpd.conf</code>. I included a snapshot of a fresh config file at the end of this post.

# Configuration

Now we can start Apache and check that it is running with the **apachectl** command, that provides a front-end control interface for the httpd daemon. For reference, the man page for this utility is at the end of the post.

To start the web server, type <code>apachectl start</code> (alternatively, you can do it with <code>service httpd start</code>, and can use the httpd daemon directly to pass arguments, etc. At the end I've also included the httpd man page.

If you're getting a message like "Could not reliably determine the server's fully qualified domain name", you can modify the ServerName in the configuration file and then restart the server. After making changes to the config file, it's also a good idea to check if there are any errors in it:

``` plain
[root@freehat ~]# apachectl configtest
Syntax OK
```

Now, let's check the status of the server:

``` plain
[root@localhost ~]# apachectl status
* httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2016-08-02 15:53:53 EEST; 1min 42s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 5941 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
[snip]
```

Also, if you want Apache to start on boot, you need to specify it:

``` plain
chkconfig httpd on
Note: Forwarding request to 'systemctl enable httpd.service'.
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
```

If you need to troubleshoot your Apache, you can consult the error log at <code>/var/log/httpd/error_log</code>.

## Site setup

To add an index for your site, create an *index.html* file inside <code>/var/www/html/</code>. This is what you'll see when visiting your site.

``` plain
[root@localhost ~]# echo 'Hello interwebz!' > /var/www/html/index.html
```

When you navigate to your website, this page will be what you'll see. On other distributions, a default index.html may be present in the website root (identified by the DocumentRoot directive).

### Virtual hosts

Virtual hosts allow you to run multiple websites on a single server. Let's see some examples of how we can do that.

#### Port-based vhosts

One option is to **put each website on a different port**. In addition to our main website on port 80, let's serve a different one on port 8080.

First, add a new Listen directive in <code>/etc/httpd/conf/httpd.conf</code>, to tell Apache to also listen on port 8080:

``` plain
Listen 80
Listen 8080
```

We could add virtual hosts in the main configuration file, but if you plan to serve many websites, better to create separate configuration files for each virtual host. Check that the line telling Apache to load other config files is uncommented:

``` plain
# Load config files in the "/etc/httpd/conf.d" directory, if any.
IncludeOptional conf.d/*.conf
```

Now, create a config file for a specific virtual host (it must end in .conf):

``` plain
[root@freehat ~]# nano /etc/httpd/conf.d/skynet.conf
[root@freehat ~]# cat /etc/httpd/conf.d/skynet.conf
<VirtualHost *:8080>
    DocumentRoot "/var/www/html/skynet"
    ServerName skynet.local
</VirtualHost>
```

Next, create the necessary document root directory:

``` plain
[root@freehat ~]# mkdir /var/www/html/skynet
```

Put some resource that you want to be server in the folder:

``` plain
[root@freehat ~]# echo 'You are marked for extermination' >  /var/www/html/skynet/index.html
```

The last step is to tell Apache to reload its configuration files with <code>service httpd reload</code>. However, now I ran into some SELinux alert. I looked in <code>/var/log/messages</code>:

``` plain
Aug  3 16:16:16 localhost setroubleshoot: SELinux is preventing /usr/sbin/httpd from name_connect access on the tcp_socket port 8080. For complete SELinux messages. run sealert -l bc571bd3-344f-4a4e-830a-ce744154d527
```

I ran the mentioned command but was confused by my first brush with SELinux, so I did some google-fu which led me to [this post](https://www.certdepot.net/rhel7-use-selinux-port-labelling/). You can verify what ports are allowed by SELinux for HTTP traffic with the below command:

``` plain
[root@freehat ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
```

To add port 8080 to http_port_t, I ran <code>semanage port -m -t http_port_t -p tcp 8080</code>, and then I checked it was added. Should be all good now. If you go visit your server on port 8080, you should get a friendly message. To differentiate it from the website on port 80, create another virtual host configuration for that respective server.

You can now access your sites from your own machine, but if you try navigating to them from another computer, it will be just like the websites don't exist! It turns out, iptables is blocking access, and you have to tell it to open the ports:

``` plain
[root@freehat sysconfig]# iptables -I INPUT -p tcp --dport 80 -j ACCEPT
[root@freehat sysconfig]# iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
```

And if you want these changes to survive a reboot, you should save them. I had to do some additional work for that. It turns out, [RHEL 7 and CentOS 7 use *firewalld*](https://stackoverflow.com/questions/24756240/how-can-i-use-iptables-on-centos-7) to manage iptables. So first I followed the instructions on StackOverflow to stop and mask the firewalld service:

``` plain
systemctl stop firewalld
[root@freehat ~]# systemctl mask firewalld
Created symlink from /etc/systemd/system/firewalld.service to /dev/null.
```

Then I had to install *iptables-services*: <code>yum install iptables-services</code>. Next I enabled it at boot:

``` plain
systemctl enable iptables
Created symlink from /etc/systemd/system/basic.target.wants/iptables.service to /usr/lib/systemd/system/iptables.service.
```

I also modified the "no" defaults in <code>/etc/sysconfig/iptables-config</code> to "yes":

``` plain
# Save current firewall rules on stop.
#   Value: yes|no,  default: no
# Saves all firewall rules to /etc/sysconfig/iptables if firewall gets stopped
# (e.g. on system shutdown).
IPTABLES_SAVE_ON_STOP="yes"

# Save current firewall rules on restart.
#   Value: yes|no,  default: no
# Saves all firewall rules to /etc/sysconfig/iptables if firewall gets
# restarted.
IPTABLES_SAVE_ON_RESTART="yes"
```

Finally, I was able to save the changes and checked that they persisted after reboot:

``` plain
service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
```

#### Name-based vhosts

We've seen the virtual hosts running on different ports, now let's see how we can **serve them on the same port, but with different names**. I modified one of the config files so that both sites will run on port 80.Let's see the current virtual host settings:

``` plain
httpd -S
VirtualHost configuration:
*:80                   is a NameVirtualHost
         default server myweb.local (/etc/httpd/conf.d/myweb.conf:1)
         port 80 namevhost myweb.local (/etc/httpd/conf.d/myweb.conf:1)
         port 80 namevhost skynet.local (/etc/httpd/conf.d/skynet.conf:1)
ServerRoot: "/etc/httpd"
Main DocumentRoot: "/var/www/html"
Main ErrorLog: "/etc/httpd/logs/error_log"
[snip]
```

We will use the server names to navigate to those sites. Because I am testing locally with no DNS, I modified <code>/etc/hosts</code> and added these entries:

``` plain
192.168.80.153 myweb.local
192.168.80.153 skynet.local
```

Now I was able to use the names of the sites in the web browser.

### Password protection

It's time to restrict access to the skynet website! There are a number of ways that Apache can protect a resource with a password and ensure that is accessible only to certain users. In this example, I will want the skynet website to allow only a user called tx. First, I used the *htpasswd* utility to create a *.htpasswd* file (you can call it whatever you want), and placed it in <code>/etc/httpd</code>. 

``` plain
htpasswd -c /etc/httpd/.htpasswd tx
New password: 
Re-type new password: 
Adding password for user tx
```

This file holds the allowed user/password combination:

``` plain
cat /etc/httpd/.htpasswd
tx:$apr1$2FI77JZQ$IrMFnxvHzvtRhWGiQfvxL0
```

The next step is to tell Apache to restrict access to a resource based on this file. If you have access to the main configuration file or the virtual host config file, the preferred way is to add a Directory directive there:

``` plain
nano /etc/httpd/conf.d/skynet.conf
<VirtualHost *:80>
    DocumentRoot "/var/www/html/skynet"
    ServerName skynet.local
    <Directory "/var/www/html/skynet">
        AuthType Basic
        AuthName "Authorized Only"
        AuthUserFile /etc/httpd/.htpasswd
        Require valid-user
    </Directory>
</VirtualHost>

```

The Directory specified is the one that will be password-protected. Next, we specify the type of authentication (basic in this case, so no super sensitive files should be stored there), the name which will be displayed on the prompt, the location where Apache can find the password file, and the allowed user(s). Maybe we'll want to add some terminators later, so instead of just specifying one user, I allowed any valid user/password combination from the password file. 

Now if you try to go to the skynet website, you will see the window prompting for username and password.

Another way you can restrict directory access is via a **.htaccess** file. Due to performance and security reasons, you should avoid this, if possible. 

First, create a .htaccess file in the directory you want to protect:

``` plain
nano /var/www/html/skynet/.htaccess
AuthType Basic
AuthName "Authorized Only"
AuthUserFile /etc/httpd/.htpasswd
Require valid-user
```

For the .htaccess file to work, you need to edit the <code>/etc/httpd/conf/httpd.conf</code> file, and change the [AllowOverride directive](https://httpd.apache.org/docs/current/mod/core.html#allowoverride) inside Directory. When set to None, as it is by default, .htaccess files are ignored. Change it to AuthConfig (I had to do it both under /var/www and /var/www/html).

### HTTPS with self-signed certificate

In the next demo, I will encrypt the skynet website with SSL (removed all the authorization bits for demo purposes).

First, I installed [mod_ssl](https://httpd.apache.org/docs/current/mod/mod_ssl.html) with <code>yum install mod_ssl</code>. Next, I created a directory for Apache to store its server key and certificate: <code>mkdir /etc/httpd/myssl</code>

Inside this directory, I created a RSA private key of length 2048:

``` plain
[root@freehat myssl]# openssl genrsa -out https.key 2048
Generating RSA private key, 2048 bit long modulus
....................+++
..............+++
e is 65537 (0x10001)
```

Then I created a Certificate Signing Request (CSR):

``` plain
[root@freehat myssl]# openssl req -new -key https.key -out server.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

Lastly, I generated an x509 certificate with a duration of an year:

``` plain
[root@freehat myssl]# openssl x509 -req -days 365 -in server.csr -signkey https.key -out mycert.crt
Signature ok
subject=/C=XX/L=Default City/O=Default Company Ltd
Getting Private key
```

We're not done yet. Edit the file <code>/etc/httpd/conf.d/ssl.conf</code>, and comment the lines with the location of the server key and certificate, replacing them with new ones pointing at our previously created files:

``` plain
#SSLCertificateFile /etc/pki/tls/certs/localhost.crt
SSLCertificateFile /etc/httpd/myssl/mycert.crt
#SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
SSLCertificateKeyFile /etc/httpd/myssl/https.key
```

Now restart the server and go to website (mine is at https://192.168.80.153/ ). Your browser will warn you about the untrusted connection, add it to exceptions and now you have https on your site.

### Logging

Lastly, let's go over Apache's logging system. There are 2 types of logs kept, error logs for when some error happens, and access logs, that provide information about incoming requests. By default, these logs are located at <code>/var/log/httpd/error_log</code> and <code>/var/log/httpd/access_log</code> on CentOS

For your sites, you can configure the logs and choose where to store them. I decided to separate the logs for my skynet website. I made a folder for them in /var/log/httpd/skynet-logs, and modified the skynet virtual host config file and added the path and format for the logs:

``` plain
CustomLog /var/log/httpd/skynet-logs/skynet-access.log combined
ErrorLog /var/log/httpd/skynet-logs/skynet-error.log
```

I wanted the combined format instead of the common one, because it provides a bit more information, including also the User-agent and the Referer

Hopefully, this post has helped others in setting up an Apache web server. Since I've installed it on CentOS 7, I also had to account for SELinux and some other differences from a Debian distro, so it's been a good learning experience.

If you want to become an Apache wizard, the [Apache 2.4 documentation](https://httpd.apache.org/docs/2.4/) should be a good starting point.

``` plain
________________________________________
/ Give thought to your reputation.       \
| Consider changing name and moving to a |
\ new town.                              /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

**Freshly installed httpd.conf file**

``` plain
#
# This is the main Apache HTTP server configuration file.  It contains the
# configuration directives that give the server its instructions.
# See <URL:http://httpd.apache.org/docs/2.4/> for detailed information.
# In particular, see 
# <URL:http://httpd.apache.org/docs/2.4/mod/directives.html>
# for a discussion of each configuration directive.
#
# Do NOT simply read the instructions in here without understanding
# what they do.  They're here only as hints or reminders.  If you are unsure
# consult the online docs. You have been warned.  
#
# Configuration and logfile names: If the filenames you specify for many
# of the server's control files begin with "/" (or "drive:/" for Win32), the
# server will use that explicit path.  If the filenames do *not* begin
# with "/", the value of ServerRoot is prepended -- so 'log/access_log'
# with ServerRoot set to '/www' will be interpreted by the
# server as '/www/log/access_log', where as '/log/access_log' will be
# interpreted as '/log/access_log'.

#
# ServerRoot: The top of the directory tree under which the server's
# configuration, error, and log files are kept.
#
# Do not add a slash at the end of the directory path.  If you point
# ServerRoot at a non-local disk, be sure to specify a local disk on the
# Mutex directive, if file-based mutexes are used.  If you wish to share the
# same ServerRoot for multiple httpd daemons, you will need to change at
# least PidFile.
#
ServerRoot "/etc/httpd"

#
# Listen: Allows you to bind Apache to specific IP addresses and/or
# ports, instead of the default. See also the <VirtualHost>
# directive.
#
# Change this to Listen on specific IP addresses as shown below to 
# prevent Apache from glomming onto all bound IP addresses.
#
#Listen 12.34.56.78:80
Listen 80

#
# Dynamic Shared Object (DSO) Support
#
# To be able to use the functionality of a module which was built as a DSO you
# have to place corresponding `LoadModule' lines at this location so the
# directives contained in it are actually available _before_ they are used.
# Statically compiled modules (those listed by `httpd -l') do not need
# to be loaded here.
#
# Example:
# LoadModule foo_module modules/mod_foo.so
#
Include conf.modules.d/*.conf

#
# If you wish httpd to run as a different user or group, you must run
# httpd as root initially and it will switch.  
#
# User/Group: The name (or #number) of the user/group to run httpd as.
# It is usually good practice to create a dedicated user and group for
# running httpd, as with most system services.
#
User apache
Group apache

# 'Main' server configuration
#
# The directives in this section set up the values used by the 'main'
# server, which responds to any requests that aren't handled by a
# <VirtualHost> definition.  These values also provide defaults for
# any <VirtualHost> containers you may define later in the file.
#
# All of these directives may appear inside <VirtualHost> containers,
# in which case these default settings will be overridden for the
# virtual host being defined.
#

#
# ServerAdmin: Your address, where problems with the server should be
# e-mailed.  This address appears on some server-generated pages, such
# as error documents.  e.g. admin@your-domain.com
#
ServerAdmin root@localhost

#
# ServerName gives the name and port that the server uses to identify itself.
# This can often be determined automatically, but we recommend you specify
# it explicitly to prevent problems during startup.
#
# If your host doesn't have a registered DNS name, enter its IP address here.
#
#ServerName www.example.com:80

#
# Deny access to the entirety of your server's filesystem. You must
# explicitly permit access to web content directories in other 
# <Directory> blocks below.
#
<Directory />
    AllowOverride none
    Require all denied
</Directory>

#
# Note that from this point forward you must specifically allow
# particular features to be enabled - so if something's not working as
# you might expect, make sure that you have specifically enabled it
# below.
#

#
# DocumentRoot: The directory out of which you will serve your
# documents. By default, all requests are taken from this directory, but
# symbolic links and aliases may be used to point to other locations.
#
DocumentRoot "/var/www/html"

#
# Relax access to content within /var/www.
#
<Directory "/var/www">
    AllowOverride None
    # Allow open access:
    Require all granted
</Directory>

# Further relax access to the default document root:
<Directory "/var/www/html">
    #
    # Possible values for the Options directive are "None", "All",
    # or any combination of:
    #   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
    #
    # Note that "MultiViews" must be named *explicitly* --- "Options All"
    # doesn't give it to you.
    #
    # The Options directive is both complicated and important.  Please see
    # http://httpd.apache.org/docs/2.4/mod/core.html#options
    # for more information.
    #
    Options Indexes FollowSymLinks

    #
    # AllowOverride controls what directives may be placed in .htaccess files.
    # It can be "All", "None", or any combination of the keywords:
    #   Options FileInfo AuthConfig Limit
    #
    AllowOverride None

    #
    # Controls who can get stuff from this server.
    #
    Require all granted
</Directory>

#
# DirectoryIndex: sets the file that Apache will serve if a directory
# is requested.
#
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>

#
# The following lines prevent .htaccess and .htpasswd files from being 
# viewed by Web clients. 
#
<Files ".ht*">
    Require all denied
</Files>

#
# ErrorLog: The location of the error log file.
# If you do not specify an ErrorLog directive within a <VirtualHost>
# container, error messages relating to that virtual host will be
# logged here.  If you *do* define an error logfile for a <VirtualHost>
# container, that host's errors will be logged there and not here.
#
ErrorLog "logs/error_log"

#
# LogLevel: Control the number of messages logged to the error_log.
# Possible values include: debug, info, notice, warn, error, crit,
# alert, emerg.
#
LogLevel warn

<IfModule log_config_module>
    #
    # The following directives define some format nicknames for use with
    # a CustomLog directive (see below).
    #
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common

    <IfModule logio_module>
      # You need to enable mod_logio.c to use %I and %O
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>

    #
    # The location and format of the access logfile (Common Logfile Format).
    # If you do not define any access logfiles within a <VirtualHost>
    # container, they will be logged here.  Contrariwise, if you *do*
    # define per-<VirtualHost> access logfiles, transactions will be
    # logged therein and *not* in this file.
    #
    #CustomLog "logs/access_log" common

    #
    # If you prefer a logfile with access, agent, and referer information
    # (Combined Logfile Format) you can use the following directive.
    #
    CustomLog "logs/access_log" combined
</IfModule>

<IfModule alias_module>
    #
    # Redirect: Allows you to tell clients about documents that used to 
    # exist in your server's namespace, but do not anymore. The client 
    # will make a new request for the document at its new location.
    # Example:
    # Redirect permanent /foo http://www.example.com/bar

    #
    # Alias: Maps web paths into filesystem paths and is used to
    # access content that does not live under the DocumentRoot.
    # Example:
    # Alias /webpath /full/filesystem/path
    #
    # If you include a trailing / on /webpath then the server will
    # require it to be present in the URL.  You will also likely
    # need to provide a <Directory> section to allow access to
    # the filesystem path.

    #
    # ScriptAlias: This controls which directories contain server scripts. 
    # ScriptAliases are essentially the same as Aliases, except that
    # documents in the target directory are treated as applications and
    # run by the server when requested rather than as documents sent to the
    # client.  The same rules about trailing "/" apply to ScriptAlias
    # directives as to Alias.
    #
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"

</IfModule>

#
# "/var/www/cgi-bin" should be changed to whatever your ScriptAliased
# CGI directory exists, if you have that configured.
#
<Directory "/var/www/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>

<IfModule mime_module>
    #
    # TypesConfig points to the file containing the list of mappings from
    # filename extension to MIME-type.
    #
    TypesConfig /etc/mime.types

    #
    # AddType allows you to add to or override the MIME configuration
    # file specified in TypesConfig for specific file types.
    #
    #AddType application/x-gzip .tgz
    #
    # AddEncoding allows you to have certain browsers uncompress
    # information on the fly. Note: Not all browsers support this.
    #
    #AddEncoding x-compress .Z
    #AddEncoding x-gzip .gz .tgz
    #
    # If the AddEncoding directives above are commented-out, then you
    # probably should define those extensions to indicate media types:
    #
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz

    #
    # AddHandler allows you to map certain file extensions to "handlers":
    # actions unrelated to filetype. These can be either built into the server
    # or added with the Action directive (see below)
    #
    # To use CGI scripts outside of ScriptAliased directories:
    # (You will also need to add "ExecCGI" to the "Options" directive.)
    #
    #AddHandler cgi-script .cgi

    # For type maps (negotiated resources):
    #AddHandler type-map var

    #
    # Filters allow you to process content before it is sent to the client.
    #
    # To parse .shtml files for server-side includes (SSI):
    # (You will also need to add "Includes" to the "Options" directive.)
    #
    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
</IfModule>

#
# Specify a default charset for all content served; this enables
# interpretation of all content as UTF-8 by default.  To use the 
# default browser choice (ISO-8859-1), or to allow the META tags
# in HTML content to override this choice, comment out this
# directive:
#
AddDefaultCharset UTF-8

<IfModule mime_magic_module>
    #
    # The mod_mime_magic module allows the server to use various hints from the
    # contents of the file itself to determine its type.  The MIMEMagicFile
    # directive tells the module where the hint definitions are located.
    #
    MIMEMagicFile conf/magic
</IfModule>

#
# Customizable error responses come in three flavors:
# 1) plain text 2) local redirects 3) external redirects
#
# Some examples:
#ErrorDocument 500 "The server made a boo boo."
#ErrorDocument 404 /missing.html
#ErrorDocument 404 "/cgi-bin/missing_handler.pl"
#ErrorDocument 402 http://www.example.com/subscription_info.html
#

#
# EnableMMAP and EnableSendfile: On systems that support it, 
# memory-mapping or the sendfile syscall may be used to deliver
# files.  This usually improves server performance, but must
# be turned off when serving from networked-mounted 
# filesystems or if support for these functions is otherwise
# broken on your system.
# Defaults if commented: EnableMMAP On, EnableSendfile Off
#
#EnableMMAP off
EnableSendfile on

# Supplemental configuration
#
# Load config files in the "/etc/httpd/conf.d" directory, if any.
IncludeOptional conf.d/*.conf
```

**apachectl utility**

``` plain
APACHECTL(8)			     apachectl			      APACHECTL(8)



NAME
       apachectl - Apache HTTP Server Control Interface


SYNOPSIS
       When  acting  in	 pass-through  mode,  apachectl can take all the arguments
       available for the httpd binary.


       apachectl [ httpd-argument ]


       When acting in SysV init mode, apachectl takes simple,  one-word	 commands,
       defined below.


       apachectl command



SUMMARY
       apachectl  is  a front end to the Apache HyperText Transfer Protocol (HTTP)
       server. It is designed to help the administrator control the functioning of
       the Apache httpd daemon.


       The  apachectl script can operate in two modes. First, it can act as a sim‐
       ple front-end to the httpd command that simply sets any necessary  environ‐
       ment  variables	and  then  invokes httpd, passing through any command line
       arguments. Second, apachectl can act as a SysV init script,  taking  simple
       one-word arguments like start, restart, and stop, and translating them into
       appropriate signals to httpd.


       If your Apache installation uses non-standard paths, you will need to  edit
       the  apachectl script to set the appropriate paths to the httpd binary. You
       can also specify any necessary httpd command line arguments. See	 the  com‐
       ments in the script for details.


       The  apachectl script returns a 0 exit value on success, and >0 if an error
       occurs. For more details, view the comments in the script.



OPTIONS
       Only the SysV init-style options are  defined  here.  Other  arguments  are
       defined on the httpd manual page.



       start  Start  the Apache httpd daemon. Gives an error if it is already run‐
	      ning. This is equivalent to apachectl -k start.

       stop   Stops the Apache httpd daemon. This is equivalent	 to  apachectl	-k
	      stop.

       restart
	      Restarts	the  Apache httpd daemon. If the daemon is not running, it
	      is started. This	command	 automatically	checks	the  configuration
	      files  as	 in  configtest before initiating the restart to make sure
	      the daemon doesn't die. This is equivalent to apachectl -k restart.

       fullstatus
	      Displays a full status report from mod_status. For this to work, you
	      need  to	have  mod_status  enabled  on your server and a text-based
	      browser such as lynx available on	 your  system.	The  URL  used	to
	      access  the  status report can be set by editing the STATUSURL vari‐
	      able in the script.

       status Displays a brief status report using systemd.

       graceful
	      Gracefully restarts the Apache httpd daemon. If the  daemon  is  not
	      running,	it  is	not started. This differs from a normal restart in
	      that currently open connections are not aborted. A  side	effect	is
	      that  old	 log files will not be closed immediately. This means that
	      if used in a log rotation script, a substantial delay may be  neces‐
	      sary  to	ensure that the old log files are closed before processing
	      them. This command automatically checks the configuration	 files	as
	      in  configtest  before  initiating  the  restart to make sure Apache
	      doesn't die. This is equivalent to apachectl -k graceful.

       graceful-stop
	      Gracefully stops the Apache httpd daemon. This differs from a normal
	      stop  in	that  currently	 open  connections are not aborted. A side
	      effect is that old log files will not be closed immediately. This is
	      equivalent to apachectl -k graceful-stop.

       configtest
	      Run  a  configuration  file syntax test. It parses the configuration
	      files and either reports Syntax Ok or detailed information about the
	      particular syntax error. This is equivalent to apachectl -t.


       The  following  option  was  available  in  earlier  versions  but has been
       removed.



       startssl
	      To start httpd with SSL support, you should edit your  configuration
	      file  to	include	 the  relevant	directives and then use the normal
	      apachectl start.




Apache HTTP Server		    2005-08-26			      APACHECTL(8)
```

**httpd reference**

``` plain
HTTPD(8)			       httpd				  HTTPD(8)



NAME
       httpd - Apache Hypertext Transfer Protocol Server


SYNOPSIS
       httpd  [	 -d serverroot ] [ -f config ] [ -C directive ] [ -c directive ] [
       -D parameter ] [	 -e  level  ]  [  -E  file  ]  [  -k  start|restart|grace‐
       ful|stop|graceful-stop  ] [ -R directory ] [ -h ] [ -l ] [ -L ] [ -S ] [ -t
       ] [ -v ] [ -V ] [ -X ] [ -M ] [ -T ]


       On Windows systems, the following additional arguments are available:


       httpd [ -k install|config|uninstall ] [ -n name ] [ -w ]



SUMMARY
       httpd is the Apache HyperText Transfer Protocol (HTTP) server  program.	It
       is  designed  to be run as a standalone daemon process. When used like this
       it will create a pool of child processes or threads to handle requests.


       In general, httpd should not be invoked	directly,  but	rather	should	be
       invoked	via apachectl on Unix-based systems or as a service on Windows NT,
       2000 and XP and as a console application on Windows 9x and ME.



OPTIONS
       -d serverroot
	      Set the initial value for the ServerRoot	directive  to  serverroot.
	      This can be overridden by the ServerRoot directive in the configura‐
	      tion file. The default is /etc/httpd.

       -f config
	      Uses the directives in the file config on startup.  If  config  does
	      not  begin  with	a /, then it is taken to be a path relative to the
	      ServerRoot. The default is conf/httpd.conf.

       -k start|restart|graceful|stop|graceful-stop
	      Signals httpd to start, restart, or stop. See Stopping Apache  httpd
	      for more information.

       -C directive
	      Process the configuration directive before reading config files.

       -c directive
	      Process the configuration directive after reading config files.

       -D parameter
	      Sets  a  configuration  parameter	 which can be used with <IfDefine>
	      sections in the configuration files to conditionally skip or process
	      commands at server startup and restart. Also can be used to set cer‐
	      tain less-common startup parameters including  -DNO_DETACH  (prevent
	      the  parent  from forking) and -DFOREGROUND (prevent the parent from
	      calling setsid() et al).

       -e level
	      Sets the LogLevel to level during server startup. This is useful for
	      temporarily  increasing  the verbosity of the error messages to find
	      problems during startup.

       -E file
	      Send error messages during server startup to file.

       -h     Output a short summary of available command line options.

       -l     Output a list of modules compiled into the  server.  This	 will  not
	      list dynamically loaded modules included using the LoadModule direc‐
	      tive.

       -L     Output a list of directives provided  by	static	modules,  together
	      with  expected  arguments	 and  places where the directive is valid.
	      Directives provided by shared modules are not listed.

       -M     Dump a list of loaded Static and Shared Modules.

       -S     Show the settings as parsed from the  config  file  (currently  only
	      shows the virtualhost settings).

       -T (Available in 2.3.8 and later)
	      Skip document root check at startup/restart.

       -t     Run  syntax  tests for configuration files only. The program immedi‐
	      ately exits after these syntax parsing tests with	 either	 a  return
	      code  of 0 (Syntax OK) or return code not equal to 0 (Syntax Error).
	      If -D DUMP_VHOSTS is also set, details of the virtual host  configu‐
	      ration  will be printed. If -D DUMP_MODULES  is set, all loaded mod‐
	      ules will be printed.

       -v     Print the version of httpd, and then exit.

       -V     Print the version and build parameters of httpd, and then exit.

       -X     Run httpd in debug mode. Only one worker will  be	 started  and  the
	      server will not detach from the console.


       The following arguments are available only on the Windows platform:



       -k install|config|uninstall
	      Install Apache httpd as a Windows NT service; change startup options
	      for the Apache httpd service; and uninstall the  Apache  httpd  ser‐
	      vice.

       -n name
	      The name of the Apache httpd service to signal.

       -w     Keep  the console window open on error so that the error message can
	      be read.




Apache HTTP Server		    2012-02-10				  HTTPD(8)
```

