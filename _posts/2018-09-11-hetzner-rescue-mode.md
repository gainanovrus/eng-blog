---
published: false
layout: single
title: Work with host system in Hetzner Rescue Mode
excerpt: >-
  Hetzner Rescue Mode allow repair host system when you cannot access in usually
  way. In this post I will show how to mount LVM volume and change some file.
  As example add new ssh-key to authorized_keys file.
categories: linux
tags: centos linux hetzner
toc: true
header:
  teaser: /assets/images/gitlab-ci/gitlab-ci-simple-pipeline.png
  og_image: /assets/images/gitlab-ci/gitlab-ci-simple-pipeline.png

change-runner-type:
  - url: /assets/images/gitlab-ci/change-runner-type.png
    image_path: /assets/images/gitlab-ci/change-runner-type.png
    title: "Determine the landing project with a current Runner (nginx)"
---

The Hetzner Rescue System is a Linux live environment that allows you to have
administrative access to your server. This makes it possible to carry out
repairs to the installed system, to check file systems or to install a new
operating system.

## Problem

In my example in the host was been installed CentOS 7 with SWRAID 1 and LVM.
And I want to get an access to this host by ssh public key (was forgotten a root
password).

```
$ ssh root@xxx.xxx.xxx.xxx
root@xxx.xxx.xxx.xxx's password:
Permission denied, please try again.
root@xxx.xxx.xxx.xxx's password:
Permission denied, please try again.
root@xxx.xxx.xxx.xxx's password:
Permission denied (publickey,password).
```
Where is `xxx.xxx.xxx.xxx` public IP of the server

## Instructions

1. Activating a rescue mode for selected server in Hetzner [web panel][web-panel].
Choose your ssh key for connecting to the rescue server without password.

<figure>
	<a href="/assets/images/hetzner-rescue/activate-rescue.png">
    <img src="/assets/images/hetzner-rescue/activate-rescue.png">
  </a>
</figure>


2. Reset the server. Select first or second option

<figure>
	<a href="/assets/images/hetzner-rescue/send-reboot.png">
    <img src="/assets/images/hetzner-rescue/send-reboot.png">
  </a>
</figure>


3. Connect to the server by ssh.
Where is `xxx.xxx.xxx.xxx` public IP of the server (it showed in the web panel)
```
$ ssh root@xxx.xxx.xxx.xxx
```

The server will return some information
```
-------------------------------------------------------------------

  Welcome to the Hetzner Rescue System.

  This Rescue System is based on Debian 8.0 (jessie) with a newer
  kernel. You can install software as in a normal system.

  To install a new operating system from one of our prebuilt
  images, run 'installimage' and follow the instructions.

  More information at http://wiki.hetzner.de

-------------------------------------------------------------------

Hardware data:

   CPU1: AMD Ryzen 7 1700X Eight-Core Processor (Cores 16)
   Memory:  64370 MB
   Disk /dev/sda: 512 GB (=> 476 GiB)
   Disk /dev/sdb: 512 GB (=> 476 GiB)
   Total capacity 953 GiB with 2 Disks

Network data:
   eth0  LINK: yes
         MAC:  00:00:00:00:00:00
         IP:   xxx.xxx.xxx.xxx
         IPv6: 2001:00f8:0011:2412::2/64
         RealTek RTL-8169 Gigabit Ethernet driver
```

4. Because I use LVM in the host system I run commands for scanning disk for
 Volume Groups and Physical Volumes.
```
root@rescue ~ # vgscan
  Reading all physical volumes.  This may take a while...
  Found volume group "vg0" using metadata type lvm2

root@rescue ~ # pvscan
  PV /dev/md1   VG vg0   lvm2 [476,31 GiB / 341,31 GiB free]
  Total: 1 [476,31 GiB] / in use: 1 [476,31 GiB] / in no VG: 0 [0   ]
```

5. Fine. Was founded a group named `vg0`. It is group that I want to mount.
Activate it.
```
root@rescue ~ # lvm vgchange -a y
4 logical volume(s) in volume group "vg0" now active
```

And view a result of it.
```
root@rescue ~ # lvscan
ACTIVE            '/dev/vg0/root' [10,00 GiB] inherit
ACTIVE            '/dev/vg0/tmp' [5,00 GiB] inherit
ACTIVE            '/dev/vg0/home' [20,00 GiB] inherit
```

We see the a familiar file structure, don't it?

