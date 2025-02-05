---
layout: post
title: "T-Mobile Dual eSIM"
date: 2024-02-18 23:10:00 -0500
categories: cellular
tags: cellular tmobile esim ios
---
Get eSIM on your iPhone with two T-Mobile accounts.

With the latest iPhones not including a physical SIM card slot, eSIM is the only option. For those that rock dual sim with two 
T-Mobile accounts (1 physical sim, 1 eSIM), the switch to two eSIMs can be a bit chaotic. After dealing with a very knowledgeable representative, I decided to document the steps below.

1. Call T-Mobile from another phone as you will be shutting off your cell plans in iOS. They will have to use a special system to override 
having two plans on one `EID`.
2. Go to Settings > General > About and note which `IMEI` is used by the physical SIM or is not in use. The section will be labeled with the 
label you created for the physical SIM cell plan. Also note the `EID`.
3. Remove the physical SIM. Then go to Settings > Cellular and tap the line that says no SIM. You will be prompted to update contacts.
4. Turn off the other lines with the toggle next to the phone number
5. Give the T-Mobile Rep the `IMEI` and `EID`
6. After a couple minutes, you should see a pop up in settings to enable the cellular plan. If not, you may have to go to Settings > Cellular >
Add Cellular Plan and scan the QR code below or enter the SM-DP+ Address: `T-MOBILE.GDSB.NET` leaving the Activation code blank and the 
confirmation code blank.
![QR-Code](https://github.com/northportio/T-Mobile-Dual-ESIM/blob/main/eSIM-QR.png?raw=true)
7. After it's enabled, enable the other line and make sure the cellular plan settings are what you want them to be (Ex: Primary Phone Line, Preffered Cellular Data line, etc.)

That's it!