---
published: true
layout: single
title: RHEL8. Install to DELL server with RAID (SAS2008)
excerpt: >-
  In RHEL/CentOS 8 was be removed many drivers for popular enterprise raid-controllers.
  It brings some troubles when you try to install OS.
  Without drivers an installation destination section will be empty.
  No drives will be found. To see your HDD in the server need add drivers back.
categories: linux
tags: dell centos raid drivers
toc: true
header:
  teaser: /assets/images/dell-rhel8-install-drivers.jpg
  og_image: /assets/images/dell-rhel8-install-drivers.jpg
---

In RHEL/CentOS 8 was be [removed many drivers][removed-drivers] for popular enterprise raid-controllers.
It brings some troubles when you try to install OS.
Without drivers an installation destination section will be empty.
No drives will be found. To see your HDD in the server need add drivers back.

With ELRepo it will be easy. [ELRepo repository][elrepo] includes a lot of drivers.
The full list of supported drives [here][supported-drives].
I found an ID of raid-controller for my DELL PowerEdge C6320 in the list.
Read next, and I show how it is possible install CentOS 8 in the DELL server.

## Setup

First of all we must understand what raid controller is installed into server. Use command `lspci -nn` for show all connected devices
```
Serial Attached SCSI controller [0107]:
LSI Logic / Symbios Logic SAS2008 PCI-Express Fusion-MPT SAS-2 [Falcon] [1000:0072] (rev 03)
```
I get this output in my another similar server with CentOS 7.

Also I check it in the BIOS, and the name is similar - `LSI SAS2 MPT Controller SAS2008`.

Main part of this - `1000:0072` - is a device ID.
It helps to us found a right driver from ELRepo.

Next, we need to choose what DUD (driver update disk) we should to use in installation process for our hardware.
Follow to [this][supported-drives] page and find our ID (`1000:0072`).
It point to the `mpt3sas.ko` section.
So, we need to use a las `mpt3sas` iso images from [here][elrepo].
When I wrote this article it is - [`dd-mpt3sas-28.100.00.00-2.el8_1.elrepo.iso`][dd-mpt3sas-iso].

For using them during installation need to made some required actions.
Read the [instruction][red-hat-inst-dd] from Red Hat about it.

I just show you the simple method that I used.
My server has connected to local network and internet.
DHCP-server in the LAN assign an IP-address to the server, and server can download any resources from network.
So it is the reason why I prefer to add drivers by http link. What we will need to do?

Run the installation process. When boot menu window has been shown,
press the `Tab` key on your keyboard to display the boot command line.

![boot-menu]({{ site.baseurl }}/assets/images/dell-centos/boot-menu.png){: .align-center}

Append the `inst.dd=https://elrepo.org/linux/dud/el8/x86_64/dd-mpt3sas-28.100.00.00-2.el8_1.elrepo.iso`
boot option to the command line and press `Enter` to execute the boot process.

The installer should detect the DUD iso and install the proper drivers.
And raid controller with drives will be accessed as installation destination.

![destination-window-with-raid-controller]({{ site.baseurl }}/assets/images/dell-centos/destination-with-raid.png){: .align-center}

So, now you can install the last RHEL/CentOS into DELL Server.
After reboot the system the drivers will be included into installed system.

