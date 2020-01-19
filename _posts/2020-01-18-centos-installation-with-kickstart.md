---
published: true
layout: single
title: CentOS 8. Automated installation using Kickstart
excerpt: >-
  You can install CentOS 8 automatically using a Kickstart file.
  The file has the answer to all the questions that installer asks when you manually install it.
  In this article I will show and explain my file to install CentOS 8.
categories: linux
tags: centos kickstart linux
toc: true
header:
  teaser: /assets/images/centos-installation-with-kickstart.png
  og_image: /assets/images/centos-installation-with-kickstart.png
---

First of all if you have never seen a kickstart file before and you have installed
a flavor of Redhat Linux on a system go look in the `/root` dir you should see a
file called `anaconda-ks.cfg` open it up and you will see the [parameters][ks-parameters] you
entered during your install in the kickstart file.

It is a good way to understand by example (providing you can remember the options you selected at boot time).

Below I will give you an example of a kickstart file I use it in my work with remote servers.
I chose to use a kickstart install with scripts over imaging software for the
Linux installs as this enabled me to use the image on various types of hardware and
with the tweak of a script I could greatly customize the installs in the future.
Lastly I change the configuration (update packages, settings) using [Ansible playbooks][ansible-post] after I've install the system.

## Define the Kickstart file

<script src="https://gist.github.com/GRomR1/b567d0e295b7af050d4e518f54694c12.js"></script>

With [this][my-ks-gist] kickstart file, we will do the following actions:
* Define installation source and additional repos (minimal and standart)
* Define Root Password (see [here][root-pw-sf] and [here][root-pw-rh] to manual create crypted password)
* Install the OS from entirely from the network
* Accept de EULA
* Establish the timezone and enable NTP time synchronization on the host
* Format all drives
* Auto-partition the disk (two parts):
  * /boot part - format ext4 part with 1024MiB size
  * LVM part on full space of disk with next LV:
    * / - 50000 MiB
    * /tmp - 30000 MiB
    * /var/log - 10000 MiB
    * free space - 10000 MiB
* Remove swap part
* Disable kdump
* Skip X Windows packages
* Activate first network device that linked up

Also I like to add post section with commands that allow remote access to the new system using ssh key file:
```
%post
/bin/mkdir /root/.ssh
/bin/chmod 700 /root/.ssh
/bin/echo -e 'ssh-rsa AAAAB3Nza.....9WhQ== admin@example.com' > /root/.ssh/authorized_keys
/bin/chown -R root:root /root/.ssh
/bin/chmod 0400 /root/.ssh/*

%end
```

## Starting the Kickstart Installation

Installing CentOS using kickstart is a 3 step process

1. Create the kickstart file (if you don't already have one)
2. Make the kickstart file available to the boot process (e.g. put kickstart file on a web server, e.g. in [github](https://gist.github.com/) like [me][my-ks-raw])
3. Access the boot prompt during the Centos installation and then tell the boot prompt where your kickstart file is located.
 Specified by the initrd boot parameter `inst.ks=<YOUR_KS_CFG_DESTINATION>` where `<YOUR_KS_CFG_DESTINATION>` should be change to real destination.
 Press 'Tab' button on [first boot option][boot-options] to modify initrd boot parameter.

Ultimately you can modify the ISO file and the `isolinux/isolinux.cfg` and make it autostart and everything,
but for this post it just wasn’t the right approach. Also you can run PXE server to automate full installation.

![destination-window-with-raid-controller]({{ site.url }}{{ site.baseurl }}/assets/images/dell-centos/destination-with-raid.png){: .align-center}

That’s all! Automatic Kickstart installations offer a great deal of benefits for system administrators in environments that they have to perform system installations on multiple machines the same time, in a short period of time, without the need to manually interfere with the installation process.

## Additional information

* [RedHat - Kickstart Syntax Reference](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax) - full list of Kickstart commands and options for RHEL7 (work on RHEL8 also)
* [Centos - Kickstart Installations](https://docs.centos.org/en-US/centos/install-guide/Kickstart2/) - an instruction on Centos site
* [How to Use Kickstart to Install CentOS 7](https://linuxhint.com/install-centos-7-kickstart/) - useful article describes a using of Kickstart Configurator app to manual create Kickstart file
* [Kickstart CentOS 7 installation](https://shawnliu.me/post/kickstart-centos-7-installation/) - good material with information about creation custom ISO file of CentOS
* [Automated Installations of Multiple RHEL/CentOS 7 Distributions using PXE Server and Kickstart Files](https://www.tecmint.com/multiple-centos-installations-using-kickstart/) - some instruction about installation RHEL with Kickstart and PXE
* [https://github.com/CentOS/Community-Kickstarts](https://github.com/CentOS/Community-Kickstarts) - a pack with many different kickstart file examples by CentOS Community

[boot-options]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/chap-anaconda-boot-options#sect-boot-options-installer
[ks-parameters]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/installation_guide/sect-kickstart-syntax
[root-pw-rh]: https://access.redhat.com/solutions/44662
[root-pw-sf]: https://serverfault.com/questions/588532/anaconda-kickstart-and-rootpw-option
[my-ks-raw]: https://gist.githubusercontent.com/GRomR1/b567d0e295b7af050d4e518f54694c12/raw/a9f7f7db1cf1691e289169b573109c608c57a882/ks.cfg
[my-ks-gist]: https://gist.github.com/GRomR1/b567d0e295b7af050d4e518f54694c12
[ansible-post]: {% post_url 2019-01-07-ansible-why-i-use-it %}
