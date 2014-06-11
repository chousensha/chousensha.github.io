---
layout: post
title: "Network tools - netstat"
date: 2014-06-11 21:14:59 +0300
comments: true
categories: [networking, netstat]
keywords: networking, netstat
description: Netstat usage examples
---

Netstat is a very important tool for gathering information about the connections on a machine or troubleshooting network problems. It's a default utility for both Windows and Linux, thus there is no excuse in not getting familiar with it, it's very useful for any system admin, network guy or good old home user that would like to know what really is coming and going to and from his computer.
<!-- more -->

If ran with no arguments, netstat produses an overwhelming output of all the open sockets in the system. Here is its *man* page description:

> netstat  - Print network connections, routing tables, interface statistics, 
> masquerade connections, and multicast memberships

Netstat is great when run with specific flags to zoom in on the information that is relevant to us. Here are some examples:

**Display the PID and the name of the program to which eack socket belongs, along with the path**

{% img center /images/netstat-p.png 'netstat -p' 'netstat --program' %}

**Display only listening sockets**

{% img center /images/netstat-l.png 'netstat -l' 'netstat --listening' %}

**Display all ports (both listening and non-listening**

{% img center /images/netstat-a.png 'netstat -a' 'netstat --all' %}

**Display listening sockets and established connections**

{% img center /images/netstat-ap.png 'netstat -ap' 'netstat -ap' %}

**Display TCP ports and connections**

{% img center /images/netstat-at.png 'netstat -at' 'netstat tcp' %}

**Display TCP statistics**

{% img center /images/netstat-st.png 'netstat -st' 'netstat stats' %}

**Display kernel routing tables**

{% img center /images/netstat-r.png 'netstat -r' 'netstat route' %}

**Display all TCP connections and listening ports using numeric values for addresses and ports, instead of resolving their names**

{% img center /images/netstat-antp.png 'netstat -antp' 'netstat antp' %}




> Q:	Why is Poland just like the United States?
> A:	In the United States you can't buy anything for zlotys and in
>	Poland you can't either, while in the U.S. you can get whatever
>	you want for dollars, just as you can in Poland.
>	-- being told in Poland, 1987

