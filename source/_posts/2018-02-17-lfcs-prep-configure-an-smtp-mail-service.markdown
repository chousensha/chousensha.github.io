---
layout: post
title: "LFCS prep - Configure an SMTP mail service"
date: 2018-02-17 12:16:33 -0500
comments: true
categories: [linux, sysadmin]
keywords: smtp, mail, postfix, smtp linux, linux mail, mail aliases, lfcs, rhcsa, lfcsa
description: Configure SMTP server, restrict access to it and use mail aliases
---

Today we'll look at configuring a mail service for LFCS objectives. We'll use the Postfix implementation that already comes preinstalled  on CentOS.

<!-- more -->

The configuration file with all Postfix parameters is <code>/etc/postfix/main.cf</code> and it is massive. Instead of manually editing it, it's easier to use **postconf**. Without arguments, it displays all parameters. If given a parameter name, it will show that parameter's value, and with the **-e** flag it can modify parameters.

Another important file for mail configuration is <code>/etc/aliases</code>. If you want a mail destined to a user to be delivered to another user, you can set an alias:

``` 
username: alias 
```

The alias will get the mails destined for the user. I've added an alias on my system so that root's mail will be delivered to nixhat:

``` 
root: nixhat
```

After changing any alias settings, you have to refresh the alias DB with the command <code>postalias /etc/aliases</code>

Now it's time to edit some parameters in order to setup a working mail configuration. 

* The **myorigin** parameter specifies the domain that locally-posted mail appears to come from 

The default is to use the hostname for this, but I will change it to use just the domain (example.com in this case)

``` 
postconf -e 'myorigin=$mydomain'
[root@centos ~]# postconf myorigin
myorigin = $mydomain
```

* The **inet_interfaces** parameter specifies the network interface addresses that this mail system receives mail on

Here I have mine listening on the Ethernet interface in addition to localhost:

``` 
postconf inet_interfaces
inet_interfaces = 192.168.217.131, localhost
```

* The **relayhost** parameter specifies the default host to send mail to when no entry is matched in the optional transport(5) table. When no relayhost is given, mail is routed directly to the destination. On an intranet, specify the organizational domain name. In the case of SMTP, specify a domain, host, host:port, [host]:port, [address] or [address]:port; the form [host] turns off MX lookups.

If you want to forward mail to a central host, use the **relayhost** parameter

* The **mydestination** parameter specifies the list of domains that this machine considers itself the final destination for. This parameter is important for receiving messages 

``` 
postconf mydestination
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
```

* The **mynetworks** parameter specifies the list of "trusted" SMTP clients that are allowed to relay mail through Postfix. It can allow clients on the local subnet, in a range, just a host, or manually specified.

``` 
postconf mynetworks
mynetworks = 192.168.217.0/24 127.0.0.0/8 [::1]/128
```

## Restricting access to the server 

For receiving mail, you can add some settings to harden the security of your server:

* **smtpd_helo_required** - Require that a remote SMTP client sends HELO or EHLO before commencing a MAIL transaction

``` 
postconf smtpd_helo_required
smtpd_helo_required = yes
```

* **smtpd_helo_restrictions** - other optional restrictions: 

**permit_mynetworks** - Permit the request when the client IP address matches any network or network address listed in $mynetworks

 **reject_invalid_helo_hostname** - Reject the request when the HELO or EHLO hostname is malformed

``` 
postconf smtpd_helo_restrictions
smtpd_helo_restrictions = permit_mynetworks, reject_invalid_helo_hostname
```

* **smtpd_sender_restrictions** - Optional restrictions that the Postfix SMTP server applies in the context of a client MAIL FROM command:

**reject_unknown_sender_domain** - Reject the request when Postfix is not final destination for the sender address, and the MAIL FROM domain has 1) no DNS MX and no DNS A record, or 2) a malformed MX record such as a record with a zero-length MX hostname

``` 
postconf smtpd_sender_restrictions
smtpd_sender_restrictions = permit_mynetworks, reject_unknown_sender_domain
```

* **smtpd_recipient_restrictions** - Optional restrictions that the Postfix SMTP server applies in the context of a client RCPT TO command

**reject_unauth_destination** - reject the request unless 1) Postfix is the mail forwarder; or 2) Postfix is the final destination 

``` 
postconf smtpd_recipient_restrictions
smtpd_recipient_restrictions = permit_mynetworks, reject_unauth_destination
```

After changing Postfix settings, it is a good idea to run <code>postfix check</code> to check the validity of the config file. Then restart Postfix. 

Now to test it:

``` 
echo "This is a test" | mail -s "test" root@example.com
```

On user nixhat:

``` 
mail
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/nixhat": 1 message 1 new
>N  1 root                  Sat Feb 17 18:08  18/569   "test"
cat  /var/spool/mail/nixhat
From root@example.com  Sat Feb 17 18:08:09 2018
Return-Path: <root@example.com>
X-Original-To: root@example.com
Delivered-To: root@example.com
Received: by centos.example.com (Postfix, from userid 0)
	id 51255218FED7; Sat, 17 Feb 2018 18:08:09 +0200 (EET)
Date: Sat, 17 Feb 2018 18:08:09 +0200
To: root@example.com
Subject: test
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Message-Id: <20180217160809.51255218FED7@centos.example.com>
From: root@example.com (root)
Status: O

This is a test
```

So it works. If you want to see more about the status of your mails, look inside <code>/var/log/maillog</code>:

``` 
Feb 17 18:21:11 centos postfix/pickup[8896]: 4FADF218FED8: uid=0 from=<root>
Feb 17 18:21:11 centos postfix/cleanup[9359]: 4FADF218FED8: message-id=<20180217162111.4FADF218FED8@centos.example.com>
Feb 17 18:21:11 centos postfix/qmgr[8897]: 4FADF218FED8: from=<root@example.com>, size=431, nrcpt=1 (queue active)
Feb 17 18:21:11 centos postfix/local[9361]: 4FADF218FED8: to=<nixhat@example.com>, orig_to=<root@example.com>, relay=local, delay=0.08, delays=0.07/0.02/0/0, dsn=2.0.0, status=sent (delivered to mailbox)
Feb 17 18:21:11 centos postfix/qmgr[8897]: 4FADF218FED8: removed
```

Some other useful actions with mail would be to view the mail queue with **mailq** and flush the queue with **postfix flush**

Also see **man 5 postconf** for all Postfix parameters

``` 
 _________________________________________
/ Q: What lies on the bottom of the ocean \
\ and twitches? A: A nervous wreck.       /
 -----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
