---
layout: post
title: "urlcrazy - Kali Linux tools"
date: 2017-07-05 06:44:07 -0400
comments: true
categories: [penetration testing, tools]
keywords: urlcrazy, urlcrazy kali, urlcrazy tutorial, urlcrazy tool, urlcrazy example
description: urlcrazy tutorial
---

**Objective** you want to detect possible URL hijacking or phishing of a domain, where unsuspecting users are lured to a malicious domain that is very similar to the original one. urlcrazy ftw!

Homepage: https://www.morningstarsecurity.com/research/urlcrazy

<!-- more --> 

## urlcrazy description

> Generate and test domain typos and variations to detect and perform typo squatting, URL hijacking,
> phishing, and corporate espionage.

> Usage

> Detect typo squatters profiting from typos on your domain name

> Protect your brand by registering popular typos
    
> Identify typo domain names that will receive traffic intended for another domain

> Conduct phishing attacks during a penetration test

> Features

> Generates 15 types of domain variants

> Knows over 8000 common misspellings

> Supports cosmic ray induced bit flipping

> Multiple keyboard layouts (qwerty, azerty, qwertz, dvorak)

> Checks if a domain variant is valid

> Test if domain variants are in use

> Estimate popularity of a domain variant URLCrazy requires Linux and the Ruby interpreter.

## urlcrazy options

``` 
urlcrazy -h
URLCrazy version 0.5
by Andrew Horton (urbanadventurer)
http://www.morningstarsecurity.com/research/urlcrazy

Generate and test domain typos and variations to detect and perform typo squatting, URL hijacking,
phishing, and corporate espionage.

Supports the following domain variations:
Character omission, character repeat, adjacent character swap, adjacent character replacement, double
character replacement, adjacent character insertion, missing dot, strip dashes, singular or pluralise,
common misspellings, vowel swaps, homophones, bit flipping (cosmic rays), homoglyphs, wrong top level
domain, and wrong second level domain.

Usage: /usr/bin/urlcrazy [options] domain

Options
 -k, --keyboard=LAYOUT	Options are: qwerty, azerty, qwertz, dvorak (default: qwerty)
 -p, --popularity	Check domain popularity with Google
 -r, --no-resolve	Do not resolve DNS
 -i, --show-invalid	Show invalid domain names
 -f, --format=TYPE	Human readable or CSV (default: human readable)
 -o, --output=FILE	Output file
 -h, --help		This help
 -v, --version   	Print version information. This version is 0.5
```

## urlcrazy usage

