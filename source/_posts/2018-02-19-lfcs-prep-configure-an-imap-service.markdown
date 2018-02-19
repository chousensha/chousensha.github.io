---
layout: post
title: "LFCS prep - Configure an IMAP service"
date: 2018-02-19 12:49:18 -0500
comments: true
categories: [linux, sysadmin]
keywords: imap, imaps, mail, dovecot, imap linux, linux mail, dovecot mail, lfcs, rhcsa, lfcsa
description: Configure IMAP mail
---

Today we continue with the IMAP/S of LFCS, by configuring Dovecot.

[Dovecot](https://www.dovecot.org/) is an open source IMAP / POP3 server written with security in mind. It can be installed on CentOS via the *dovecot* package.

<!-- more -->

Inside the <code>/etc/dovecot.conf</code> file you have to add / edit some settings:

``` 
protocols = imap pop3
```

The above also include the secure versions of IMAPS and POP3 secure. 

Next you have to edit the mailbox and namespace file <code>/etc/dovecot/conf.d/10-mail.conf</code>:

``` 
mail_location = mbox:~/mail:INBOX=/var/mail/%u
mail_privileged_group = mail
```

These will set the mailbox location to */var/mail/username* and will set the group to *mail* for privileged operations.

Inside <code>/etc/dovecot/conf.d/10-ssl.conf</code> ensure that the below lines are uncommented:

``` 
ssl_cert = </etc/pki/dovecot/certs/dovecot.pem
ssl_key = </etc/pki/dovecot/private/dovecot.pem
```

Dump the non-default settings if you want an overview of Dovecot's settings:

``` 
dovecot -n
# 2.2.10: /etc/dovecot/dovecot.conf
# OS: Linux 3.10.0-514.21.1.el7.x86_64 x86_64 CentOS Linux release 7.3.1611 (Core)  
first_valid_uid = 1000
mail_location = mbox:~/mail:INBOX=/var/mail/%u
mail_privileged_group = mail
mbox_write_locks = fcntl
namespace inbox {
  inbox = yes
  location = 
  mailbox Drafts {
    special_use = \Drafts
  }
  mailbox Junk {
    special_use = \Junk
  }
  mailbox Sent {
    special_use = \Sent
  }
  mailbox "Sent Messages" {
    special_use = \Sent
  }
  mailbox Trash {
    special_use = \Trash
  }
  prefix = 
}
passdb {
  driver = pam
}
protocols = imap pop3
ssl = required
ssl_cert = </etc/pki/dovecot/certs/dovecot.pem
ssl_key = </etc/pki/dovecot/private/dovecot.pem
userdb {
  driver = passwd
}
```

Start dovecot and check that it's listening on its ports:

``` 
netstat -ltpn | grep dovecot
tcp        0      0 0.0.0.0:110             0.0.0.0:*               LISTEN      4629/dovecot        
tcp        0      0 0.0.0.0:143             0.0.0.0:*               LISTEN      4629/dovecot        
tcp        0      0 0.0.0.0:993             0.0.0.0:*               LISTEN      4629/dovecot        
tcp        0      0 0.0.0.0:995             0.0.0.0:*               LISTEN      4629/dovecot        
```

Send a mail to a user:

``` 
echo "Readme" | mail -s "README" nixhat@example.com
```

The mail will be readable in */var/mail/nixhat*. I installed the mutt client to view it:

``` 
i:Exit  -:PrevPg  <Space>:NextPg v:View Attachm.  d:Del  r:Reply  j:Next ?:Help
Date: Mon, 19 Feb 2018 19:35:30 +0200
From: root <root@example.com>
To: nixhat@example.com
Subject: README
User-Agent: Heirloom mailx 12.5 7/5/10

Readme
```

``` 
 ________________________________________
/ Avert misunderstanding by calm, poise, \
\ and balance.                           /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


