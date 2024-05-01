---
layout: post
title: "Comma.ai OpenPilot with a 2023 Ford Escape"
date: 2024-04-30 21:00:00 -0500
categories: self-driving
tags: self-driving ai ford adas
---

# Enabling Autonomous Driving With The Comma 3X for a 2023 Ford Escape
Being a technology entrepreneur means that technology is not just your job but also your passion. For a personal project, I decided to install the Comma 3X on my 2023 Ford Escape Hybrid from [Comma.ai](comma.ai).

Many cars today have some capability of autonomous driving. But they are either restricted by a paywall or a room full of lawyers. Some automotive manufacturers go about self driving by trying to define every scenario. As an analogy, One way is two define what a chair is (flat eleveted surface with 4 legs and a vertical back a 90º). Another way to go about it is to give the computer thousands of images of chairs and it will learn based off of that. Ford's Blue Cruise (which is only on certain high trim models) only works on previously mapped roads by the manufacturer. But that means if a new highway opens up, you have to wait for Ford to map that road. With the Comma 3X, there is no wait. 

All of these systems Are advanced drivers assist system (ADAS). Just because it is from a legacy automotive manufacturer, doesn't mean that their system is great. Believe it or not, in 2020, [consumer reports rated](https://data.consumerreports.org/wp-content/uploads/2020/11/consumer-reports-active-driving-assistance-systems-november-16-2020.pdf) all of these systems including Comma Ai's Comma Two (which is two generations old now) and, came out as the leader.

Comma Ai also opensoursed their openpilot software. There's something about not relying on software made by a mega corporation. But the true benefit is being able to contribute and add support for your vehicle And not just waiting for when it is profitable as many manufacturers don't focus on making their previous models better, it's all about the new models.

Believe it or not many vehicles are capable of using the Comma 3X to enable some level of autonomous driving. So far there's over 250 supported vehicles. If your car has adaptive cruise control, and lane centering, There's a good chance that it may work. Be sure to check with Comma Ai for compatibility with your vehicle.

In my case, my car was not fully supported, but through the advice of some very knowledgeable members of The Comma discord community and in-depth research myself, I got it to mostly work well.

## What did the physical install look like? Quite Involved (2-3 Hours)

This install is not for the faint of heart. It is time consuming and a little risky (due to having to disconnect airbag seatbelt tensioner wire which is the yellow plug in the second photo below). But the risks can be mitigated (disconnecting the negative terminal whenever you are working with anything electrical or touching the airbag system). If you are handy, know your way around cars/electrical, and willing to put in the effort, it is very doable.

