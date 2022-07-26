---
layout: post
title:  "Using Avaya 1100 Series IP phones with FreePBX"
date:   2022-01-06
tags: voip freepbx asterisk ip phones nortel avaya 1100 1120e 1140e series
comments: true
toc: true
published: false
---

I had a few Avaya/Nortel 1120e/1140e IP phones laying around at home and decided to create a house phone and intercom system. Setting these phones up with FreePBX/Astericks was a chore and after many hours of trial and error and google searching, i've managed to get these working pretty well with FreePBX. 

# Converting Phones from UNISTIM to SIP
---
If you have one of these phones, there is a good chance that it is running the old UNISTIM firmware. You will need to flash the SIP firmware to the phone in order to get it to work with FreePBX.

To do this, you will need to setup a TFTP provisioning server. Any TFTP server will do such as [TFTPD64](https://pjo2.github.io/tftpd64/). You will need to point the TFTP server to an empty directory. This directory will contain all the provisoning and configuration files for the phone. 

You will also need to change your DHCP server settings to point to the TFTP server's IP address, more specifically option 66. Most consumer routers will not have this function so I recommend that you use TFTPD64's DHCP server temporarily whilst setting up the phones. Since I was using windows server's DHCP server for my home network, this was done pretty easily without the need of using another DHCP server.