The information about the mpt3sas module presented below:
```
[root@cpu7 ~]# modinfo mpt3sas
filename:       /lib/modules/4.18.0-147.3.1.el8_1.x86_64/weak-updates/mpt3sas/mpt3sas.ko
alias:          mpt2sas
version:        28.100.00.00
license:        GPL
description:    LSI MPT Fusion SAS 3.0 Device Driver
author:         Avago Technologies <MPT-FusionLinux.pdl@avagotech.com>
rhelversion:    8.1
srcversion:     6ABC8F89B061AC0B78BCED5
alias:          pci:v00001000d000000E6sv*sd*bc*sc*i*
alias:          pci:v00001000d000000E5sv*sd*bc*sc*i*
alias:          pci:v00001000d000000B2sv*sd*bc*sc*i*
alias:          pci:v00001000d000000E2sv*sd*bc*sc*i*
alias:          pci:v00001000d000000E1sv*sd*bc*sc*i*
alias:          pci:v00001000d000000D1sv*sd*bc*sc*i*
alias:          pci:v00001000d000000ACsv*sd*bc*sc*i*
alias:          pci:v00001000d000000ABsv*sd*bc*sc*i*
alias:          pci:v00001000d000000AAsv*sd*bc*sc*i*
alias:          pci:v00001000d000000AFsv*sd*bc*sc*i*
alias:          pci:v00001000d000000AEsv*sd*bc*sc*i*
alias:          pci:v00001000d000000ADsv*sd*bc*sc*i*
alias:          pci:v00001000d000000C3sv*sd*bc*sc*i*
alias:          pci:v00001000d000000C2sv*sd*bc*sc*i*
alias:          pci:v00001000d000000C1sv*sd*bc*sc*i*
alias:          pci:v00001000d000000C0sv*sd*bc*sc*i*
alias:          pci:v00001000d000000C8sv*sd*bc*sc*i*
alias:          pci:v00001000d000000C7sv*sd*bc*sc*i*
alias:          pci:v00001000d000000C6sv*sd*bc*sc*i*
alias:          pci:v00001000d000000C5sv*sd*bc*sc*i*
alias:          pci:v00001000d000000C4sv*sd*bc*sc*i*
alias:          pci:v00001000d000000C9sv*sd*bc*sc*i*
alias:          pci:v00001000d00000095sv*sd*bc*sc*i*
alias:          pci:v00001000d00000094sv*sd*bc*sc*i*
alias:          pci:v00001000d00000091sv*sd*bc*sc*i*
alias:          pci:v00001000d00000090sv*sd*bc*sc*i*
alias:          pci:v00001000d00000097sv*sd*bc*sc*i*
alias:          pci:v00001000d00000096sv*sd*bc*sc*i*
alias:          pci:v00001000d0000007Esv*sd*bc*sc*i*
alias:          pci:v00001000d000002B1sv*sd*bc*sc*i*
alias:          pci:v00001000d000002B0sv*sd*bc*sc*i*
alias:          pci:v00001000d0000006Esv*sd*bc*sc*i*
alias:          pci:v00001000d00000087sv*sd*bc*sc*i*
alias:          pci:v00001000d00000086sv*sd*bc*sc*i*
alias:          pci:v00001000d00000085sv*sd*bc*sc*i*
alias:          pci:v00001000d00000084sv*sd*bc*sc*i*
alias:          pci:v00001000d00000083sv*sd*bc*sc*i*
alias:          pci:v00001000d00000082sv*sd*bc*sc*i*
alias:          pci:v00001000d00000081sv*sd*bc*sc*i*
alias:          pci:v00001000d00000080sv*sd*bc*sc*i*
alias:          pci:v00001000d00000065sv*sd*bc*sc*i*
alias:          pci:v00001000d00000064sv*sd*bc*sc*i*
alias:          pci:v00001000d00000077sv*sd*bc*sc*i*
alias:          pci:v00001000d00000076sv*sd*bc*sc*i*
alias:          pci:v00001000d00000074sv*sd*bc*sc*i*
alias:          pci:v00001000d00000072sv*sd*bc*sc*i*
alias:          pci:v00001000d00000070sv*sd*bc*sc*i*
depends:        scsi_transport_sas,raid_class
name:           mpt3sas
vermagic:       4.18.0-147.el8.x86_64 SMP mod_unload modversions
sig_id:         PKCS#7
signer:         ELRepo.org Secure Boot Key
sig_key:        E9:D4:71:CF:B4:FE:13:6C
sig_hashalgo:   sha256
signature:      15:E2:B9:0D:5D:4D:A8:C0:4B:A6:75:CD:F7:53:C4:E9:C8:02:88:94:
                8F:2F:36:13:E8:56:55:FE:FC:D7:74:CD:37:29:25:7D:11:11:99:E5:
                8C:B2:98:3F:C6:A6:FA:CC:BC:53:66:26
parm:           logging_level: bits for enabling additional logging info (default=0)
parm:           max_sectors:max sectors, range 64 to 32767  default=32767 (ushort)
parm:           missing_delay: device missing delay , io missing delay (array of int)
parm:           max_lun: max lun, default=16895  (ullong)
parm:           hbas_to_enumerate: 0 - enumerates both SAS 2.0 & SAS 3.0 generation HBAs
                  1 - enumerates only SAS 2.0 generation HBAs
                  2 - enumerates only SAS 3.0 generation HBAs (default=0) (ushort)
parm:           diag_buffer_enable: post diag buffers (TRACE=1/SNAPSHOT=2/EXTENDED=4/default=0) (int)
parm:           disable_discovery: disable discovery  (int)
parm:           prot_mask: host protection capabilities mask, def=7  (int)
parm:           max_queue_depth: max controller queue depth  (int)
parm:           max_sgl_entries: max sg entries  (int)
parm:           msix_disable: disable msix routed interrupts (default=0) (int)
parm:           smp_affinity_enable:SMP affinity feature enable/disable Default: enable(1) (int)
parm:           max_msix_vectors: max msix vectors (int)
parm:           irqpoll_weight:irq poll weight (default= one fourth of HBA queue depth) (int)
parm:           mpt3sas_fwfault_debug: enable detection of firmware fault and halt firmware - (default=0)
```

## Additional information

* [The ELRepo Blog - RHEL 8.0 and support for removed adapters](https://elrepoproject.blogspot.com/2019/08/rhel-80-and-support-for-removed-adapters.html) - an article in ELRepo blog about DUD that they provide to community
* [Installing RHEL 8.1 on Dell R710_R610 with H700 Raid Controller _ FATMIN](https://fatmin.com/2019/11/23/installing-rhel-8-1-on-dell-r710-r610-with-h700-raid-controller/) - instruction for another DELL server with megaraid_sas drivers
* [removal of SAS-2 controller drivers in RHEL 8 - Red Hat Customer Portal](https://access.redhat.com/discussions/3722151) - a hot discussions in redhat forum about removed drivers
* [Updating drivers during installation Red Hat Enterprise Linux 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/performing_an_advanced_rhel_installation/updating-drivers-during-installation_installing-rhel-as-an-experienced-user) - official instruction about driver update during installation process


[removed-drivers]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/considerations_in_adopting_rhel_8/hardware-enablement_considerations-in-adopting-rhel-8#removed-adapters_hardware-enablement
[elrepo]: https://elrepo.org/linux/dud/el8/x86_64/
[supported-drives]: http://elrepo.org/tiki/DeviceIDs
[dd-mpt3sas-iso]: https://elrepo.org/linux/dud/el8/x86_64/dd-mpt3sas-28.100.00.00-2.el8_1.elrepo.iso
[red-hat-inst-dd]: https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/performing_an_advanced_rhel_installation/updating-drivers-during-installation_installing-rhel-as-an-experienced-user#performing-an-assisted-driver-update_updating-drivers-during-installation
