---
layout: post
title: "LFCS prep - Securing SSH"
date: 2018-01-05 16:30:55 -0500
comments: true
categories: [linux, sysadmin]
keywords: ssh, ssh key, ssh root, lfcs, lfcsa, rhcsa, lfcs exam, rhcsa exam
description: Key based authentication and no root logins on SSH
---

SSH-ing between various systems is something you will definitely be doing a lot in infosec. So today I'll quickly go over an exam objective of configuring SSH for key based authentication and disabling root logins for extra security.

<!-- more -->

First of all, let's confirm that I can SSH to my machine as root and login with root's password:

``` 
ssh root@192.168.217.129
The authenticity of host '192.168.217.129 (192.168.217.129)' can't be established.
ECDSA key fingerprint is SHA256:20TtBMwQAz9KAEK2mApqOksugeZ2yi8oecLRKz8RwB4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.217.129' (ECDSA) to the list of known hosts.
root@192.168.217.129's password: 
Last login: Fri Jan  5 14:04:25 2018
[root@CentOS7 ~]# 
```

The CentOS server's public key has been added to <code>~/.ssh/known_hosts</code> on my client machine, and I was able to login as root with no problem. Now it's time to change these dangerous defaults. On the client, generate a new pair of keys:

``` 
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:OLHfszGKJK7wPNhc6OLZv8NeMKOIDuV0OHmBeynLLVI root@kali
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|   .             |
|  . . .          |
|   + o +         |
|  E.=++ S        |
|.*oOo +o .       |
|=*=o+ ... =      |
|==Bo =.. . =     |
|.=+++++ . .      |
+----[SHA256]-----+
```

Now you have a private and public key inside the .ssh directory. Take a look at their permissions:

``` 
-rw-------  1 root root 1675 Jan  5 16:10 id_rsa
-rw-r--r--  1 root root  391 Jan  5 16:10 id_rsa.pub
```

The private key has read and write permissions only for the owner (600), while the public key has read/write permissions for the owner, and read also for group and others (644).

The public key won't do any good sitting around on the client machine, it needs to be copied to the server:

``` 
ssh-copy-id root@192.168.217.129
The authenticity of host '192.168.217.129 (192.168.217.129)' can't be established.
ECDSA key fingerprint is SHA256:20TtBMwQAz9KAEK2mApqOksugeZ2yi8oecLRKz8RwB4.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.217.129's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.217.129'"
and check to make sure that only the key(s) you wanted were added.
```

If you get the error **sign_and_send_pubkey: signing failed: agent refused operation** when trying to SSH to the server, run **ssh-add* to add the private key identities to the authentication agent. You can then verify they were added:

``` 
ssh-add -l
2048 SHA256:OLHfszGKJK7wPNhc6OLZv8NeMKOIDuV0OHmBeynLLVI /root/.ssh/id_rsa (RSA)
2048 SHA256:OLHfszGKJK7wPNhc6OLZv8NeMKOIDuV0OHmBeynLLVI root@kali (RSA)
```

Now you should be able to login with the public key:

``` 
ssh root@192.168.217.129
Last login: Fri Jan  5 15:06:34 2018 from 192.168.217.132
[root@CentOS7 ~]# 
```

That worked, but we don't want root to be able to directly SSH into our host. So edit the <code>/etc/ssh/sshd_config</code> file on the server. The default settings are commented, and you can add your own changes and leave them uncommented to override the defaults. I added the following to disable root login and password authentication:

``` 
PermitRootLogin no
PasswordAuthentication no
```

Restart ssh and try to login again: 

``` 
ssh root@192.168.217.129
root@192.168.217.129: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

In practice, you would generate your keys for an account that you will use on that system and if needed, give it sudo privileges to do the work you need.

``` 
 _____________________________________
/ You learn to write as if to someone \
| else because NEXT YEAR YOU WILL BE  |
\ "SOMEONE ELSE."                     /
 -------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
