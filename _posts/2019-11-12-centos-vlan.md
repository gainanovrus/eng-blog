---
published: false
layout: single
title: Setup VLAN interface on CentOS
excerpt: >-
  Sometimes you don’t have access to ESXi by WebGUI.
  You might need to restart a server remotely via SSH or direct console.
  In these cases it’s great to be able to turn virtual machines on and off via the command line.
categories: sysad
tags: esxi shell vm
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-10-24
---

There are some scenarios where we want to assign multiple IPs from different
VLAN on the same Ethernet card (nic) on Linux servers (CentOS / RHEL).
This can be done by enabling VLAN tagged interface.

Let’s assume we have a Linux Server, there we have one Ethernet card (`em1`).

So attach VLANs 4 tag to NIC `em1` using the `ip` command
```
ip link add link em1 name vlan4 type vlan id 4
```

To view the VLAN, issue the following command:
```
ip -d link show vlan4
```

Bring up the interface using below ip command:
```
ip link set dev vlan4 up
```

Now we can assign the IP address to tagged interface from their respective VLANs using beneath ip command,
```
ip addr add 172.16.3.188/24 dev vlan4
```

To delete created VLAN interface use next command
```
ip link delete dev vlan4@em1
```

It will work to the next reboot.
For saving the configuration edit the network settings stored in directory `/etc/sysconfig/network-scripts/`.

## Additional information

* [ip command cheat sheet](https://access.redhat.com/sites/default/files/attachments/rh_ip_command_cheatsheet_1214_jcs_print.pdf) -
  For Red Hat Enterprise Linux.
* [Configure 802.1q vlan tagging using the command line](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/sec-configure_802_1q_vlan_tagging_using_the_command_line) -
  Manual from RHEL authors.
* [How to Configure VLAN tagged NIC (Ethernet Card) on Linux Servers](https://www.linuxtechi.com/vlan-tagged-nic-ethernet-card-centos-rhel-servers/)-
  Another instruction to configure VLAN on Linux.
