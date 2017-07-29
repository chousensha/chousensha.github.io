---
layout: post
title: "sslyze - Kali Linux tools"
date: 2017-07-16 04:57:44 -0400
comments: true
categories: [penetration testing, tools]
keywords: sslyze, sslyze python, sslyze kali, sslyze tool, sslyze commands, sslyze tutorial
description: sslyze tutorial
---

**Objective**: test the SSL/TLS security posture of a target as a standalone tool or as a custom made solution. sslyze is a fast and powerful SSL/TLS scanning Python tool that can be used both from the command line or as a library to include in your own scripts. It's being updated frequently and it's been tested on Windows, Linux and MacOS platforms.

Homepage: https://github.com/nabla-c0d3/sslyze

<!-- more -->

## sslyze description

> SSLyze is a Python library and a CLI tool that can analyze the SSL configuration of a server by connecting to it. It
> is designed to be fast and comprehensive, and should help organizations and testers identify mis-configurations 
> affecting their SSL/TLS servers.

> Key features include:

> * Python API, in order to run scans and process the results directly from Python.

> * Scans are automatically dispatched among multiple processes, making them very fast.

> * Performance testing: session resumption and TLS tickets support.

> * Security testing: weak cipher suites, insecure renegotiation, CRIME, Heartbleed and more.

> * Server certificate validation and revocation checking through OCSP stapling.

> * Support for StartTLS handshakes on SMTP, XMPP, LDAP, POP, IMAP, RDP, PostGres and FTP.

> * Support for client certificates when scanning servers that perform mutual authentication.

> * Scan results can be written to an XML or JSON file for further processing.

> * And much more!

> SSLyze is all Python code but it uses an OpenSSL wrapper written in C called nassl, which was specifically developed
> for allowing SSLyze to access the low-level OpenSSL APIs needed to perform deep SSL testing.

Documentation: https://nabla-c0d3.github.io/sslyze/documentation/

## sslyze options

