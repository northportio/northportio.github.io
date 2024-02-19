---
layout: post
title: "T-Mobile Static IPv4 APN Setup"
date: 2024-02-18 20:00:00 -0500
categories: Cellular Service
tags: cellular,t-mobile,static ip,apn,ios,ipados,router
---


# T-Mobile Static IPv4 APN Setup
Getting a static IPv4 address can be a great thing. You can open up services on your firewall to communicate with your 
specific IPv4 addresses in situations where a VPN will not cut it like VoIP/SIP. You can also forward ports to
a cellular router ([GL-iNet GL-X750](https://www.gl-inet.com/products/gl-x750/)). 

## But first, there are a few prerequisites:

1. You have to have a business account with T-Mobile
  - ***For those with personal accounts, stop here. Don't waste your time because this won't work and you may loose cellular data access.
Following these steps will NOT get you a static IP!***
  - For those with business accounts, please continue:
2. You need to discuss this with your account representative. Explain the reasons why you need a static IP. They will likely have to consult an SE
(Solutions Engineer). You may have to speak to the SE yourself.
3. Your account representative will add the static ip add-on to your line
- ***BEFORE THIS HAPPENS, UNDERSTAND THE FOLLOWING:***
  - Once the static ip add-on is added to the line, cellular data will stop working
  - You must add the APN information immediately to regain cellular data access under your new IPv4 address
 4. Add the APN information
 - The original APN for personal and default business lines is `fast.t-mobile.com`
 - The APN to get a Static IPv4 address is `b2b.static`
 - On iOS, you don't normally see the option to input an APN so you will have to create a profile using Apple Configurator 2 with the configuration below or install one from [here](https://github.com/northportio/T-Mobile-Static-IPv4/releases/tag/latest)
 
 ![APN Config](https://github.com/northportio/T-Mobile-Static-IPv4/blob/main/APN/apnconfig.png)
 
 5. Check your IP
 - You can google `what is my ip` or go to a reliable site such as [ifconfig.tv](https://ifconfig.tv) and your new IPv4 address should be displayed
 
 That's it