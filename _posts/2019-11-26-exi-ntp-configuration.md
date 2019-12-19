---
published: true
layout: single
title: ESXi. NTP Configuration
excerpt: >-
  How to configure date and time using NTP on a Vmware ESXi server.
categories: sysad
tags: esxi
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-10-24
---

Would you like to learn how to set date and time using the Vmware ESXi NTP feature?
In this tutorial, I am going to show you how to configure date and time using NTP on a Vmware ESXi server.
This tutorial was tested on Vmware ESXi 6.7

First, you need to access the Vmware web interface.
After a successful login into ESXi go to **Manage > System > Time & date**. Click on **Edit**.

![edit-ntp]({{ site.url }}{{ site.baseurl }}/assets/images/esxi/ntp/1.png){: .align-center}

Select `Start and stop with host` option.

Insert desired NTP servers comma separated, i.e.:
```
0.ru.pool.ntp.org, 1.ru.pool.ntp.org, 2.ru.pool.ntp.org, 3.ru.pool.ntp.org
```

Now, we need to start the NTP service. Click on the **Actions** button. Select the NTP service menu.
Click on the **Start** option.

![start-ntp]({{ site.url }}{{ site.baseurl }}/assets/images/esxi/ntp/2.png){: .align-center}

The NTP will start immediately.

Congratulations! You successfully finished the Vmware ESXi NTP configuration.

## Additional information

* [Tutorial - Vmware ESXi NTP Configuration](https://techexpert.tips/vmware/vmware-esxi-ntp-configuration/)
* [ESXi 6.7 — настройка NTP](https://internet-lab.ru/esxi_6_7_ntp)