``` 
Usage: sslyze.py [options] target1.com target2.com:443 etc...

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  --xml_out=XML_FILE    Writes the scan results as an XML document to the file
                        XML_FILE. If XML_FILE is set to "-", the XML output
                        will instead be printed to stdout.
  --targets_in=TARGETS_IN
                        Reads the list of targets to scan from the file
                        TARGETS_IN. It should contain one host:port per line.
  --timeout=TIMEOUT     Sets the timeout value in seconds used for every
                        socket connection made to the target server(s).
                        Default is 5s.
  --nb_retries=NB_RETRIES
                        Sets the number retry attempts for all network
                        connections initiated throughout the scan. Increase
                        this value if you are getting a lot of
                        timeout/connection errors when scanning a specific
                        server. Decrease this value to increase the speed of
                        the scans; results may however return connection
                        errors. Default is 4 connection attempts.
  --https_tunnel=HTTPS_TUNNEL
                        Tunnels all traffic to the target server(s) through an
                        HTTP CONNECT proxy. HTTP_TUNNEL should be the proxy's
                        URL: 'http://USER:PW@HOST:PORT/'. For proxies
                        requiring authentication, only Basic Authentication is
                        supported.
  --starttls=STARTTLS   Performs StartTLS handshakes when connecting to the
                        target server(s). STARTTLS should be one of: ['smtp',
                        'xmpp', 'xmpp_server', 'pop3', 'ftp', 'imap', 'ldap',
                        'rdp', 'postgres', 'auto']. The 'auto' option will
                        cause SSLyze to deduce the protocol (ftp, imap, etc.)
                        from the supplied port number, for each target
                        servers.
  --xmpp_to=XMPP_TO     Optional setting for STARTTLS XMPP.  XMPP_TO should be
                        the hostname to be put in the 'to' attribute of the
                        XMPP stream. Default is the server's hostname.
  --sni=SNI             Use Server Name Indication to specify the hostname to
                        connect to. Will only affect TLS 1.0+ connections.
  --quiet               Hide script standard outputs. Will only affect script
                        output if --xml_out is set.
  --regular             Regular HTTPS scan; shortcut for --sslv2 --sslv3
                        --tlsv1 --tlsv1_1 --tlsv1_2 --reneg --resum
                        --certinfo=basic --http_get --hide_rejected_ciphers
                        --compression --heartbleed

  Client certificate support:
    --cert=CERT         Client certificate chain filename. The certificates
                        must be in PEM format and must be sorted starting with
                        the subject's client certificate, followed by
                        intermediate CA certificates if applicable.
    --key=KEY           Client private key filename.
    --keyform=KEYFORM   Client private key format. DER or PEM (default).
    --pass=KEYPASS      Client private key passphrase.

  PluginOpenSSLCipherSuites:
    Scans the server(s) for supported OpenSSL cipher suites.

    --sslv2             Lists the SSL 2.0 OpenSSL cipher suites supported by
                        the server(s).
    --sslv3             Lists the SSL 3.0 OpenSSL cipher suites supported by
                        the server(s).
    --tlsv1             Lists the TLS 1.0 OpenSSL cipher suites supported by
                        the server(s).
    --tlsv1_1           Lists the TLS 1.1 OpenSSL cipher suites supported by
                        the server(s).
    --tlsv1_2           Lists the TLS 1.2 OpenSSL cipher suites supported by
                        the server(s).
    --http_get          Option - For each cipher suite, sends an HTTP GET
                        request after completing the SSL handshake and returns
                        the HTTP status code.
    --hide_rejected_ciphers
                        Option - Hides the (usually long) list of cipher
                        suites that were rejected by the server(s).

  PluginChromeSha1Deprecation:
    --chrome_sha1       Determines if the server will be affected by Google
                        Chrome's SHA-1 deprecation plans. See
                        http://googleonlinesecurity.blogspot.com/2014/09
                        /gradually-sunsetting-sha-1.html for more information

  PluginSessionRenegotiation:
    --reneg             Tests the server(s) for client-initiated renegotiation
                        and secure renegotiation support.

  PluginHeartbleed:
    --heartbleed        Tests the server(s) for the OpenSSL Heartbleed
                        vulnerability (experimental).

  PluginCertInfo:
    --certinfo=CERTINFO
                        Verifies the validity of the server(s) certificate(s)
                        against various trust stores, checks for support for
                        OCSP stapling, and prints relevant fields of the
                        certificate. CERTINFO should be 'basic' or 'full'.
    --ca_file=CA_FILE   Local Certificate Authority file (in PEM format), to
                        verify the validity of the server(s) certificate(s)
                        against.

  PluginCompression:
    --compression       Tests the server(s) for Zlib compression support.

  PluginHSTS:
    --hsts              Checks support for HTTP Strict Transport Security
                        (HSTS) by collecting any Strict-Transport-Security
                        field present in the HTTP response sent back by the
                        server(s).

  PluginSessionResumption:
    Analyzes the target server's SSL session resumption capabilities.

    --resum             Tests the server(s) for session resumption support
                        using session IDs and TLS session tickets (RFC 5077).
    --resum_rate        Performs 100 session resumptions with the server(s),
                        in order to estimate the session resumption rate.
```

## sslyze usage

