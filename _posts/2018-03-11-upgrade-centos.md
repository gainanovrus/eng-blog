---
layout: single
title:  "Upgrade CentOS 7.x to latest version"
excerpt: "The latest version of a system has latest bug fixes and new opportunities and in this article will be showed how to update CentOS 7"
toc: true
categories: linux
tags: centos
header:
  teaser: /assets/images/upgrade-centos.jpg
  og_image: /assets/images/upgrade-centos-min.jpg
---
Usually, you want to use the last version of the operating system and if it is true
you will update your OS early or later. In this article, you can found a simple and small instruction to update Centos 7 from 7.x to the latest version (now it is 7.4)
See CentOS 7.4 [release note][redhat-release] for more information about changes.

## Instructions

Check the current version
```
# cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
```

Upgrading is the same as the update option with `-obsoletes` flag.
Before upgrading purge all old cache and download a new one.
And next, you can upgrade OS.
```
yum clean all
yum makecache
yum -y upgrade
```

Below is described info that I got (I left only main info)
```
# yum clean all
...
Cleaning up everything
Cleaning up list of fastest mirrors

# yum makecache
...
Metadata Cache Created

# yum -y upgrade
...
Complete!
```

After the upgrade process has completed seeing changes reboot a system
```
systemctl reboot
```

And repeat to check the current version of the system
```
# cat /etc/redhat-release
CentOS Linux release 7.4.1708 (Core)
```

Of course, don't do this procedure on a system with users.

## Additional information
* [Upgrading from CentOS 7.3 to 7.4](http://blog.dailystuff.nl/2017/10/upgrading-from-centos-7-3-to-7-4/)
* [How to Update CentOS 7.0/7.1/7.2/7.3 to CentOS 7.4](https://www.itzgeek.com/how-tos/linux/centos-how-tos/how-to-update-centos-7-07-17-2-to-centos-7-3.html)

[redhat-release]:https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7
