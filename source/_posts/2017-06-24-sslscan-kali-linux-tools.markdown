---
layout: post
title: "sslscan - Kali Linux tools"
date: 2017-06-24 07:36:03 -0400
comments: true
categories: [penetration testing, tools]
keywords: sslscan, sslscan linux, sslscan kali, sslscan tool, ssl scan, ssl scanner
description: sslscan tutorial
---

**Objective**: you want to assess the SSL security posture of a target by listing the supported cipher suites. sslscan is a fast SSL/TLS scanner that has been extended from its original version, and at the time of this post, its last update was 2 days ago.

Homepage: https://github.com/rbsec/sslscan

<!-- more -->

## sslscan description

> sslscan  queries SSL/TLS services, such as HTTPS, in order to determine the ciphers that are supported.

> SSLScan is designed to be easy, lean and fast. The output includes preferred  ciphers of the SSL/TLS service, and 
> text and XML output formats are supported. It is TLS SNI aware when used with a  supported  version of OpenSSL.

> Output is colour coded to indicate security issues. Colours are as follows:

> Red Background:  NULL cipher (no encryption)

> Red:             Broken cipher (<= 40 bit), broken  protocol  (SSLv2  or SSLv3) or broken certificate signing 
> algorithm (MD5)

> Yellow:           Weak  cipher  (<=  56  bit or RC4) or weak certificate signing algorithm (SHA-1)

> Purple:          Anonymous cipher (ADH or AECDH)

Manpage: http://manpages.ubuntu.com/manpages/xenial/man1/sslscan.1.html

## sslscan options

``` 
           ___ ___| |___  ___ __ _ _ __
          / __/ __| / __|/ __/ _` | '_ \
          \__ \__ \ \__ \ (_| (_| | | | |
          |___/___/_|___/\___\__,_|_| |_|


		1.11.10-static
		OpenSSL 1.0.2-chacha (1.0.2g-dev)
Command:
  sslscan [Options] [host:port | host]

Options:
  --targets=<file>     A file containing a list of hosts to check.
                       Hosts can  be supplied  with ports (host:port)
  --sni-name=<name>    Hostname for SNI
  --ipv4               Only use IPv4
  --ipv6               Only use IPv6
  --show-certificate   Show full certificate information
  --no-check-certificate  Don't warn about weak certificate algorithm or keys
  --show-client-cas    Show trusted CAs for TLS client auth
  --show-ciphers       Show supported client ciphers
  --show-cipher-ids    Show cipher ids
  --show-times         Show handhake times in milliseconds
  --ssl2               Only check SSLv2 ciphers
  --ssl3               Only check SSLv3 ciphers
  --tls10              Only check TLSv1.0 ciphers
  --tls11              Only check TLSv1.1 ciphers
  --tls12              Only check TLSv1.2 ciphers
  --tlsall             Only check TLS ciphers (all versions)
  --ocsp               Request OCSP response from server
  --pk=<file>          A file containing the private key or a PKCS#12 file
                       containing a private key/certificate pair
  --pkpass=<password>  The password for the private  key or PKCS#12 file
  --certs=<file>       A file containing PEM/ASN1 formatted client certificates
  --no-ciphersuites    Do not check for supported ciphersuites
  --no-fallback        Do not check for TLS Fallback SCSV
  --no-renegotiation   Do not check for TLS renegotiation
  --no-compression     Do not check for TLS compression (CRIME)
  --no-heartbleed      Do not check for OpenSSL Heartbleed (CVE-2014-0160)
  --starttls-ftp       STARTTLS setup for FTP
  --starttls-imap      STARTTLS setup for IMAP
  --starttls-irc       STARTTLS setup for IRC
  --starttls-ldap      STARTTLS setup for LDAP
  --starttls-pop3      STARTTLS setup for POP3
  --starttls-smtp      STARTTLS setup for SMTP
  --starttls-mysql     STARTTLS setup for MYSQL
  --starttls-xmpp      STARTTLS setup for XMPP
  --starttls-psql      STARTTLS setup for PostgreSQL
  --xmpp-server        Use a server-to-server XMPP handshake
  --http               Test a HTTP connection
  --rdp                Send RDP preamble before starting scan
  --bugs               Enable SSL implementation bug work-arounds
  --timeout=<sec>      Set socket timeout. Default is 3s
  --sleep=<msec>       Pause between connection request. Default is disabled
  --xml=<file>         Output results to an XML file
                       <file> can be -, which means stdout
  --version            Display the program version
  --verbose            Display verbose output
  --no-cipher-details  Disable EC curve names and EDH/RSA key lengths output
  --no-colour          Disable coloured output
  --help               Display the  help text  you are  now reading

