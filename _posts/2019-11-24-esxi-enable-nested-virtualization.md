---
published: true
layout: single
title: ESXi. Enabling Nested virtualization
excerpt: >-
  Configure ESXi for allowing Linux guest run VirtualBox/KVM/Proxmox and create a nested guests.
categories: sysad
tags: esxi vm
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2020-05-21
---

Nested virtualization is when you run an hypervisor, like PVE or others, inside a virtual machine
(which is of course running on another hypervisor) instead that on real hardware.
In other words, you have a host hypervisor, hosting a guest hypervisor (as a vm), which can hosts its own vms.

This obviously adds an overhead to the nested environment, but it could be useful in some cases:
* test (or learn) how to manage hypervisors before actual implementation,
* test some dangerous/tricky procedure involving hypervisors before actually doing it on the real thing,
* enable businesses to deploy their own virtualization environment, e.g. on public services (cloud)

## Requirements

In order to have the fastest possible performance, near to native, any hypervisor should have access to some (real) hardware features that are generally useful for virtualization, the so called 'hardware-assisted virtualization extensions' (see http://en.wikipedia.org/wiki/Hardware-assisted_virtualization).

In nested virtualization, also the guest hypervisor should have access to hardware-assisted virtualization extensions, and that implies that the host hypervisor should expose those extension to its virtual machines. In principle it works without those extensions too but with poor performance and it is not an option for productive environment (but maybe sufficient for some test cases).

## Configuration

To enable ESXi nested virtualization use next command that added the vhv.allow parameter to the configuration file.
```sh
echo 'vhv.allow = "TRUE"' >> /etc/vmware/config
```

You can now enable VHV on a per VM basis and using the Web Client which basically adds the `vhv.enable = "true"` parameter to the VM's .VMX configuration file.

If you are creating a nested ESXi/Proxmox VM, and you enable the checkbox for:

> Hardware virtualization: **Expose hardware assisted virtualization to the guest OS**

![enale-vhv]({{ site.url }}{{ site.baseurl }}/assets/images/esxi/nested/1.png){: .align-center}

This sets the same value as if manually adding ​`vhv.enable = “true”​` to the .vmx file. So with that said, if you created a nested ESXi VM using the Web Client and enabled that option then you don’t need to add it manually.

Once installed the guest OS, if GNU/Linux you can enter and verify that the hardware virtualization support is enabled by doing

```
# egrep '(vmx|svm)' --color=always /proc/cpuinfo
```

Non empty results will sign about success of enabling virtualization:
```
flags           : fpu ... ... vmx ... ... arch_capabilities
```

## Additional information

* [Enabling Nested virtualization on ESXi](https://www.virtuallyghetto.com/2012/08/how-to-enable-nested-esxi-other.html)
* [Nested Virtualization - VirtualBox inside ESXi](https://egustafson.github.io/post/esxi-nested-virtualbox/) -
  Good explanation about nested virtualization with VirtualBox
* [Nested Virtualization](https://pve.proxmox.com/wiki/Nested_Virtualization) -
  From Proxmox authors
