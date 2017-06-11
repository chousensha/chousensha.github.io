---
layout: post
title: "Automater - Kali Linux tools"
date: 2017-06-11 03:57:26 -0400
comments: true
categories: [penetration testing, tools]
keywords: automater, automater tutorial, automater kali linux, automater kali
description: Automater tutorial
---

**Objective**: you want to check suspicious IPs, domains and hashes for maliciousness. Maybe you've heard that your favorite news site has been hacked and is serving malware to its users. You'd like to confirm if something is dangerous or not, without navigating to it and risking to get compromised in the process. There is a Python tool on Kali that can help you with just that! Enter Automater!

<!-- more -->

Homepage: http://www.tekdefense.com/automater/

If you want to get it on your distro, you can grab it from its [Github repository](https://github.com/1aN0rmus/TekDefense-Automater)

# Automater description

> Automater is a URL/Domain, IP Address, and Md5 Hash OSINT tool aimed at making the analysis process easier for 
> intrusion Analysts. Given a target (URL, IP, or HASH) or a file full of targets Automater will return relevant 
> results from sources like the following: IPvoid.com, Robtex.com, Fortiguard.com, unshorten.me, Urlvoid.com, 
> Labs.alienvault.com, ThreatExpert, VxVault, and VirusTotal. 

# Automater options

``` 
automater -h
usage: Automater.py [-h] [-o OUTPUT] [-b] [-f CEF] [-w WEB] [-c CSV]
                    [-d DELAY] [-s SOURCE] [--proxy PROXY] [-a USERAGENT] [-V]
                    [-r] [-v]
                    target

IP, URL, and Hash Passive Analysis tool

positional arguments:
  target                List one IP Address (CIDR or dash notation accepted),
                        URL or Hash to query or pass the filename of a file
                        containing IP Address info, URL or Hash to query each
                        separated by a newline.

optional arguments:
  -h, --help            show this help message and exit
  -o OUTPUT, --output OUTPUT
                        This option will output the results to a file.
  -b, --bot             This option will output minimized results for a bot.
  -f CEF, --cef CEF     This option will output the results to a CEF formatted
                        file.
  -w WEB, --web WEB     This option will output the results to an HTML file.
  -c CSV, --csv CSV     This option will output the results to a CSV file.
  -d DELAY, --delay DELAY
                        This will change the delay to the inputted seconds.
                        Default is 2.
  -s SOURCE, --source SOURCE
                        This option will only run the target against a
                        specific source engine to pull associated domains.
                        Options are defined in the name attribute of the site
                        element in the XML configuration file. This can be a
                        list of names separated by a semicolon.
  --proxy PROXY         This option will set a proxy to use (eg.
                        proxy.example.com:8080)
  -a USERAGENT, --useragent USERAGENT
                        This option allows the user to set the user-agent seen
                        by web servers being utilized. By default, the user-
                        agent is set to Automater/version
  -V, --vercheck        This option checks and reports versioning for
                        Automater. Checks each python module in the Automater
                        scope. Default, (no -V) is False
  -r, --refreshxml      This option refreshes the tekdefense.xml file from the
                        remote GitHub site. Default (no -r) is False.
  -v, --verbose         This option prints messages to the screen. Default (no
                        -v) is False.
```

You can see there are multiple output options for further processing, like CEF formatted file, CSV and HTML. Also, it's important to remember that tools usually come with their default user agents, and it might be beneficial to change this when you run it, and make it look like a real browser.

# Automater usage

* check IP for malware

``` 
automater 185.62.190.110
/usr/lib/python2.7/dist-packages/urllib3/connectionpool.py:845: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)

____________________     Results found for: 185.62.190.110     ____________________
No results found in the RTex DNS
No results found in the FNet URL
[+] VT ASN: No results found
[+] VT Country: ZZ
[+] VT AS Owner: No results found
[+] VT pDNS: ('2016-08-24 00:00:00', 'cl0.f-aws.com')
[+] VT pDNS: ('2017-05-19 00:00:00', 'mail.attw.io')
[+] VT pDNS: ('2016-06-05 00:00:00', 'weinne.net')
[+] VT pDNS: ('2017-02-08 00:00:00', 'www[.]woodmann.com')
[+] VT Malware: ('2017-06-02 10:46:35', 'ceeca0c7dc341fa57532470f2d7caaa427bf77e1e533b7ff3d9d8e245d6ea5fd')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/Bin_Corso_2009-2-4_20.20_Corso_7.02.34.rar', '2017-06-07 17:23:57')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Tools/Vbdec34.zip', '2017-06-04 23:44:26')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/tools/', '2017-06-02 11:41:58')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/tools/dongles/vdog104.zip', '2017-06-02 10:46:33')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/archives/bartpak7.rar', '2017-06-01 19:35:43')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/bin_de_decompiler_2009-7-18_22.55_de_decompiler_lite.zip|>de_decompiler_lite.exe', '2017-06-01 14:57:19')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/index.php/Category:RCE_Tools', '2017-05-31 09:16:42')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/index.php/Category:SoftICE_Extensions', '2017-05-27 18:28:27')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/forum/attachment.php?s=d1dedc683453c119989330b5967a2dea&attachmentid=2311&d=1278836882', '2017-05-26 20:45:35')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/fravia/exe/cryptpad.exe', '2017-05-26 18:00:12')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/Bin_LordPE_2010-6-29_3.9_LordPE_1.41_Deluxe_b.zip', '2017-05-26 13:50:29')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/index.php/LordPE', '2017-05-26 13:42:00')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/tools/dongles/admon25.rar', '2017-05-25 05:51:39')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/bin_wintruder_2008-10-24_22.21_wintruder.zip', '2017-05-24 12:33:31')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/Bin_Echo_Mirage_2014-1-11_17.28_EchoMirage-3.1.rar', '2017-05-24 10:42:52')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Archives/Kgensrcs.zip', '2017-05-19 18:39:05')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/bin_zeroadd_2014-5-9_1.29_zeroadd.zip', '2017-05-16 08:33:15')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Unpackers/Arm201b1.zip', '2017-05-11 22:16:38')
[+] VT Mal URLs: ("hxxp://www[.]woodmann.com/collaborative/knowledge/images/bin_stuxnet's_rootkit_(mrxnet)_into_c++_2011-2-6_13.54_mrxnet.rar", '2017-05-11 06:28:02')
[+] VT Mal URLs: ("hxxp://www[.]woodmann.com/collaborative/knowledge/images/Bin_Stuxnet's_Rootkit_(MRxNet)_into_C++_2011-2-6_13.54_MRxNet.rar", '2017-05-10 11:56:48')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/', '2017-05-09 13:25:03')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Tools/Dongles/Edgehasp.zip', '2017-05-09 06:50:13')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/index.php/Kernel_Detective', '2017-05-07 23:49:45')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/BobSoft/Pages/Plugins/ImmDbg', '2017-05-02 19:47:49')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/index.php/AdmiralDebilitate', '2017-04-29 23:08:33')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Tutorials/Nolflex3.htm', '2017-04-21 23:28:28')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/Bin_ImpREC_2011-7-16_8.11_ImpREC_1.7e.rar', '2017-04-20 17:33:51')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/index.php/Detect_It_Easy', '2017-04-19 22:55:52')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/forum/attachment.php', '2017-04-19 21:13:35')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/Bin_IIDKing_2007-10-19_23.37_tf23.zip', '2017-04-18 10:44:44')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/Bin_RTA_2011-9-6_20.52_rta2b2.zip', '2017-04-13 08:16:56')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Tools/Dongles/Haspdll.zip', '2017-04-11 17:42:56')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools', '2017-04-10 16:46:49')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Unpackers/ArmStripper01b6.rar', '2017-04-10 04:09:19')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/index.php/Echo_Mirage', '2017-04-08 16:34:47')
[+] VT Mal URLs: ('hxxp://185.62.190.110/accessroot/arteam/site/download.php?view.331', '2017-04-05 05:34:20')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/Bin_Unpacker_PECompact_2014-1-15_15.34_Unpacker_PECompact.rar', '2017-04-02 21:39:34')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/forum/forum.php', '2017-03-24 13:36:54')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/Bin_Superkill_2009-6-4_22.16_Superkill-V1.0.zip', '2017-03-16 01:02:57')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Tools/Wm.zip', '2017-03-16 01:02:57')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Tools/Cdtools.zip', '2017-03-16 01:02:57')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/FLEXlm/Lmv8gen.zip', '2017-03-15 18:06:40')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/krobar/collections/tkc/06.zip', '2017-03-14 19:32:26')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Archives/Crackmes.zip', '2017-03-14 19:32:26')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/knowledge/images/bin_virut.a_malware_analysis_paper_2010-9-3_15.53_virut.a.rar', '2017-03-14 14:34:44')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/index.php', '2017-03-14 09:22:56')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/krobar/collections/stones.zip', '2017-03-11 00:34:59')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/fravia/sources/WINUSER.H', '2017-03-08 11:57:20')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/bin_chimprec_2008-6-24_13.59_chimprec.zip|>chimprec.exe', '2017-03-08 11:15:11')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/tools/images/', '2017-03-03 07:12:55')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/krobar/collections/id-site.zip', '2017-02-24 01:16:48')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/collaborative/knowledge/images/Bin_Stuxnet&', '2017-02-15 15:19:26')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/BobSoft/Files/Other/PEiD-0.95-20081103.zip', '2017-02-15 00:46:58')
[+] VT Mal URLs: ('hxxp://www[.]woodmann.com/crackz/Tools/Miscarc.zip', '2017-02-09 14:37:08')
[+] VT Mal URLs: ('hxxp://cl0.f-aws.com/metin2sometimes/client/pack/icepack.mp3.lz', '2016-10-09 15:24:40')
[+] VT Mal URLs: ('hxxp://cl0.f-aws.com/metin2sometimes/client/pack/efect.txt.lz', '2016-10-09 15:24:40')
[+] VT Mal URLs: ('hxxp://cl0.f-aws.com/metin2sometimes/client/lib/__future__.pyc.lz', '2016-10-09 15:24:39')
[+] VT Mal URLs: ('hxxp://cl0.f-aws.com/metin2sometimes/client/miles/mssa3d.m3d.lz', '2016-10-09 15:24:39')
[+] VT Mal URLs: ('hxxp://cl0.f-aws.com/metin2sometimes/app/normalize.css', '2016-10-09 15:24:37')
[+] VT Mal URLs: ('hxxp://cl0.f-aws.com/metin2sometimes/client/bgm/xmas.mp3.lz', '2016-10-09 15:24:38')
[+] VT Mal URLs: ('hxxp://cl0.f-aws.com/metin2sometimes/client/devil.dll.lz', '2016-10-09 15:24:37')
[+] VT Mal URLs: ('hxxp://cl0.f-aws.com/metin2asgard/client/mark/10.tga.lz', '2016-09-27 17:36:49')
[+] VT Mal URLs: ('hxxp://cl0.f-aws.com/metin2asgard/client/mark/10_1.tga.lz', '2016-09-27 17:34:08')
[+] VT Mal URLs: ('hxxp://185.62.190.110/Deutsche-Bank/db/erfolg.html', '2016-03-07 08:56:47')
[+] VT Mal URLs: ('hxxp://185.62.190.110/Deutsche-Bank/db/db.php', '2016-02-29 13:36:45')
[+] Blacklist from IPVoid: No results found
[+] ISP from IPvoid: No results found
[+] Country from IPVoid: No results found
[+] Malc0de Date: No results found
[+] Malc0de IP: No results found
[+] Malc0de Country: No results found
[+] Malc0de ASN: No results found
[+] Malc0de ASN Name: No results found
[+] Malc0de MD5: No results found
[+] Reputation Authority Score: 50/100
[+] FreeGeoIP Country Name: Netherlands
[+] FreeGeoIP Region Name: No results found
[+] FreeGeoIP City: No results found
[+] FreeGeoIP Zipcode: No results found
[+] FreeGeoIP Latitude: 52.3824
[+] FreeGeoIP Longitude: 4.8995
[+] SANS total target IPs seen: No results found
[+] SANS total packets blocked: No results found
[+] SANS last seen on: No results found
[+] SANS first seen on: No results found
No results found in the THIP
No results found in the TekHP
[+] ProjectHoneypot activity type: No results found
[+] ProjectHoneypot first mail received: No results found
[+] ProjectHoneypot last mail received: No results found
[+] ProjectHoneypot total mails received: No results found
[+] ProjectHoneypot spider first seen: No results found
[+] ProjectHoneypot spider last seen: No results found
[+] ProjectHoneypot spider sightings: No results found
[+] ProjectHoneypot user-agent sightings: No results found
[+] ProjectHoneypot first post on: No results found
[+] ProjectHoneypot last post on: No results found
[+] ProjectHoneypot form posts: No results found
[+] ProjectHoneypot first rule break on: No results found
[+] ProjectHoneypot last rule break on: No results found
[+] ProjectHoneypot rule break sightings: No results found
[+] ProjectHoneypot first dictionary attack on: No results found
[+] ProjectHoneypot last dictionary attack on: No results found
[+] ProjectHoneypot dictionary attack sightings: No results found
[+] ProjectHoneypot harvester first seen: No results found
[+] ProjectHoneypot harvester last seen: No results found
[+] ProjectHoneypot harvester sightings: No results found
[+] ProjectHoneypot harvester results: No results found
```

I've used the IP of the Woodmann reverse engineering community. From the output, you can see that it's clean, excepting some files flagged by VirusTotal, as would be expected from a place where executable samples are shared for RE :)

