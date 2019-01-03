---
layout: single
title:  "Upgrade kernel of CentOS 7 to latest version"
excerpt: "The Kernel is the brain for an operating system. It's like the core element of every OS. Some popular OSs that use the Linux kernel are Ubuntu, CentOS, and Debian. In this tutorial, I will show you how to upgrade CentOS 7 kernel to the latest version. And in this guide, we will install the latest stable version 4.15.8."
toc: true
categories: linux
tags: centos
header:
  teaser: /assets/images/upgrade-centos-kernel.png
  og_image: /assets/images/upgrade-centos-kernel.png
---
Previously I was writing about [upgrading][upgrade-centos7] system now 
I will show how to upgrade the CentOS7 kernel to the latest version.
Using ELRepo repository we can get easy kernel updates. 
ELRepo is focused on the packages related to hardware, 
including filesystem drivers, graphic drivers, network drivers,
sound card drivers, webcam, and others.

## Instructions

Check the current kernel version. 
```
# uname -sr
Linux 4.11.3-1.el7.elrepo.x86_64
```

If we now go to <www.kernel.org>, we will see that the latest kernel version is 4.15 at the time of this writing 
(other versions are available from the same site).

To enable the ELRepo repository on CentOS 7, do
```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm 
```

Next, install the latest mainline stable kernel
```
yum --enablerepo=elrepo-kernel install kernel-ml
```

I had this output of that command
```
# yum --enablerepo=elrepo-kernel install kernel-ml
Loaded plugins: fastestmirror, langpacks
elrepo-kernel                                                                                                                                                          | 2.9 kB  00:00:00
elrepo-kernel/primary_db                                                                                                                                               | 1.7 MB  00:00:01
Loading mirror speeds from cached hostfile
 * base: mirror.reconn.ru
 * elrepo: nl.mirror.babylon.network
 * elrepo-kernel: nl.mirror.babylon.network
 * epel: mirror.logol.ru
 * extras: mirror.reconn.ru
 * updates: mirror.reconn.ru
Resolving Dependencies
--> Running transaction check
---> Package kernel-ml.x86_64 0:4.15.8-1.el7.elrepo will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================================================================================
 Package                                    Arch                                    Version                                              Repository                                      Size
==============================================================================================================================================================================================
Installing:
 kernel-ml                                  x86_64                                  4.15.8-1.el7.elrepo                                  elrepo-kernel                                   44 M

Transaction Summary
==============================================================================================================================================================================================
Install  1 Package

Total download size: 44 M
Installed size: 198 M
Is this ok [y/d/N]: y
Downloading packages:
kernel-ml-4.15.8-1.el7.elrepo.x86_64.rpm                                                                                                                               |  44 MB  00:00:05
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : kernel-ml-4.15.8-1.el7.elrepo.x86_64                                                                                                                                       1/1
  Verifying  : kernel-ml-4.15.8-1.el7.elrepo.x86_64                                                                                                                                       1/1

Installed:
  kernel-ml.x86_64 0:4.15.8-1.el7.elrepo

Complete!
```

Next, reboot your machine to apply the latest kernel
```
systemctl reboot
```

If you just reboot the system and run command to check the version of your kernel
```
# uname -sr
Linux 4.11.3-1.el7.elrepo.x86_64
```

You will see your current kernel. WTF? But I updated it.
Your system will be updated but to run the system with new kernel need configure Grub loader.

Let's check the current configuration
```
# awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (4.15.8-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (4.11.3-1.el7.elrepo.x86_64) 7 (Core)
2 : CentOS Linux (0-rescue-d39b765b9cf142baa7ed9ca6543375de) 7 (Core)
```

As you can see the new kernel has the second option in the loading process.
We want to use kernel 4.15 as our default, so you can use the following command to make this happen.
```
grub2-set-default 0
```

When you want to revert back to the old kernel, 
you can change the value of the `grub2-set-default` command to 1.

Finally, generate the grub2 config with `gurb2-mkconfig` command, and then reboot the server.
```
grub2-mkconfig -o /boot/grub2/grub.cfg
systemctl reboot
```

After system will load check again the kernel version
```
# uname -sr
Linux 4.15.8-1.el7.elrepo.x86_64
```

Excellent! We have the latest kernel.

Of course, don't do this procedure on a system with users.

## Additional information
* [How to Upgrade Kernel on CentOS 7](https://www.howtoforge.com/tutorial/how-to-upgrade-kernel-in-centos-7-server/)
* [How to Install or Upgrade to Kernel 4.15 in CentOS 7](https://www.tecmint.com/install-upgrade-kernel-version-in-centos-7/)

[upgrade-centos7]: {% post_url 2018-03-11-upgrade-centos %}