``` 
sslyze --regular bugcrowd.com



 AVAILABLE PLUGINS
 -----------------

  PluginSessionResumption
  PluginCompression
  PluginSessionRenegotiation
  PluginChromeSha1Deprecation
  PluginHSTS
  PluginHeartbleed
  PluginOpenSSLCipherSuites
  PluginCertInfo



 CHECKING HOST(S) AVAILABILITY
 -----------------------------

   bugcrowd.com:443                    => 104.20.4.239:443



 SCAN RESULTS FOR BUGCROWD.COM:443 - 104.20.4.239:443
 ----------------------------------------------------

  * Deflate Compression:
      OK - Compression disabled          

  * Session Renegotiation:
      Client-initiated Renegotiations:   OK - Rejected
      Secure Renegotiation:              OK - Supported

  * OpenSSL Heartbleed:
      OK - Not vulnerable to Heartbleed  

  * Session Resumption:
      With Session IDs:                  OK - Supported (5 successful, 0 failed, 0 errors, 5 total attempts).
      With TLS Session Tickets:          OK - Supported

  * SSLV2 Cipher Suites:
      Server rejected all cipher suites.

  * Certificate - Content:
      SHA1 Fingerprint:                  db65dbb15a1819ad4692c47cfc0bc966beb6e44f
      Common Name:                       bugcrowd.com
      Issuer:                            DigiCert SHA2 Extended Validation Server CA
      Serial Number:                     075FE475034104F4909DDD4B5CD8C1AC
      Not Before:                        Oct 10 00:00:00 2015 GMT
      Not After:                         Oct 13 12:00:00 2017 GMT
      Signature Algorithm:               sha256WithRSAEncryption
      Public Key Algorithm:              rsaEncryption
      Key Size:                          2048 bit
      Exponent:                          65537 (0x10001)
      X509v3 Subject Alternative Name:   {'DNS': ['bugcrowd.com', 'forum.bugcrowd.com', 'blog.bugcrowd.com', 'docs.bugcrowd.com', 'tracker.bugcrowd.com']}

  * Certificate - Trust:
      Hostname Validation:               OK - Subject Alternative Name matches
      Google CA Store (09/2015):         OK - Certificate is trusted
      Java 6 CA Store (Update 65):       OK - Certificate is trusted
      Microsoft CA Store (09/2015):      OK - Certificate is trusted
      Mozilla NSS CA Store (09/2015):    OK - Certificate is trusted
      Apple CA Store (OS X 10.10.5):     OK - Certificate is trusted
      Certificate Chain Received:        ['bugcrowd.com', 'DigiCert SHA2 Extended Validation Server CA']

  * Certificate - OCSP Stapling:
      OCSP Response Status:              successful
      Validation w/ Mozilla's CA Store:  OK - Response is trusted
      Responder Id:                      3DD350A5D6A0ADEEF34A600A65D321D4F8F8D60F
      Cert Status:                       good
      Cert Serial Number:                075FE475034104F4909DDD4B5CD8C1AC
      This Update:                       Jul 12 12:10:00 2017 GMT
      Next Update:                       Jul 19 11:25:00 2017 GMT

  * TLSV1_2 Cipher Suites:
      Preferred:                       
                 ECDHE-RSA-AES128-GCM-SHA256   ECDH-256 bits  128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
      Accepted:                        
                 ECDHE-RSA-AES256-SHA384       ECDH-256 bits  256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 ECDHE-RSA-AES256-SHA          ECDH-256 bits  256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 ECDHE-RSA-AES256-GCM-SHA384   ECDH-256 bits  256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES256-SHA256                 -              256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES256-SHA                    -              256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES256-GCM-SHA384             -              256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 ECDHE-RSA-AES128-SHA256       ECDH-256 bits  128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 ECDHE-RSA-AES128-SHA          ECDH-256 bits  128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 ECDHE-RSA-AES128-GCM-SHA256   ECDH-256 bits  128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES128-SHA256                 -              128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES128-SHA                    -              128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES128-GCM-SHA256             -              128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/

  * TLSV1_1 Cipher Suites:
      Preferred:                       
                 ECDHE-RSA-AES128-SHA          ECDH-256 bits  128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
      Accepted:                        
                 ECDHE-RSA-AES256-SHA          ECDH-256 bits  256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES256-SHA                    -              256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 ECDHE-RSA-AES128-SHA          ECDH-256 bits  128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES128-SHA                    -              128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/

  * TLSV1 Cipher Suites:
      Preferred:                       
                 ECDHE-RSA-AES128-SHA          ECDH-256 bits  128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
      Accepted:                        
                 ECDHE-RSA-AES256-SHA          ECDH-256 bits  256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES256-SHA                    -              256 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 ECDHE-RSA-AES128-SHA          ECDH-256 bits  128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 AES128-SHA                    -              128 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/
                 DES-CBC3-SHA                  -              112 bits      HTTP 301 Unknown Error - https://www.bugcrowd.com/

  * SSLV3 Cipher Suites:
      Server rejected all cipher suites.



 SCAN COMPLETED IN 1.71 S
 ------------------------
```

``` 
 _______________________________________
/ Noise proves nothing. Often a hen who \
| has merely laid an egg cackles as if  |
| she laid an asteroid.                 |
|                                       |
\ -- Mark Twain                         /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