* check domain for malware

``` 
automater corefitness.info
/usr/lib/python2.7/dist-packages/urllib3/connectionpool.py:845: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)

____________________     Results found for: corefitness.info     ____________________
No results found in the FNet URL
No results found in the Un Redirect
[+] IP from URLVoid: 76.74.155.21
[+] Blacklist from URLVoid: No results found
[+] Domain Age from URLVoid: 2009-12-08 (8 years ago)
[+] Geo Coordinates from URLVoid: 40.6888 / -74.0203
[+] Country from URLVoid:  (US) United States
[+] pDNS data from VirusTotal: ('2017-04-02', '107.180.51.40')
[+] pDNS data from VirusTotal: ('2016-12-14', '76.74.155.21')
[+] pDNS malicious URLs from VirusTotal: ('2017-06-10', 'hxxp://corefitness.info/')
[+] pDNS malicious URLs from VirusTotal: ('2017-06-09', 'hxxp://corefitness.info/b/')
[+] pDNS malicious URLs from VirusTotal: ('2017-06-07', 'hxxp://corefitness.info/')
[+] pDNS malicious URLs from VirusTotal: ('2017-04-04', 'hxxp://corefitness.info/')
[+] pDNS malicious URLs from VirusTotal: ('2017-03-16', 'hxxp://corefitness.info/')
[+] pDNS malicious URLs from VirusTotal: ('2017-03-08', 'hxxp://corefitness.info/workouts/men/arms/')
[+] pDNS malicious URLs from VirusTotal: ('2017-03-07', 'hxxp://corefitness.info/')
[+] pDNS malicious URLs from VirusTotal: ('2017-02-25', 'hxxp://corefitness.info/workouts/women/arms/')
[+] pDNS malicious URLs from VirusTotal: ('2017-02-05', 'hxxp://corefitness.info/workouts/men/arms/')
[+] pDNS malicious URLs from VirusTotal: ('2016-12-23', 'hxxp://corefitness.info/b/')
[+] pDNS malicious URLs from VirusTotal: ('2016-12-22', 'hxxp://corefitness.info/')
[+] pDNS malicious URLs from VirusTotal: ('2016-12-21', 'hxxp://corefitness.info/1.exe/')
[+] Malc0de Date: No results found
[+] Malc0de IP: No results found
[+] Malc0de Country: No results found
[+] Malc0de ASN: No results found
[+] Malc0de ASN Name: No results found
[+] Malc0de MD5: No results found
No results found in the THIP
[+] McAfee Web Risk: No results found
[+] McAfee Web Category: No results found
[+] McAfee Last Seen: No results found
```