Example:
  sslscan 127.0.0.1
  sslscan [::1]
```

## sslscan usage

``` 
sslscan https://www.cylance.com
Version: 1.11.10-static
OpenSSL 1.0.2-chacha (1.0.2g-dev)

Testing SSL server www.cylance.com on port 443 using SNI name www.cylance.com

  TLS Fallback SCSV:
Server supports TLS Fallback SCSV

  TLS renegotiation:
Secure session renegotiation supported

  TLS Compression:
Compression disabled

  Heartbleed:
TLS 1.2 not vulnerable to heartbleed
TLS 1.1 not vulnerable to heartbleed
TLS 1.0 not vulnerable to heartbleed

  Supported Server Cipher(s):
Preferred TLSv1.2  128 bits  ECDHE-RSA-AES128-GCM-SHA256   Curve P-256 DHE 256
Accepted  TLSv1.2  128 bits  ECDHE-RSA-AES128-SHA256       Curve P-256 DHE 256
Accepted  TLSv1.2  128 bits  ECDHE-RSA-AES128-SHA          Curve P-256 DHE 256
Accepted  TLSv1.2  256 bits  ECDHE-RSA-AES256-GCM-SHA384   Curve P-256 DHE 256
Accepted  TLSv1.2  256 bits  ECDHE-RSA-AES256-SHA384       Curve P-256 DHE 256
Accepted  TLSv1.2  256 bits  ECDHE-RSA-AES256-SHA          Curve P-256 DHE 256
Accepted  TLSv1.2  128 bits  AES128-GCM-SHA256            
Accepted  TLSv1.2  128 bits  AES128-SHA256                
Accepted  TLSv1.2  128 bits  AES128-SHA                   
Accepted  TLSv1.2  256 bits  AES256-GCM-SHA384            
Accepted  TLSv1.2  256 bits  AES256-SHA256                
Accepted  TLSv1.2  256 bits  AES256-SHA                   
Accepted  TLSv1.2  112 bits  DES-CBC3-SHA                 
Preferred TLSv1.1  128 bits  ECDHE-RSA-AES128-SHA          Curve P-256 DHE 256
Accepted  TLSv1.1  256 bits  ECDHE-RSA-AES256-SHA          Curve P-256 DHE 256
Accepted  TLSv1.1  128 bits  AES128-SHA                   
Accepted  TLSv1.1  256 bits  AES256-SHA                   
Accepted  TLSv1.1  112 bits  DES-CBC3-SHA                 
Preferred TLSv1.0  128 bits  ECDHE-RSA-AES128-SHA          Curve P-256 DHE 256
Accepted  TLSv1.0  256 bits  ECDHE-RSA-AES256-SHA          Curve P-256 DHE 256
Accepted  TLSv1.0  128 bits  AES128-SHA                   
Accepted  TLSv1.0  256 bits  AES256-SHA                   
Accepted  TLSv1.0  112 bits  DES-CBC3-SHA                 

  SSL Certificate:
Signature Algorithm: sha256WithRSAEncryption
RSA Key Strength:    2048

Subject:  *.cylance.com
Altnames: DNS:*.cylance.com, DNS:cylance.com, DNS:www.cylance.com, DNS:info.cylance.com, DNS:blog.cylance.com, DNS:education.cylance.com, DNS:support.cylance.com
Issuer:   DigiCert SHA2 Secure Server CA

Not valid before: May 24 00:00:00 2017 GMT
Not valid after:  May 29 12:00:00 2018 GMT
```

``` 
 ____________________________________
/ Q: Do you know what the death rate \
\ around here is? A: One per person. /
 ------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

