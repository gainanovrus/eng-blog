---
published: false
layout: single
title: Hacking. Wi-Fi Penetration on MacOS
excerpt: >-
  Today I want to show how to crack WPA/WPA2 passwords on MacOS.
  It took me 20 minutes to hack a password with 8 digits.
  My methods based on airport, aircrack-ng, tcpdump, wireshark, hashcat and some other software.
categories: sysad
tags: python xml sketches
toc: true
header:
  teaser: /assets/images/selenium/selenium_python_logo.png
  og_image: /assets/images/selenium/selenium_python_logo.png
---

> Disclaimer: this post for education purposes only.

hcxpcaptool  capture.cap

hcxpcaptool -z capture.16800 capture.cap

hcxpcaptool -o capture.hccapx capture.cap
reading from capture.cap

summary capture file:                           
---------------------
file name........................: capture.cap
file type........................: pcap 2.4
file hardware information........: unknown
capture device vendor information: 000000
file os information..............: unknown
file application information.....: unknown (no custom options)
network type.....................: DLT_IEEE802_11_RADIO (127)
endianness.......................: little endian
read errors......................: flawless
minimum time stamp...............: 17.07.2020 15:51:18 (GMT)
maximum time stamp...............: 17.07.2020 18:33:03 (GMT)
packets inside...................: 5
skipped damaged packets..........: 0
packets with GPS NMEA data.......: 0
packets with GPS data (JSON old).: 0
packets with FCS.................: 5
beacons (total)..................: 1
EAPOL packets (total)............: 4
EAPOL packets (WPA2).............: 4
best handshakes (total)..........: 1 (ap-less: 0)

summary output file(s):
-----------------------
1 handshake(s) written to capture.hccapx
message pair M32E2...............: 1

[aircrack]: https://www.aircrack-ng.org/
[hashcat]: https://hashcat.net
[wireshark]: https://www.wireshark.org/
[hcxtools]: https://github.com/ZerBea/hcxtools
[jamwifi]: http://macheads101.com/pages/downloads/mac/JamWiFi.app.zip
