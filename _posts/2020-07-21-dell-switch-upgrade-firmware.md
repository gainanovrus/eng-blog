---
published: true
layout: single
title: DELL. Upgrade firmware on Dell S4048 switch (S-series, OS9)
excerpt: >-
  Instructions for upgrading the last firmware of Dell Networking system.
categories: sysad
tags: dell networks firmware
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
---

Today I would like install the last version of the Dell networking OS9 system.
Follow to my guide I tested on my Dell PowerSwitch [S4048-ON][s4048-specs].

## Prerequisites

I already configure management interface and ssh server. Read my [previous][ssh-server] post if you didn't make it.

## Download the latest firmware

Open the browser. Enter to the site [force10networks](https://www.force10networks.com/). It requires an account.
Register the user. Fill the Register form and put `DELL` as a Force10 contact. You will receive after 2 or 3 days your credential.

Follow to the [link](https://www.force10networks.com/CSPortal20/Software/SSeriesDownloads.aspx).
The Software Center page will be open. Try to find by switch name - `S4048-ON`.

In the head of table you see the last version of firmware. Download the file with name - [`FTOS-SK-9.14.2.7.bin`](https://www.force10networks.com/CSPortal20/Software/software/FTOS-SK-9.14.2.7.bin). The file `ONIE-FTOS-SK-xxx` need to update system by ONIE loader. We will upgrading system with easy way.

[![last-firmware]({{ site.url }}{{ site.baseurl }}/assets/images/dell-switch/dell-switch-last-firmware.png)]({{ site.url }}{{ site.baseurl }}/assets/images/dell-switch/dell-switch-last-firmware.png){: .align-center}

Also download and carefully read the [Release Notes](https://www.force10networks.com/CSPortal20/Software/documentation/S4048-ON-9.14.2.7-RN.pdf).

## Check the current system

The commands below will show a useful information about your device:
- `show os-version`
- `show system stack-unit 1`
- `show revision`
- `show version`
- `show boot sys stack-unit all`

I recommend to run them before upgrade the firmware. It could help to prevent any errors if it will be.

For example my output of the command `show version`:

```
DellEMC# show version

Dell EMC Real Time Operating System Software
Dell EMC Operating System Version:  2.0
Dell EMC Application Software Version:  9.13(0.1)
Copyright (c) 1999-2018 by Dell Inc. All Rights Reserved.
Build Time: Tue Feb 27 09:27:11 2018
Build Path: /build/build05/SW/SRC
Dell EMC Networking OS uptime is 1 week(s), 1 day(s), 10 hour(s), 59 minute(s)

System image file is "system://A"

System Type: S4048-ON
Control Processor: Intel Rangeley with 3 Gbytes (3201302528 bytes) of memory, core(s) 2.

8G bytes of boot flash memory.

  1 54-port TE/FG (SK-ON)
 48 Ten GigabitEthernet/IEEE 802.3 interface(s)
  6 Forty GigabitEthernet/IEEE 802.3 interface(s)
```

So my current version of system - `9.13(0.1)`. And I will upgrade to `9.14(2.7)`

## Backup configuration

Dell EMC Networking recommends that you back up your startup configuration and any important files and directories to an external media prior to upgrading the system.

Run the copy command to save running-config to some network host by SSH:

```
copy running-config scp:
```

## Upgrading

Follow these steps carefully to upgrade your S4048-ON systems.

* Copy Firmware from my Mac to flash memory on device

```
DellEMC# copy scp://admin:some_strong_password@192.168.101.3 flash://FTOS-SK-9.14.2.7.bin

Source file name []: /Users/admin/Documents/dell/firmware/FTOS-SK-9.14.2.7.bin
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
72581613 bytes successfully copied
```

* Verify a hash sum (it should be the same as presented in Download page)

```
DellEMC# verify md5 FTOS-SK-9.14.2.7.bin
MD5 hash for FTOS-SK-9.14.2.7.bin : ae09a0042db441291b119708b890bb8c
```

* Upgrade the Dell EMC Networking OS in flash partition `A:` or `B:`

```
DellEMC# upgrade system flash://FTOS-SK-9.14.2.7.bin A:

!................................................!
72581613 bytes successfully copied
System image upgrade completed successfully.
```

* Verify that the Dell EMC Networking OS has been upgraded correctly in the upgraded flash partition

```
DellEMC# show boot system stack-unit all

Current system image information in the system:
===============================================

Type          Boot Type       A                                  B
----------------------------------------------------------------------------------------------
stack-unit 1  FLASH BOOT      9.14(2.7)[boot]                    9.13(0.1)                         
stack-unit 2 is not present.
stack-unit 3 is not present.
stack-unit 4 is not present.
stack-unit 5 is not present.
stack-unit 6 is not present.
```

> NOTE: If your boot flash partition is different then `A:`
> You should change the Primary Boot Parameter - `DellEMC(conf)# boot system stack-unit 1 primary system: A:`

* Save the configuration so that the configuration will be retained after a reload using write memory command.

```
DellEMC# write memory
```

* Reload the unit

```
DellEMC# reload

Proceed with reload [confirm yes/no]: yes
```

## Verification

Verify the switch has been upgraded to the Dell EMC Networking OS version 9.14(2.7)

```
DellEMC# show version

Dell EMC Real Time Operating System Software
Dell EMC Operating System Version:  2.0
Dell EMC Application Software Version:  9.14(2.7)
Copyright (c) 1999-2019 by Dell Inc. All Rights Reserved.
Build Time: Tue Jun 23 09:25:55 2020
Build Path: /build/build02/SW/SRC
Dell EMC Networking OS uptime is 12 hour(s), 53 minute(s)

System image file is "system://A"

System Type: S4048-ON
Control Processor: Intel Rangeley with 3 Gbytes (3201302528 bytes) of memory, core(s) 2.

8G bytes of boot flash memory.

  1 54-port TE/FG (SK-ON)
 48 Ten GigabitEthernet/IEEE 802.3 interface(s)
  6 Forty GigabitEthernet/IEEE 802.3 interface(s)
```


## Conclusion

I spend more time to understand how to upgrade the system then the switch is upgrading.
I'm hope this article could save a time you. Have a nice flashing.

## Additional information

* [Software Center for Force10 devises](https://www.force10networks.com/CSPortal20/Software/SSeriesDownloads.aspx) - site where the firmware for Dell devices placed
* [Dell Command Line Reference Guide for the S4048–ON System](https://www.dell.com/support/manuals/us/en/19/force10-s4048-on/s4048-on-9.14.0.0-cli/about-this-guide) - dell man page included full command list



[s4048-specs]: https://i.dell.com/sites/doccontent/shared-content/data-sheets/en/Documents/Dell-EMC-Networking-S4048-ON-Spec-Sheet.pdf
[ssh-server]: {% post_url 2020-07-19-dell-ssh-server-configure %}