6. Now display the devices that you can mount
```
root@rescue ~ # ls -l /dev/mapper/vg*
lrwxrwxrwx 1 root root 7 Sep 11 07:26 /dev/mapper/vg0-home -> ../dm-2
lrwxrwxrwx 1 root root 7 Sep 11 07:26 /dev/mapper/vg0-one -> ../dm-3
lrwxrwxrwx 1 root root 7 Sep 11 07:26 /dev/mapper/vg0-root -> ../dm-0
lrwxrwxrwx 1 root root 7 Sep 11 07:26 /dev/mapper/vg0-tmp -> ../dm-1
```

7. Then mount the desired MD device of the host server.
I will mount `/` in the host server as `/mnt` in the rescue server (current session)
```
root@rescue ~ # mount /dev/mapper/vg0-root /mnt
```

Now we can work with files as usual
```
root@rescue ~ # ls -l /mnt
total 1,2M
lrwxrwxrwx  1 root root    7 May 14 08:39 bin -> usr/bin
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 boot
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 dev
drwxr-xr-x 71 root root 4,0K Sep 10 16:09 etc
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 home
-rw-r-----  1 root root  733 Sep 10 16:09 installimage.conf
-rw-r-----  1 root root 1,2M Sep 10 16:09 installimage.debug
lrwxrwxrwx  1 root root    7 May 14 08:39 lib -> usr/lib
lrwxrwxrwx  1 root root    9 May 14 08:39 lib64 -> usr/lib64
drwx------  2 root root  16K Sep 10 16:07 lost+found
drwxr-xr-x  2 root root 4,0K Apr 11 06:59 media
drwxr-xr-x  2 root root 4,0K Apr 11 06:59 mnt
drwxr-xr-x  2 root root 4,0K Apr 11 06:59 opt
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 proc
dr-xr-x---  4 root root 4,0K Sep 10 16:08 root
drwxr-xr-x  3 root root 4,0K Sep 10 16:08 run
lrwxrwxrwx  1 root root    8 May 14 08:39 sbin -> usr/sbin
drwxr-xr-x  2 root root 4,0K Apr 11 06:59 srv
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 sys
drwxr-xr-x  2 root root 4,0K Sep 10 16:08 tmp
drwxr-xr-x 13 root root 4,0K May 14 08:39 usr
drwxr-xr-x 19 root root 4,0K May 14 08:39 var
```

8. Make some actions with host files.
As I said I want to add my ssh key into the host for get access to it without
password. For then edit an `.ssh/authorized_keys` file
```
root@rescue ~ # vi /mnt/root/.ssh/authorized_keys

root@rescue ~ # ls -l /mnt/root/.ssh/authorized_keys
-rw-r--r-- 1 root root 747 Sep 11 07:28 /mnt/root/.ssh/authorized_keys

root@rescue ~ # chmod 600 /mnt/root/.ssh/authorized_keys

root@rescue ~ # ls -l /mnt/root/.ssh/authorized_keys
-rw------- 1 root root 747 Sep 11 07:28 /mnt/root/.ssh/authorized_keys
```

9. In the finish of work with the host files unmount the root device.
```
root@rescue ~ # umount /dev/mapper/vg0-root
```

And deactivate volume group `vg0`
```
root@rescue ~ # lvm vgchange -a n vg0
  0 logical volume(s) in volume group "vg0" now active
```

10. Now you can exit from rescue mode. Just run a reboot command.
```
root@rescue ~ # reboot

Broadcast message from root@rescue on pts/0 (Tue 2018-09-11 08:58:23 CEST):

The system is going down for reboot NOW!
```

11. That's all. The server will reboot and the host system will start.
Try to connect.
```
$ ssh root@xxx.xxx.xxx.xxx
Last failed login: Mon Sep 10 16:10:44 CEST 2018 from yyy.yyy.yyy.yyy on ssh:notty
There was 1 failed login attempt since the last successful login.
[root@CentOS-75-64-minimal ~]# hostname
CentOS-75-64-minimal
```

## Conclusion
In my practice I often repair system in rescue mode. With rescue I monitor the
logs when system didn't start. It's very useful tool for fix mistakes with
network and firewall settings.

## Additional information
[lvm(8) - Linux man page][lvm]
[Hetzner Rescue System - Official Information](https://www.hetzner.com/unternehmen/rescue-system)
[Hetzner Rescue System/en - Official Manual][rescue-doc]

[web-panel]: https://robot.your-server.de/server
[rescue-doc]: https://wiki.hetzner.de/index.php/Hetzner_Rescue-System/en
[lvm]: https://linux.die.net/man/8/lvm