First off, the [#ford](https://discord.com/channels/469524606043160576/539096103468007424) channel in the [comma.ai discord server](https://discord.comma.ai/) is very helpful so if you haven't joined it already, do that now. Also, it is very necessary to get a 72 hour access to the service manual as you will need to remove lots of trim panel pieces. 

### 🚨 You will need to temporarily remove the rear seatbelt (which is integrated into the airbag system so best practice is to remove power from the car during the install) 🚨 
Remove the 13mm bolt that is grounded to the chassis to the left (**_DO NOT REMOVE THE SCREW FROM THE TERMINAL... THE TERMINAL BOLT WILL EASILY BREAK and a replacement is ~$150 USD_**)
![Battery Terminal](https://media.discordapp.net/attachments/539096103468007424/1225198432315117679/SCR-20240403-pgmk.png?ex=6632b6f5&is=66316575&hm=f1ebf28aa727773ac0082b6a89b6a4d62dbf9f47d42a0e8a8f819f72c943cb5a&=&format=webp&quality=lossless&width=1936&height=1240)

For 2023, Ford relocated the Image Processing Module (IPMA) from the rear view mirror trim area to just under the right trunk window... behind the trim. The below image includes the Q4 harness:
![23 Escape IPMA Location](https://media.discordapp.net/attachments/539096103468007424/1225198433061699674/IMG_0466.jpeg?ex=6632b6f5&is=66316575&hm=4048f0ff3781a3e6c8eaad89512bcca329366f98e73f82d71919c08c7e1e562c&=&format=webp&width=1762&height=1322)
As a result of this, third-party parts are needed to install with the increased distance (These are _**NOT**_ affiliate links):

- [Monoprice SlimRun Cat6 20ft](https://a.co/d/1rpjkyc): The ethernet cable that comes included is not long enough. I use these cables all the time at work and selected this one specifically because the slim cable is easy to route behind the bottom door-sill trim panel pieces from the OBD2 port to the harness in the trunk area.
- [USB-C Extension](https://a.co/d/cuiEBux): The USB-C cable that comes included is not long enough. To get the USB-C connection to the 3X from the harness in the trunk area, you will need an extender, good news is that the distance is perfect if you are running along the headliner to the rear view mirror. Once run, use the right-angle cable that came in the box to plug into the 3X.

You will need tools:
- Trim Panel Removal Tools (never use metal to pry off a trim panel piece, make sure they're plastic)
- 7mm Socket 
- 10mm Socket
- 13mm Socket 
- T25 Torx Screwdriver
- Gloves (Trunk weather seal has "goo")

Trim Panels removed/moved slightly:
- Driver-Side lower door-sill trim: To tuck the cat6 cable
- Trunk bottom trim pieces: To access IPMA
- Right Lower Loadspace Trim: To access IPMA
- Right Upper Loadspace Trim: To access IPMA & run USB-C to headliner
- B-Pillar Trim: To run USB-C behind headliner
- Rear-View Mirror Trim: To run USB-C cable to 3X on windshield

![Right Lower Loadspace Trim](https://media.discordapp.net/attachments/539096103468007424/1225198431568527511/SCR-20240403-pein.jpeg?ex=6632b6f5&is=66316575&hm=b5b4c12db907b16655da1c74cb69c12dbb4ec8ed6637473b70a427d0cc8e79e7&=&format=webp&width=1988&height=1322)

For more on the install, see [this discord thread](https://discord.com/channels/469524606043160576/539096103468007424/1225198431312679126)

## Software Work
As I mentioned, The 2023 Ford Escape is not fully supported yet, so some software work is needed. Below is some info about the changes I made to enable openpilot to work

## Q&A/Info about this [Pull Request](https://github.com/commaai/openpilot/pull/32319)

### Why did I create a new model rather than just adding the fingerprint to the MK4 Escape?
Although the 2023 Ford escape is still considered generation 4, the refresh was the complete overhaul of the systems electronics. _**It is now using CAN-FD rather than CAN**_ unlike the 2020-2022 (see [this wikipedia entry](https://en.wikipedia.org/wiki/Ford_Escape#2023_refresh)). 

### Other than fingerprints and values, what else did I use to get this to work?

1. **_Change The [Panda](https://github.com/commaai/panda) Branch_**: The stock panda sub module does not allow the 3X to recognize the car. In order for the 3X recognize the car, I had to change the panda sub module to the branch [`isotp-canfd-support`](https://github.com/commaai/panda/tree/isotp-canfd-support). See https://github.com/commaai/panda/pull/1524 for more info on this.
2. _**Comment out `dashcamOnly` from `selfdrive/car/ford/interface.py`**_: Currently all Q4 harnesses are not supported from my understanding. Commenting out this reference allows stock openpilot to see the car.

## What's next?

1. **_Testing, Testing, Testing_**: More testing is needed as I've noticed some "Controls Lagging" & "Controls Mismatch" Errors. Nothing that I found dangerous but annoying and usually require a reboot of the device
2. **_Collaboration_**: I will admit that I am not a CAN network expert and not a well-versed python dev so please feel free to pick my brain, request logs, & send code snippets to test. 

**Comma Car Port Checklist**

- [ ] added entry to CAR in selfdrive/car/*/values.py and ran `selfdrive/car/docs.py` to generate new docs
- [x] test route added to [routes.py](https://github.com/commaai/openpilot/blob/master/selfdrive/car/tests/routes.py)
- [ ] route with openpilot:
- [ ] route with stock system:
- [x] car harness used: Ford Q4
