---
layout: post
title: "LFCS prep - LDAP server and client configuration"
date: 2018-02-11 10:37:49 -0500
comments: true
categories: [linux, sysadmin]
keywords: ldap, authconfig, ldap server, ldap client, openldap, ldap authentication, lfcs ldap, lfcs, rhcsa, lfcsa
description: Configuring LDAP authentication
---

Something that you may come up with in the exam and that I've been avoiding is the LDAP topic. In this post we'll look at server and client configuration for LDAP.

For this scenario, the hostname of my LDAP server is centos.example.com. The domain is example.com and the entry is inside /etc/hosts

I used [CertDepot](https://www.certdepot.net/rhel7-configure-ldap-directory-service-user-connection/) as inspiration..the LDAP syntax and topic really keeps me away from approaching it.

<!-- more -->

## Server configuration 

To get started, install these packages:

``` 
yum install -y openldap openldap-clients openldap-servers migrationtools
```

Some info about them:

- openldap is the open source implementation of LDAP 

- openldap-clients contains LDAP client utilities 

- openldap-servers is the server package 

- The MigrationTools are a set of Perl scripts for migrating users, groups, aliases, hosts, netgroups, networks, protocols, RPCs, and services from existing nameservices (flat files, NIS, and NetInfo) to LDAP.

Create LDAP password from a key, (the string secret below):

``` 
slappasswd -s secret -n > /etc/openldap/passwd
```

Inside the file take note of the generated password: <code>{SSHA}T4srVIBK+rJ9DXlVG7dvZnnAQhAZuO07</code>

Generate an X509 certificate valid for 1 year (note the paths of the certificate and private key):

``` 
openssl req -new -x509 -nodes -out /etc/openldap/certs/cert.pem -keyout /etc/openldap/certs/key.pem -days 365
```

At the prompt, you can leave as blank the values except the below, where you put your server's hostname:

``` 
Common Name (eg, your name or your server's hostname) []:centos.example.com
```

Change the ownership of the files inside <code>/etc/openldap/certs</code> and change the permissions of the private key:

``` 
cd /etc/openldap/certs
chown ldap:ldap *
chmod 600 key.pem 
ll
total 88
-rw-r--r--. 1 ldap ldap 65536 Feb 11 14:08 cert8.db
-rw-r--r--. 1 ldap ldap  1298 Feb 11 14:27 cert.pem
-rw-r--r--. 1 ldap ldap 16384 Feb 11 14:08 key3.db
-rw-------. 1 ldap ldap  1704 Feb 11 14:27 key.pem
-r--r-----. 1 ldap ldap    45 Apr 26  2016 password
-rw-r--r--. 1 ldap ldap 16384 Apr 26  2016 secmod.db
```

Copy the LDAP database:

``` 
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
```

Check the slapd configuration (below errors are safe):

``` 
slaptest
5a8038c4 hdb_db_open: database "dc=my-domain,dc=com": db_open(/var/lib/ldap/id2entry.bdb) failed: No such file or directory (2).
5a8038c4 backend_startup_one (type=hdb, suffix="dc=my-domain,dc=com"): bi_db_open failed! (2)
slap_startup failed (test would succeed using the -u switch)
```

Change the ownership of the DB to ldap:

``` 
chown ldap:ldap /var/lib/ldap/*
```

Now start the service

``` 
systemctl start slapd
```

Check that the LDAP service is listening:

``` 
netstat -lpt | grep ldap
tcp        0      0 0.0.0.0:ldap            0.0.0.0:*               LISTEN      5558/slapd          
tcp6       0      0 [::]:ldap               [::]:*                  LISTEN      5558/slapd    
```

Next you have to add some LDAP schemas

``` 
cd /etc/openldap/schema
```

Mainly, add the following:

- cosine (Cosine and Internet X.500)

``` 
ldapadd -Y EXTERNAL -H ldapi:/// -D "cn=config" -f cosine.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"
```

- nis (Network Information Services)

``` 
ldapadd -Y EXTERNAL -H ldapi:/// -D "cn=config" -f nis.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"
```

Create the <code>/etc/openldap/changes.ldif</code> file with the below content and remember the password value from earlier:

``` 
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=example,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=example,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}T4srVIBK+rJ9DXlVG7dvZnnAQhAZuO07

dn: cn=config
changetype: modify
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/cert.pem

dn: cn=config
changetype: modify
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/key.pem

dn: cn=config
changetype: modify
replace: olcLogLevel
olcLogLevel: -1

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=example,dc=com" read by * none
```

In summary, here you have to be mindful of the certificate and key paths, and of your previously generated password.

Now create the configuration:

``` 
ldapmodify -Y EXTERNAL -H ldapi:/// -f /etc/openldap/changes.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "cn=config"

modifying entry "cn=config"

modifying entry "cn=config"

modifying entry "olcDatabase={1}monitor,cn=config"
```

Create the file <code>/etc/openldap/base.ldif</code> with the following content:

``` 
dn: dc=example,dc=com
dc: example
objectClass: top
objectClass: domain

dn: ou=People,dc=example,dc=com
ou: People
objectClass: top
objectClass: organizationalUnit

dn: ou=Group,dc=example,dc=com
ou: Group
objectClass: top
objectClass: organizationalUnit
```

Create the directory structure:

``` 
ldapadd -x -w secret -D cn=Manager,dc=example,dc=com -f /etc/openldap/base.ldif
adding new entry "dc=example,dc=com"

adding new entry "ou=People,dc=example,dc=com"

adding new entry "ou=Group,dc=example,dc=com"
```

This command uses simple authentication and reads the previously created file.

Now it's time to create a sample user account and see if things are working:

``` 
mkdir /home/ldap
useradd -d /home/ldap/sam sam
passwd sam 
```

Some additional steps are needed for user account migration:

``` 
cd /usr/share/migrationtools 
```

Edit **migrate_common.ph** to have your domain values:

``` 
# Default DNS domain
$DEFAULT_MAIL_DOMAIN = "example.com";

# Default base 
$DEFAULT_BASE = "dc=example,dc=com";
```

Add the user to the directory:

``` 
cat /etc/passwd | grep sam > passwd
cat passwd 
sam:x:1004:1004::/home/ldap/sam:/bin/bash
ldapadd -x -w secret -D cn=Manager,dc=example,dc=com -f users.ldif
./migrate_passwd.pl passwd users.ldif
```

Check out the *users.ldif* file:

``` 
cat users.ldif 
dn: uid=sam,ou=People,dc=example,dc=com
uid: sam
cn: sam
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword: {crypt}$6$L49gzmY1$n0RNoX86rhT4QMvmsKWkMruHT/BCh7yBm2cQOdb4pbOA7XM0RB4d4GsoDpSILQSjYduiPponzFDb8O7bIb1kA.
shadowLastChange: 17573
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1004
gidNumber: 1004
homeDirectory: /home/ldap/sam
```

Add the user entry:

``` 
ldapadd -x -w secret -D cn=Manager,dc=example,dc=com -f users.ldif
adding new entry "uid=sam,ou=People,dc=example,dc=com"
```

Do the same for group:

``` 
cat /etc/group | grep sam > group
cat group
sam:x:1004:
```

Migrate the group:

``` 
./migrate_group.pl group groups.ldif
```

Check the groups.ldif file:

``` 
cat groups.ldif 
dn: cn=sam,ou=Group,dc=example,dc=com
objectClass: posixGroup
objectClass: top
cn: sam
userPassword: {crypt}x
gidNumber: 1004
```

Add the group:

``` 
ldapadd -x -w secret -D cn=Manager,dc=example,dc=com -f groups.ldif
adding new entry "cn=sam,ou=Group,dc=example,dc=com"
```

Now search for the user:

``` 
ldapsearch -x cn=sam -b dc=example,dc=com
# extended LDIF
#
# LDAPv3
# base <dc=example,dc=com> with scope subtree
# filter: cn=sam
# requesting: ALL
#

# sam, People, example.com
dn: uid=sam,ou=People,dc=example,dc=com
uid: sam
cn: sam
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JEw0OWd6bVkxJG4wUk5vWDg2cmhUNFFNdm1zS1drTXJ1SFQvQkN
 oN3lCbTJjUU9kYjRwYk9BN1hNMFJCNGQ0R3NvRHBTSUxRU2pZZHVpUHBvbnpGRGI4TzdiSWIxa0Eu
shadowLastChange: 17573
shadowMin: 0
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 1004
gidNumber: 1004
homeDirectory: /home/ldap/sam

# sam, Group, example.com
dn: cn=sam,ou=Group,dc=example,dc=com
objectClass: posixGroup
objectClass: top
cn: sam
userPassword:: e2NyeXB0fXg=
gidNumber: 1004

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
```

You might also need to allow ldap in the firewall. 

## Client configuration

If you hoped it was over, we still have to verify the authentication from a client machine. On my client, I also added the server in /etc/hosts:

``` 
192.168.217.131 centos.example.com
```

The interface used for configuring authentication is <code>authconfig</code>. For this example I will go over the **nslcd** route. 

On the client, install the needed packages:

``` 
yum install -y openldap-clients nss-pam-ldapd
```

Run the following:

``` 
authconfig --enableforcelegacy --update
authconfig --enableldap --enableldapauth --ldapserver="centos.example.com" --ldapbasedn="dc=example,dc=com" --update
```

Download the certificate:

``` 
scp root@centos.example.com:/etc/openldap/certs/cert.pem /etc/openldap/cacerts/cert.pem
root@centos.example.com's password: 
cert.pem                                      100% 1298   107.3KB/s   00:00    
```

Enable TLS:

``` 
authconfig --enableldaptls --update
```

You can do a getent or try to login as the LDAP user to see if it works.

``` 
 ________________________________________
/ Your temporary financial embarrassment \
| will be relieved in a surprising       |
\ manner.                                /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