``` 
urlcrazy kali.org
/usr/share/urlcrazy/tld.rb:81: warning: key "2nd_level_registration" is duplicated and overwritten on line 81
/usr/share/urlcrazy/tld.rb:89: warning: key "2nd_level_registration" is duplicated and overwritten on line 89
/usr/share/urlcrazy/tld.rb:91: warning: key "2nd_level_registration" is duplicated and overwritten on line 91
URLCrazy Domain Report
Domain    : kali.org
Keyboard  : qwerty
At        : 2017-07-05 06:57:50 -0400

# Please wait. 63 hostnames to process

Typo Type              Typo         DNS-A            CC-A               DNS-MX                                Extn  
--------------------------------------------------------------------------------------------------------------------
Character Omission     kai.org      111.89.129.15    JP,JAPAN           kai.org                               org   
Character Omission     kal.org      69.172.201.153   US,UNITED STATES                                         org   
Character Omission     kli.org      208.97.188.67    US,UNITED STATES   vade-in2.mail.dreamhost.com           org   
Character Repeat       kaali.org    184.168.221.56   US,UNITED STATES   mailstore1.secureserver.net           org   
Character Repeat       kalii.org                     ?                                                        org   
Character Repeat       kalli.org                     ?                  mail.joker.com                        org   
Character Repeat       kkali.org                     ?                                                        org   
Character Swap         akli.org     180.240.134.81   SG,SINGAPORE       mail.akli.org                         org   
Character Swap         kail.org     185.53.178.9                        mail.h-email.net                      org   
Character Swap         klai.org     74.208.59.167    US,UNITED STATES   mx01.1and1.com                        org   
Character Replacement  jali.org                      ?                  mailin1.totalweb.net.uk               org   
Character Replacement  kaki.org                      ?                                                        org   
Character Replacement  kalo.org     54.214.44.86     US,UNITED STATES   aspmx5.googlemail.com                 org   
Character Replacement  kalu.org     64.85.171.16     US,UNITED STATES   host.rack-host.net                    org   
Character Replacement  ksli.org     203.184.176.168  HK,HONG KONG                                             org   
Character Replacement  lali.org     194.232.43.3     AT,AUSTRIA         dedi3762.your-server.de               org   
Character Insertion    kalio.org    213.186.33.87    FR,FRANCE          mx2.ovh.net                           org   
Character Insertion    kaliu.org    67.205.40.20     US,UNITED STATES   vade-in1.mail.dreamhost.com           org   
Character Insertion    kalki.org    97.74.42.79      US,UNITED STATES   smtp.secureserver.net                 org   
Character Insertion    kasli.org    217.112.35.116   GB,UNITED KINGDOM  mx25.valuehost.ru                     org   
Character Insertion    kjali.org                     ?                                                        org   
Character Insertion    klali.org                     ?                                                        org   
Missing Dot            kaliorg.com                   ?                                                        com   
Missing Dot            wwwkali.org                   ?                                                        org   
Singular or Pluralise  kalis.org    217.160.233.111  DE,GERMANY         mx00.kundenserver.de                  org   
Vowel Swap             kala.org     64.111.125.111   US,UNITED STATES   ASPMX2.GOOGLEMAIL.COM                 org   
Vowel Swap             kale.org     185.53.178.9                        mail.h-email.net                      org   
Vowel Swap             keli.org                      ?                  mail.hti.ag                           org   
Vowel Swap             kili.org     69.172.201.153   US,UNITED STATES                                         org   
Vowel Swap             koli.org     72.52.4.122      US,UNITED STATES   localhost                             org   
Vowel Swap             kuli.org     37.120.172.191   JP,JAPAN           v2.v1.biz                             org   
Homophones             kalaye.org                    ?                                                        org   
Homophones             kaleye.org                    ?                                                        org   
Bit Flipping           cali.org     23.21.169.243    US,UNITED STATES   alt1.aspmx.l.google.com               org   
Bit Flipping           iali.org                      ?                                                        org   
Bit Flipping           kadi.org     146.148.34.125   US,UNITED STATES   mail.hope-mail.com                    org   
Bit Flipping           kahi.org     206.126.4.4      պ,                kahi-org.mail.protection.outlook.com  org   
Bit Flipping           kalh.org     50.63.33.1       E�,                mailstore1.secureserver.net           org   
Bit Flipping           kalk.org     137.74.127.233                                                            org   
Bit Flipping           kalm.org                      ?                  mx.kalm.org.cust.a.hostedemail.com    org   
Bit Flipping           kaly.org     207.148.248.143  US,UNITED STATES                                         org   
Bit Flipping           kami.org     69.172.201.153   ��,                                                      org   
Bit Flipping           kani.org     178.33.20.67     FR,FRANCE          mail.kani.org                         org   
Bit Flipping           kcli.org     50.63.202.44     US,UNITED STATES   ALT1.ASPMX.L.GOOGLE.COM               org   
Bit Flipping           kqli.org                      ?                                                        org   
Bit Flipping           oali.org     184.107.215.202  CA,CANADA          oali.org                              org   
Homoglyphs             ka1i.org                      ?                                                        org   
Homoglyphs             kall.org     65.254.238.128   US,UNITED STATES   mx.kall.org                           org   
Wrong TLD              kali.ca      76.74.246.51     US,UNITED STATES   kali.ca                               ca    
Wrong TLD              kali.ch      81.221.254.5     CH,SWITZERLAND     kali.ch.2.arsmtp.com                  ch    
Wrong TLD              kali.com     75.164.208.82    US,UNITED STATES   alt2.aspmx.l.google.com               com   
Wrong TLD              kali.de      193.238.9.165    DE,GERMANY                                               de    
Wrong TLD              kali.edu                      ?                                                        edu   
Wrong TLD              kali.es      212.227.247.181  DE,GERMANY         mx01.1and1.es                         es    
Wrong TLD              kali.fr      103.224.182.248                     mx92.mb5p.com                         fr    
Wrong TLD              kali.it      195.110.124.188  IT,ITALY           mail.register.it                      it    
Wrong TLD              kali.jp      8.6.19.68        US,UNITED STATES   smtp.secureserver.net                 jp    
Wrong TLD              kali.net     45.40.165.16                        ASPMX.L.GOOGLE.COM                    net   
Wrong TLD              kali.nl      72.52.4.121      US,UNITED STATES   localhost                             nl    
Wrong TLD              kali.no      194.63.248.52    NO,NORWAY          mx02.domeneshop.no                    no    
Wrong TLD              kali.ru      185.53.179.40                                                             ru    
Wrong TLD              kali.se      92.42.73.20      SE,SWEDEN          kali-se.mail.protection.outlook.com   se    
Wrong TLD              kali.us      184.168.221.28   US,UNITED STATES   209.150.186.51                        us   
```

* check domain popularity with no DNS resolution