Here I picked a random URL from the [SANS suspicious domains list](https://isc.sans.edu/feeds/suspiciousdomains_High.txt). It seems that if your search for fitness leads you there, your computer won't be very fit D:

* chech hash for maliciousness

``` 
automater b9318a66fa7f50f2f3ecaca02a96268ad2c63db7554ea3acbde43bf517328d06
/usr/lib/python2.7/dist-packages/urllib3/connectionpool.py:845: InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
  InsecureRequestWarning)

____________________     Results found for: b9318a66fa7f50f2f3ecaca02a96268ad2c63db7554ea3acbde43bf517328d06     ____________________
[+] MD5 found on VT: 1
[+] Scan date submitted: 2017-06-09 04:35:58
[+] Detected Engines: 54
[+] Total Engines: 61
[+] Vendor | Classification: ('MicroWorld-eScan', 'Trojan.Ransom.WannaCryptor.H')
[+] Vendor | Classification: ('nProtect', 'Ransom/W32.WannaCry.Zen')
[+] Vendor | Classification: ('CAT-QuickHeal', 'Ransom.WannaCrypt.A4')
[+] Vendor | Classification: ('McAfee', 'Ransom-WannaCry!4287E15AF619')
[+] Vendor | Classification: ('Malwarebytes', 'Ransom.WannaCrypt')
[+] Vendor | Classification: ('Zillya', 'Trojan.WannaCryptGen.Win32.2')
[+] Vendor | Classification: ('SUPERAntiSpyware', 'Ransom.WannaCrypt/Variant')
[+] Vendor | Classification: ('K7GW', 'Exploit ( 0050d7a31 )')
[+] Vendor | Classification: ('K7AntiVirus', 'Exploit ( 0050d7a31 )')
[+] Vendor | Classification: ('Arcabit', 'Trojan.Ransom.WannaCryptor.H')
[+] Vendor | Classification: ('Baidu', 'Win32.Worm.Rbot.a')
[+] Vendor | Classification: ('Cyren', 'W32/Trojan.ZTSA-8671')
[+] Vendor | Classification: ('Symantec', 'Ransom.Wannacry')
[+] Vendor | Classification: ('TrendMicro-HouseCall', 'Ransom_WCRY.SMB')
[+] Vendor | Classification: ('Avast', 'Win32:WanaCry-A [Trj]')
[+] Vendor | Classification: ('Kaspersky', 'Trojan-Ransom.Win32.Wanna.m')
[+] Vendor | Classification: ('BitDefender', 'Trojan.Ransom.WannaCryptor.H')
[+] Vendor | Classification: ('NANO-Antivirus', 'Trojan.Win32.Wanna.eovgam')
[+] Vendor | Classification: ('AegisLab', 'Troj.Ransom.W32.Wanna.toNz')
[+] Vendor | Classification: ('Ad-Aware', 'Trojan.Ransom.WannaCryptor.H')
[+] Vendor | Classification: ('Emsisoft', 'Trojan-Ransom.WanaCrypt0r (A)')
[+] Vendor | Classification: ('Comodo', 'UnclassifiedMalware')
[+] Vendor | Classification: ('F-Secure', 'Trojan.Ransom.WannaCryptor.H')
[+] Vendor | Classification: ('DrWeb', 'Trojan.Encoder.11432')
[+] Vendor | Classification: ('VIPRE', 'Trojan.Win32.Generic!BT')
[+] Vendor | Classification: ('TrendMicro', 'Ransom_WCRY.SMB')
[+] Vendor | Classification: ('McAfee-GW-Edition', 'Ransom-WannaCry!4287E15AF619')
[+] Vendor | Classification: ('Sophos', 'Troj/Ransom-EMG')
[+] Vendor | Classification: ('SentinelOne', 'static engine - malicious')
[+] Vendor | Classification: ('F-Prot', 'W32/WannaCrypt.D')
[+] Vendor | Classification: ('Jiangmin', 'Trojan.WanaCry.i')
[+] Vendor | Classification: ('Webroot', 'W32.Ransom.Wannacry')
[+] Vendor | Classification: ('Avira', 'BDS/Agent.ilyda')
[+] Vendor | Classification: ('Endgame', 'malicious (high confidence)')
[+] Vendor | Classification: ('Microsoft', 'Ransom:Win32/WannaCrypt.A!rsm')
[+] Vendor | Classification: ('ZoneAlarm', 'Trojan-Ransom.Win32.Wanna.m')
[+] Vendor | Classification: ('ALYac', 'Trojan.Ransom.WannaCryptor')
[+] Vendor | Classification: ('AVware', 'Trojan.Win32.Generic!BT')
[+] Vendor | Classification: ('VBA32', 'Trojan.Filecoder')
[+] Vendor | Classification: ('ESET-NOD32', 'Win32/Exploit.CVE-2017-0147.A')
[+] Vendor | Classification: ('Tencent', 'Win32.Trojan.Raas.Auto')
[+] Vendor | Classification: ('Yandex', 'Exploit.CVE-2017-0147!')
[+] Vendor | Classification: ('Ikarus', 'Trojan-Ransom.WannaCry')
[+] Vendor | Classification: ('Fortinet', 'W32/WannaCryptor.H!tr')
[+] Vendor | Classification: ('AVG', 'Win32:WanaCry-A [Trj]')
[+] Vendor | Classification: ('Panda', 'Trj/RansomCrypt.I')
[+] Vendor | Classification: ('CrowdStrike', 'malicious_confidence_100% (W)')
[+] Vendor | Classification: ('Qihoo-360', 'HEUR/QVM41.2.2698.Malware.Gen')
[+] Hash found at ThreatExpert: No results found
[+] Malicious Indicators from ThreatExpert: No results found
[+] Date found at VXVault: No results found
[+] URL found at VXVault: No results found
[+] Malc0de Date: No results found
[+] Malc0de IP: No results found
[+] Malc0de Country: No results found
[+] Malc0de ASN: No results found
[+] Malc0de ASN Name: No results found
[+] Malc0de MD5: No results found
No results found in the THMD5
```

I've grabbed a WannaCry hash and threw it at Automater, you can see it picked it up right away.

* HTML report of a scan

{% img center /images/tools/automater.png 'automater' 'automater HTML report' %}

**Learn more:**

One of the cool features of Automater is its extensibility. You can add sites to the XML configuration file, and customize it to meet your needs. For further instructions about how to do that, check out the [author's post](http://www.tekdefense.com/news/2013/12/10/the-extensibility-of-automater.html)

``` 
__________________________
< Condense soup, not books! >
 ---------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
