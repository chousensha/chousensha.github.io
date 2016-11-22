---
layout: post
title: "Introduction to Burp Suite"
date: 2016-11-19 17:01:23 -0500
comments: true
categories: [penetration testing, tools]
keywords: burpsuite, burp suite, burpsuite lab, burp suite lab, burpsuite tutorial, burp suite tutorial, burpsuite walkthrough, burp suite walkthrough, burpsuite kali, burp suite kali, burp
description: Burp Suite intro
---


Yes, the time has come for a full post on Burp Suite! I have delayed it for too long!

Before starting, here are some resources for learning about Burp:

[Burp Suite Documentation](https://portswigger.net/burp/help/contents.html)

[Free introductory course on Burp Suite](http://aetherlab.teachable.com/p/burp-suite)

[Burp Suite for Web Application Security](https://vimeo.com/album/3510171)

<!-- more -->

Burp Suite is the primary tool used for performing web application security assessments. It acts as a proxy between your browser and the target, and it comes packed with powerful features to assist in penetration testing: spidering fuctionality, web scanning for vulnerabilities (pro version only), tools that allow you to perform different actions with web requests (will be covered in detail below), and customization ability through plugins.

# Introduction to Burp Suite

Burp is a Java application, so you need to have Java installed (version 1.6 or later) to run it. It comes pre-installed in Kali, where you can find it in the Web Application Analysis category.

To use Burp, you need to configure your browser's proxy settings. Burp's proxy is listening on 127.0.0.1:8080. If you want to manually configure your browser to use it, refer to this [getting started post](https://portswigger.net/burp/help/suite_gettingstarted.html). I recommend using a browser add-on like [FoxyProxy](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/) to set up your proxies and toggle them with a click.

For **Burp to work with HTTPS** requests, you need to add Burp's CA certificate to your browser. Go to http://burp/ and click on CA Certificate to download it to your computer. In this demo I'm using Firefox, but the steps should be similar in the rest of the browsers. In the Firefox menu, select Options or Preferences, then go to Advanced -> Certificates -> View Certificates. Click on the Authorities tab and choose Import to select your CA certificate, and check the option "Trust this CA to identify web sites". Then click Ok, and restart Firefox. Burp should work now with HTTPS traffic, without issuing any security warnings.

# Burp Suite components

The power of Burp comes from the interaction between its components, which you can individually access from the application tabs. Let's see each of them in action!

## Target

Here you can see information about your target. The site map constructs a hierarchical representation of your target. Items requested are shown in black, those that Burp inferred from links etc. are in gray. As you browse with the proxy on, Burp will keep building the site map. 

{% img center /images/tools/burp/sitemap.png 'sitemap' 'target sitemap' %}

You can configure the scope of your targets from the Scope tab. Alternately, you can right-click on the sitemap entries to perform other actions:

{% img center /images/tools/burp/sitemap-menu.png 'sitemap menu' 'target sitemap menu' %}

Clicking on the Filter bar lets you customize filtering options. Before starting the demo, I added my target to the scope and selected to hide the out-of scope items for a clearer view.

{% img center /images/tools/burp/sitemap-filter.png 'sitemap filter' 'target sitemap filter' %}

## Proxy

This is the component that allows you to intercept and modify the requests between your browser and the target. You will get familiar with the Intercept tab, where you can inspect each request and response, modify it, or send it to other tools. The history tabs keep records for the HTTP and WebSockets messages. In the Options tab you have a plethora of configuration options for your Proxy. Take special note of the Response Modification options, which you can use to automatically modify the responses HTML to remove client-side logic and controls, or perform SSL stripping.

{% img center /images/tools/burp/proxy-options.png 'proxy response options' 'response modification options' %}

Right-clicking in the Raw tab will give you more options, among which there are some very useful ones such as copying the request as a Curl command, or constructing selected strings in Javascript and SQL (MySQL, Oracle, MS-SQL)

## Spider

You can use Burp's spider to automatically crawl target applications. After you've manually browsed the application, right-click the host or URL that you want to crawl in the site map, and choose "Spider this host / branch". Then watch the number of requests being made in the Control tab. You can customize the spider in the Options tab. Passive spidering is enabled by default, allowing Burp to update the site map while you are manually browsing. Also, you can specify what the spider should do when encountering forms (ignoring them, automatically submitting with pre-determined values, prompting for guidance etc.) Be careful, as the spider may perform actions with repercussions on the target application, so an initial manual assessment followed by a strict configuration of the spidering scope would be preferred.

## Scanner

Burp has an automatic vulnerability scanner *uncontrolled drooling* that seems excellent from the reviews, but it's only available for Pro users. If you look in the Issue definitions tab, you can see the [vulnerabilites that Burp can detect](https://portswigger.net/KnowledgeBase/Issues/), among with a description for each of them *drooling intensifies*

{% img center /images/tools/burp/scanner.png 'burp scanner' 'burp web scanner' %}

## Intruder

With Burp Intruder you can perform highly-customized automated attacks against your targets, including brute forcing, fuzzing, enumeration etc. Usually, you select an interesting request and send it to Intruder. You can see the positions where payloads will be placed, marked in orange in this dummy request:

{% img center /images/tools/burp/intruder-position.png 'intruder positions' 'intruder payload positions' %}

### Attack types

In the Position tab, you can also choose the type of attack you want to perform:

- **Sniper** - single set of payloads, places each payload into each position in turn, useful for individual parameter fuzzing

- **Battering ram** - single set of payloads, places the the same payload into all positions at the same time

- **Pitchfork** - multiple payload sets, different payload set for each position

- **Cluster bomb** - multiple payload sets, tests all permutations of the payload combinations, so depending on your payloads, this attack might grow to gigantic proportions

### Payload types

You have lots of payload types available for testing, so choose what is appropriate for your target:

- **Simple list** - list of strings

- **Runtime file** - read strings at runtime from a file (for very large lists)

- **Custom iterator** - permutations of characters in multiple lists

- **Character substitution** - substitute characters, for instance when you are testing for 53cur3 p455w0rd5

- **Case modification** - change the characters' case according to the pre-defined rules

- **Recursive grep** - recursively extract data from the responses of previous requests

- **Illegal Unicode** - by using illegal Unicode representations, it might be possible to bypass filters etc.

- **Character blocks** - creates blocks of characters of specified sizes, can aid in detecting buffer overflows

- **Numbers** - numeric payloads

- **Dates** - date values

- **Brute forcer** - sets of permutations generated from specified characters

- **Null payloads** - empty string payloads

- **Character frobber** - changes the value of each character in turn, useful for determining what characters are being used for session tokens etc.

- **Bit flipper** - flips individual bits in the payload

- **Username generator** - generates usernames from the given input

- **ECB block shuffler** - shuffles ECB-encrypted blocks

- **Extension generated** - uses an extension to create the payloads

- **Copy other payload** - copies the current payload value to a different position

## Repeater

This tool is useful for sending requests over and over to the target application.

{% img center /images/tools/burp/repeater.png 'repeater' 'repeater' %}

## Sequencer

With the Sequencer tool you can test for the randomness of data tokens in an target application. The analysis is more accurate if the number of captured tokens is larger.

{% img center /images/tools/burp/sequencer.png 'Burp sequencer' 'Burp sequencer' %}

## Decoder

The Decoder tool performs encoding and decoding of different data formats, such as HTML, URL, Base64, ASCII hex, Hex, Octal, Binary, Gzip, and it also has hashing functionality for MD2, SHA-224, MD5, SHA1, SHA-384, SHA, SHA-512 and SHA-256.

{% img center /images/tools/burp/decoder.png 'Burp decoder' 'Burp decoder' %}

## Comparer

If you want to compare different responses, this tool gives you the options of words or bytes comparison

{% img center /images/tools/burp/comparer.png 'Burp comparer' 'Burp comparer' %}

## Extender

Here you can add extensions to Burp that add more functionality than what is available by default. Take a look at what extensions you can find in the BApp Store:

{% img center /images/tools/burp/extensions.png 'burp extensions' 'burp extensions' %}

In this screenshot I am sorting by rating. There are many extensions available and I suggest looking at each and determining if you will need it before choosing to install. 

### Burp Clickbandit

We're done with the main tools, but Burp has more. It even includes a cool tool for creating clickjacking attacks.

{% img center /images/tools/burp/clickbandit.png 'burp clickbandit' 'burp clickbandit' %}

In a future post I will do a lab featuring Burp's capabilities that I've enumerated so far. Expect some Mutillidae! :-)

``` plain
 _______________________________________
/ Don't you wish you had more energy... \
\ or less ambition?                     /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
