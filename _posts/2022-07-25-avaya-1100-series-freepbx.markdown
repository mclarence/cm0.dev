---
layout: post
title:  "Using Avaya 1100 Series IP phones with FreePBX"
date:   2022-01-06
tags: voip freepbx asterisk ip phones nortel avaya 1100 1120e 1140e series
comments: true
toc: true
published: true
description: "This post describes how to use Avaya 1100 Series IP phones with FreePBX."
---

I had a few Avaya/Nortel 1120e/1140e IP phones laying around at home and decided to create a house phone and intercom system. Setting these phones up with FreePBX/Astericks was a chore and after many hours of trial and error and google searching, i've managed to get these working pretty well with FreePBX. 

# Prerequisites
---
* Advanced technical knowledge. This guide is not intended for beginners.
* A working FreePBX/Asterisks server.
* A TFTP server.
* (Optional) A configurable DHCP server.

# Converting Phones from UNISTIM to SIP
---
If you have one of these phones, there is a good chance that it is running the old UNISTIM firmware. You will need to flash the SIP firmware to the phone in order to get it to work with FreePBX.

In order to do this easily, a TFTP server should be used. [TFTPD64](https://pjo2.github.io/tftpd64/) is an example of one to use. Then configure your DHCP server to include the TFTP's server IP address in DHCP option 66. If your DHCP server does not allow you to do this, then use TFTPD64's built in DHCP server. 

Next download the latest firmware for your phone's model. The latest versions are always available on [Avaya's website](https://support.avaya.com/products/P0599/1100-series-ip-deskphones/). Place the downloaded `.bin` file in the TFTP server's root directory.

In the TFTP's root directory, you will then need to create the following files depending on your phone model.

* `1120e.cfg`
* `1120eSIP.cfg`
* `1140e.cfg`
* `1140eSIP.cfg`

It is important that they are named according to your phone's model as shown as above. The phone will be looking for these files as soon as it connects to the TFTP server.

In each `.cfg` file above, include the following contents at the top:

```
[FW]
DOWNLOAD_MODE FORCED
VERSION <firmware file name>
FILENAME <firmware file name>
PROTOCOL TFTP
SERVER_IP <tftp server ip>
SECURITY_MODE 0
```
* Replace `<firmware file name>` with the file name of the firmware you downloaded from Avaya. Ensure that the firmware file is correct for the model you are upgrading.
* Replace `<tftp server ip>` with the IP address of your TFTP server.

Now power up the phone whilst connected to the network and the phone should begin to upgrade its firmware. When the phone powers up, it will obtain the TFTP server automatically from the DHCP server. Then the following will happen:
1. Read the appropriate firmware file for the phone's model. For example, a 1120e on the UNISTIM firmware will read the `1120e.cfg` file or a 1120e on the SIP firmware will read the `1120eSIP.cfg` file. 
2. The phone then downloads the firmware file specified in the configuration file.
3. The phone will write the firmware to its memory and reboot.
4. If the phone was previously on the UNISTIM firmware and now on the SIP firmware, it will now read configuration files ending in SIP.

## Error: Authentication Failed
If you receive an error `[FW] Authentication Failed` this usually means that you are upgrading the firmware of the phone that is too far ahead. In this case you will need to incrementally upgrade the phone up until the latest the version.

You may also try using the phone's BootC mode which is essentialy its recovery mode. I've found that you can usually bypass this error using this mode. When the phone immediately powers on, hold the Up + 2 key until you see `Starting DHCP...` and it will contact the TFTP server to grab the firmware files. 

If in BootC mode, the phone continues to boot normally after `Starting DHCP...` is shown, you will need to manually input the TFTP server. Again hold Up + 2 but as soon as the phone's lights turn off, immediately press the four soft keys below the screen, from left to right in quick succession. If done correctly, you should see a manual configuration prompt. This mode will allow you to input your TFTP's server IP address. Use the phone's buttons to navigate through this menu. After you have applied the settings, the phone should reboot and upgrade the firmware.

# Setting up FreePBX/Asterisk
---
Once you have upgraded the firmware to SIP, you can now configure FreePBX. In the FreePBX GUI, create a **CHAN_SIP** extension. It is important that it is on CHAN_SIP as using CHAN_PJSIP will cause the phones to loose SIP registration to the PBX after some time. Take note that when using CHAN_SIP extensions, the SIP port will be 5160/udp instead of the standard 5060/udp port.

# Creating Provisoning Files
---
We will now create provisoning files for the phones so that they can connect to the FreePBX server. This can also be done manually by inputting the SIP domain and authentication credentials manually on the phone itself.

Using the same TFTP server, open your device's model configuration file. So if you have a 1120e phone, open the `1120eSIP.cfg` file. Below the `[FW]` section, add the following section below it:

```
[DEVICE_CONFIG]
DOWNLOAD_MODE FORCED
FILENAME device.cfg
VERSION 000001
PROTOCOL TFTP

[USER_CONFIG]
DOWNLOAD_MODE FORCED
VERSION 000001
PROTOCOL TFTP
```

Ensure that `[DEVICE_CONFIG]` is followed by `[USER_CONFIG]` as the phone reads these files from top to bottom and will overwrite settings if the sections are out of order.

Next create a `device.cfg` file in the same directory on the TFTP server. In that file, put the following contents:

```
DNS_DOMAIN <dns domain>
SIP_DOMAIN1 <sip domain>
SERVER_IP1_1 <sip domain>
SERVER_PORT1_1 5160
SERVER_RETRIES1 3

FORCE_BANNER YES
BANNER THIS IS A BANNER

VMAIL *97
VMAIL_DELAY 300
MAX_APPEARANCE 1
DEF_LANG English
DEF_AUDIO_QUALITY High
LLDP_ENABLE YES
ADMIN_PASSWORD 1234
ADMIN_PASSWORD_EXPIRY 0
DEF_AUDIO_QUALITY High

TIMEZONE_OFFSET <time zone>
FORCE_TIME_ZONE YES

# Disable extended features
MAX_LOGINS 1
USB_HEADSET LOCK
EXP_MODULE_ENABLE NO
ENABLE_SERVICE_PACKAGE NO
IM_MODE DISABLED
AVAYA_AUTOMATIC_QoS NO
VQMON_PUBLISH NO
SIP_TLS_PORT 0
ENABLE_BT NO
```
Replace...
* `<dns domain>` with the DNS domain name of your PBX. This can also be set to the IP address of the PBX server.
* `<sip domain>` with the SIP domain name of your PBX. This can also be set to the IP address of the PBX server.
* `<time zone>` with the time zone offset. The offsets are:

    |Location| Time zone offset (seconds)|
    |-------|----------------------------|
    |(GMT-04:00) Atlantic Standard Time |-14400|
    |(GMT-03:30) Newfoundland | -12600|
    |(GMT-03:00) Buenos Aires | -10800|
    |(GMT-02:30) Newfoundland DST | -9000|
    |(GMT-01:00) Azores | -3600|
    |(GMT+00:00) Greenwich, Dublin, Lisbon, London |0|
    |(GMT+01:00) Central European Time |3600|
    |(GMT+02:00) Athens |7200|
    |(GMT+03:00) Moscow |10800|
    |(GMT+03:30) Tehran| 12600|
    |(GMT+04:00) Abu Dhabi |14400|
    |(GMT+04:30) Khabul| 16200|
    |(GMT+05:00) Islamabad |18000|
    |(GMT+05:30) Indian Standard Time |19800|
    |(GMT+06:00) Sri Lanka |21600|
    |(GMT+06:30) Myanmar| 23400|
    |(GMT+07:00) Bangkok |25200|
    |(GMT+08:00) China Standard Time |28800|
    |(GMT+09:00) Japan Standard Time | 32400|
    |(GMT+09:30) Australian Central Standard Time |34200|
    |(GMT+10:00) Australian Eastern Standard Time |36000|
    |(GMT+11:00) Micronesia | 39600|
    |(GMT+12:00) Fiji | 43200|
    |(GMT+13:00) New Zealand  | 46800|

If you do not have a license for these phones, ensure that everything below *Disable extended features* is included, otherwise you will not be able to login to the phone.

Now you will need to get the MAC address of the phone. This can be found at the back of the phone. Once you have the MAC address, create a file called `SIP<MAC>.cfg` replacing `<MAC>` with the phone's MAC address. In that file, put the following contents:

```
AUTOLOGIN_ENABLE USE_AUTOLOGIN_ID
AUTOLOGIN_ID_KEY01 <sip username>@<sip domain>
AUTOLOGIN_AUTHID_KEY01 <sip username>
AUTOLOGIN_PASSWD_KEY01 <sip password>
```
Replace...
* `<sip username>` with the SIP username of your desired extension.
* `<sip domain>` with the SIP domain name of your PBX that you configured in `device.cfg`
* `<sip password>` with the SIP password of your desired extension.

Once you are done, reboot the phone and it should grab the configuration files from the TFTP server and connect to your FreePBX server.

# Configuring other Phone Features.
---
There are a lot of paramters that can be configured. Refer to the [SIP Administration Guide](/assets/files/18125156.pdf) from Avaya for more information.

## Configuring Intercom/Paging Features
To get intercom/paging working on FreePBX, you will need to set `ENABLE_ANSWER_MODE` to YES in the device.cfg file. Unfortunately, enabling this will require a license for the phone. If you do not have a license and you enable this, you will be locked out of the phone until it is disabled and reset. After you have enabled this, go into the phone's preferences (press more soft key until you see *prefs*) -> Feature Options -> Answer Mode Settings and set *Allow Mode* to public and in *Allow Addresses* input the SIP addresses to allow intercom/paging from.

Then on FreePBX ssh into it and edit `/etc/asterisk/extensions_override_freepbx.conf` with the following:
```
[autoanswer]
include => autoanswer-custom
exten => s,1,GosubIf($["${ARG1}" != ""]?func-set-sipheader,s,1(Alert-Info,${ARG1}))
exten => s,n,GosubIf($["${ARG2}" != ""]?func-set-sipheader,s,1(Call-Info,${ARG2}))
exten => s,n,Gosub(func-set-sipheader,s,1(Answer-Mode,Auto))
exten => s,n,Gosub(func-apply-sipheaders,s,1())
exten => s,n,Return()
```
What this is doing is adding an additional SIP header to the SIP INVITE packet so that the phone will automatically answer the call. This header is the `Answer-Mode` header. More information about this can be found from [RFC 5373](https://datatracker.ietf.org/doc/html/rfc5373#section-4.3.3)

## Resetting the Phone
To reset the phone you will need to know the mac address of the phone. This can be found at the back of the phone. Enter the following key combination to reset the phone:

`**[7][3][6][3][9][MAC][#][#]`

If your MAC address has letters, use the corresponding number to represent the letters. For example if you have an A in the MAC address, you will need to use the number 2. If you have a F in the MAC address, you will need to use the number 3 and so on.