``` 
urlcrazy -p -r paypal.com
/usr/share/urlcrazy/tld.rb:81: warning: key "2nd_level_registration" is duplicated and overwritten on line 81
/usr/share/urlcrazy/tld.rb:89: warning: key "2nd_level_registration" is duplicated and overwritten on line 89
/usr/share/urlcrazy/tld.rb:91: warning: key "2nd_level_registration" is duplicated and overwritten on line 91
URLCrazy Domain Report
Domain    : paypal.com
Keyboard  : qwerty
At        : 2017-07-05 07:01:52 -0400

# Please wait. 80 hostnames to process

Typo Type              Typo           Pop  CC-A  Extn  
-------------------------------------------------------
Character Omission     papal.com           ?     com   
Character Omission     payal.com           ?     com   
Character Omission     paypa.com           ?     com   
Character Omission     paypal.cm           ?     cm    
Character Omission     paypl.com           ?     com   
Character Omission     pypal.com           ?     com   
Character Repeat       paaypal.com         ?     com   
Character Repeat       paypaal.com         ?     com   
Character Repeat       paypall.com         ?     com   
Character Repeat       payppal.com         ?     com   
Character Repeat       payypal.com         ?     com   
Character Repeat       ppaypal.com         ?     com   
Character Swap         apypal.com          ?     com   
Character Swap         papyal.com          ?     com   
Character Swap         payapl.com          ?     com   
Character Swap         paypla.com          ?     com   
Character Swap         pyapal.com          ?     com   
Character Replacement  oaypal.com          ?     com   
Character Replacement  patpal.com          ?     com   
Character Replacement  paupal.com          ?     com   
Character Replacement  payoal.com          ?     com   
Character Replacement  paypak.com          ?     com   
Character Replacement  paypsl.com          ?     com   
Character Replacement  psypal.com          ?     com   
Character Insertion    pasypal.com         ?     com   
Character Insertion    paypalk.com         ?     com   
Character Insertion    paypasl.com         ?     com   
Character Insertion    paypoal.com         ?     com   
Character Insertion    paytpal.com         ?     com   
Character Insertion    payupal.com         ?     com   
Character Insertion    poaypal.com         ?     com   
Missing Dot            paypalcom.com       ?     com   
Missing Dot            wwwpaypal.com       ?     com   
Singular or Pluralise  paypals.com         ?     com   
Vowel Swap             peypel.com          ?     com   
Vowel Swap             piypil.com          ?     com   
Vowel Swap             poypol.com          ?     com   
Vowel Swap             puypul.com          ?     com   
Bit Flipping           0aypal.com          ?     com   
Bit Flipping           pa9pal.com          ?     com   
Bit Flipping           paipal.com          ?     com   
Bit Flipping           paqpal.com          ?     com   
Bit Flipping           paxpal.com          ?     com   
Bit Flipping           pay0al.com          ?     com   
Bit Flipping           paypad.com          ?     com   
Bit Flipping           paypah.com          ?     com   
Bit Flipping           paypam.com          ?     com   
Bit Flipping           paypan.com          ?     com   
Bit Flipping           paypcl.com          ?     com   
Bit Flipping           paypel.com          ?     com   
Bit Flipping           paypil.com          ?     com   
Bit Flipping           paypql.com          ?     com   
Bit Flipping           payqal.com          ?     com   
Bit Flipping           payral.com          ?     com   
Bit Flipping           paytal.com          ?     com   
Bit Flipping           payxal.com          ?     com   
Bit Flipping           pcypal.com          ?     com   
Bit Flipping           peypal.com          ?     com   
Bit Flipping           piypal.com          ?     com   
Bit Flipping           pqypal.com          ?     com   
Bit Flipping           qaypal.com          ?     com   
Bit Flipping           raypal.com          ?     com   
Bit Flipping           taypal.com          ?     com   
Bit Flipping           xaypal.com          ?     com   
Homoglyphs             paypa1.com          ?     com   
Wrong TLD              paypal.ca           ?     ca    
Wrong TLD              paypal.ch           ?     ch    
Wrong TLD              paypal.de           ?     de    
Wrong TLD              paypal.edu          ?     edu   
Wrong TLD              paypal.es           ?     es    
Wrong TLD              paypal.fr           ?     fr    
Wrong TLD              paypal.it           ?     it    
Wrong TLD              paypal.jp           ?     jp    
Wrong TLD              paypal.net          ?     net   
Wrong TLD              paypal.nl           ?     nl    
Wrong TLD              paypal.no           ?     no    
Wrong TLD              paypal.org          ?     org   
Wrong TLD              paypal.ru           ?     ru    
Wrong TLD              paypal.se           ?     se    
Wrong TLD              paypal.us           ?     us    
```

``` 
 ________________________________________
/ Everything that you know is wrong, but \
\ you can be straightened out.           /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


