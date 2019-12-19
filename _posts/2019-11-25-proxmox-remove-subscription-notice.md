---
published: true
layout: single
title: Proxmox. Remove subscription notice
excerpt: >-
  Removing madden subscription notice and disabling PVE repository from apt-update list.
categories: sysad
tags: proxmox linux debian vm
toc: false
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
# last_modified_at: 2019-10-24
---

> You do not have a valid subscription for this server. Please visit www.proxmox.com to get a list of available options

If you also like me tired to view this message again and again after login in to
[Proxmox](https://www.proxmox.com/en/) you want to remove it. This message appears from 5.1 version and newer (including 6.0).

To remove it run the command bellow.
You will need to SSH to your Proxmox machine or use the node console through the PVE web interface.

Run the following commands and then clear your browser cache:
```
sed -i.bak "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js

systemctl restart pveproxy.service
```

After that the message will be hided. But for full disable unnecessary Enterprise functions need remove PVE repository from `apt` repo list.

Remove the repository:
```
rm /etc/apt/sources.list.d/pve-enterprise.list
```

Add a repository for free community version (`buster` - for Debian 10 (**Proxmox 6.x**)):
```
echo "deb http://download.proxmox.com/debian/pve buster pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list

wget http://download.proxmox.com/debian/proxmox-ve-release-6.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-6.x.gpg
```

Next update packages and system and reboot PVE.
```
apt-get update && apt-get upgrade -y
apt-get dist-upgrade
reboot
```

## Additional information

* [Remove Subscription Notice](https://johnscs.com/remove-proxmox51-subscription-notice/) -
  There I found the solution to remove the notice in one line
* [Как убрать сообщение о подписке в Proxmox?](https://www.init-d.ru/2018/03/21/proxmox-message/) -
  Second part of this instruction
* [Proxmox, Ceph, ZFS, pfsense и все-все-все](https://forum.netgate.com/topic/120102/proxmox-ceph-zfs-pfsense-%D0%B8-%D0%B2%D1%81%D0%B5-%D0%B2%D1%81%D0%B5-%D0%B2%D1%81%D0%B5) -
  The maximum information about Proxmox on one page
