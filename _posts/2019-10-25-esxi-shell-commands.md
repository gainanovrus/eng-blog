---
published: true
layout: single
title: ESXi. The command line and shell magic
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

## Introduction

[ESXi](https://www.vmware.com/products/esxi-and-esx.html) is a type-1 hypervisor, meaning it runs directly on system hardware without the need for an operating system (OS). Type-1 hypervisors are also referred to as bare-metal hypervisors because they run directly on hardware.

VMware provides a powerful and convenient graphical interface for managing ESXi servers.

In this case, using the command line interface (CLI) is what you need –
it is possible to configure all settings, including the hidden ones
in the command line which is also referred to as the console.
In addition to traditional commands that are the same in Linux and ESXi,
ESXi has its own ESXCLI commands.

ESXCLI is a part of the ESXi shell, this is a CLI framework intended to manage a virtual infrastructure
(ESXi components such as hardware, network, storage, etc.) and control ESXi itself on the low level.
All ESXCLI commands must be run in the ESXi shell (console).
Generally, ESXCLI is the command that has a wide list of subcommands called namespaces and their options.
The ESXCLI command is present right after ESXi installation along with other ESXi shell commands.
Notice that ESXCLI commands are case-sensitive, similarly to other console commands used in ESXi.

The ESXCLI log file is located in `/var/log/esxcli.log`.

The data is written to this file if an ESXCLI command has not been executed successfully.
If an ESXCLI command is run successfully, nothing is written to this log file.

The most useful ESXCLI commands are explained in today’s blog post.

## Environment

In this article I have VMware ESXi Hypervisor 6.7 U3 (Free version)

## Usage

### How to open the CLI in ESXi?

By default, ESXi shell is disabled for local and remote access;
hence, you are not able to run ESXi shell commands until you enable the ESXi shell.
VMware has made this restriction for security reasons.

You can enable this with local command line direct connected to ESXi host (named DCUI) or with WebGUI (throw web-browser).

Using the ESXi default interface. In the ESXi Direct Console User Interface (DCUI), go to `Troubleshooting Options`,
navigate to **Enable ESXi Shell** and **Enable SSH** strings and press Enter to enable each option.

After enabling the ESXi shell, press **Alt+F1** to open the console on the machine running ESXi.
You should enter your login and password after that (credentials of the root user can be used).
If you need to go back to the ESXi DCUI, press **Alt+F2**.

The Enable SSH option allows you to open the ESXi console remotely by using an SSH client.

### Command list

The entire list of all available ESXCLI namespaces and commands is displayed after running the command:
```
esxcli esxcli command list
```

### Checking hardware

To view installed PCI devices, run the following ESXCLI command:
```
esxcli hardware pci list | more
```

Check the amount of memory installed on the ESXi server:
```
esxcli hardware memory get
```

View the detailed information about installed processors:
```
esxcli hardware cpu list
```

### Power control ESXi host

Power off an ESXi host:
```
esxcli system shutdown poweroff
```

The command for rebooting the host is similar. To write a reason of rebooting use `-r`:
```
esxcli system shutdown reboot -r "applying new update"
```

### Start and stop VM

The basic command to control the virtual machines is
```
vim-cmd vmsvc
```

To list all virtual machines run:
```
vim-cmd vmsvc/getallvms
```

To work with individual VMs you need the `<ID>` provided by the previous command.

To get a summary of the machine run (where `<ID>` is a number):
```
vim-cmd vmsvc/get.summary <ID>
```

To power on/power off/suspend a VM run:
```
vim-cmd vmsvc/power.on <ID>
vim-cmd vmsvc/power.off <ID>
vim-cmd vmsvc/power.suspend <ID>
```

### Stop VM via esxcli

Use this to forcibly stop a virtual machine.

List all running virtual machines on the system to see the **World ID** of the virtual machine that you want to stop.
```
esxcli vm process list
```

Stop the virtual machine by running the following command.
```
esxcli vm process kill --type <kill_type> --world-id <ID>
```

The command supports three `--type` options. The following types are supported through the `--type` option:
* `soft`. Gives the VMX process a chance to shut down cleanly (like kill or kill -SIGTERM)
* `hard`. Stops the VMX process immediately (like kill -9 or kill -SIGKILL)
* `force`. Stops the VMX process when other options do not work.

https://pubs.vmware.com/vsphere-51/index.jsp?topic=%2Fcom.vmware.vcli.examples.doc%2Fcli_manage_vms.10.9.html

### Autostart of VM

Get a list of virtual machine IDs:
```
vim-cmd vmsvc/getallvms
```

Enable force autostart feature:
```
vim-cmd hostsvc/autostartmanager/enable_autostart true
```

Now check the VMs startup settings:
```
vim-cmd hostsvc/autostartmanager/get_autostartseq
```

Make sure that the **startAction** value of the VM has changed to `powerOn`.

### Uplink of vSwitch

For example you have one **physical nic** on your ESX then that called `vmnic0`.
If you have one **vSwitch** it will be called `vSwitch0`.

To list your vSwitch run:
```
esxcfg-vswitch -l
```

If you have no vmnic or are missing the one that you want to use for the management
network then you need to **add** it by using the following command:
```
esxcfg-vswitch -L vmnic0 vSwitch0
```

Replace `vmnic0` with your vmnic and same for the switch.

So if I wanted to **remove** `vmnic0` from `vSwitch0` I would use the following command:
```
esxcfg-vswitch -U vmnic0 vSwitch0
```

### Enter to maintenance mode

Enter the host to the maintenance mode.

```
esxcli system maintenanceMode set --enable yes
```

or

```
vim-cmd hostsvc/maintenance_mode_enter
```

Get currently status of the host
```
esxcli system maintenanceMode get
```

### Backup configuration

Using the ESXi Command Line to Back up ESXi Host.
You don’t need to install any additional software to use the ESXi command line.
You have to enable ESXi shell and remote SSH access to an ESXi host.
Once you have connected to your ESXi host via SSH, you can run the commands.

ESXi configuration is saved every hour automatically to the `/bootblank/state.tgz` file.
For this reason, you should ensure that the current ESXi configuration is written to ESXi configuration files right now to confirm that all changes made to ESXi configuration since the last autosave are saved:

```
vim-cmd hostsvc/firmware/sync_config
```

Back up ESXi configuration:
```
vim-cmd hostsvc/firmware/backup_config
```
The result:
```
Bundle can be downloaded at : http://*/downloads/52f857f5-1bed-7e86-7c8b-c8c389f4d7b6/configBundle-localhost.localdomain.tgz
```

As a result, you’ll receive a link to download the `configBundle.tgz` archive from the ESXi host.
You should replace the asterisk with the IP address of your ESXi host.
The archive file that contains the ESXi configuration backup is saved to the `/scratch/downloads` directory.
The scratch partition was mentioned in the blog post about installing ESXi on a USB Flash drive.

### Restore configuration

You should have ESXi of the same version and build number installed on the machine
where you want to restore the ESXi configuration. The UUID must be the same on both the ESXi server that was backed up and the ESXi server on which the configuration must be restored (can be obtained using the command `esxcfg-info -u`).

> Use numeric 1 as force option to override the UUID mismatch. For example, `vim-cmd hostsvc/firmware/restore_config 1 /tmp/configBundle.tgz`. The UUID value of the backed up ESXi host is mentioned in the `Manifest.txt` file inside the `configBundle.tgz` backup archive.

Once you have prepared your freshly installed ESXi host to restore ESXi configuration from a backup, connect to the ESXi host via SSH and enter the host to the maintenance mode.
```
esxcli system maintenanceMode set --enable yes
```

Copy the archive that contains the ESXi configuration backup `configBundle-xxxx.tgz` archive from a local machine to the `/tmp/` directory on the destination ESXi server.

Rename the `configBundle-xxxx.tgz` file to `configBundle.tgz` before you enter a command to restore ESXi configuration. Otherwise you will get an error message: *File /tmp/configBundle.tgz was not found*.

Restore the ESXi configuration:
```
vim-cmd hostsvc/firmware/restore_config /tmp/configBundle.tgz
```

After running this command an ESXi host will be rebooted automatically.

### Login to console by SSH rsa key

You should have a SSH key on the host that you would like to connect to ESXi.
So I think you already know how to generate ssh-keys, I just write command to add keys to the ESXi host - `esxi.machine.com`.

```
cat ~/.ssh/id_rsa.pub | ssh root@esxi.machine.com 'cat >> /etc/ssh/keys-root/authorized_keys'
```

### Configuring Login Behavior

ESXi has a good security feature to add a root account lockout for safety. After a number of failed login attempts, the server will trigger a lockout. By default, a maximum of five failed attempts is allowed before the account is locked. The account is unlocked after 15 minutes by default.

You can configure the login behavior for your ESXi host with the following advanced options:
* `Security.AccountLockFailures` - Maximum number of failed login attempts before a user's account is locked. Zero disables account locking.
* `Security.AccountUnlockTime` - Number of seconds that a user is locked out.


> Remote access for ESXi local user account 'root' has been locked for 120 seconds after xxx failed login attempts.

Are you seeing this message? Do not worry, you are in the right place.  Now, let’s look at what to do if your ESXi root account is locked. The command line to clear the lockout status and reset the count to zero for an account is shown here with the root account as an example:

```
pam_tally2 --user root --reset
```

### Conclusion

Today’s blog post has covered a series of ESXi shell commands. Using the command line
interface gives you more power in addition to WebGUI. You can use ESXi shell commands
for viewing and configuring settings that are hidden or not available in the GUI.
Use the ESXi shell commands list provided in this blog post for fine ESXi tuning.

## Additional information

* [How to Back Up and Restore VMware ESXi Host Configuration](https://www.nakivo.com/blog/back-up-and-restore-vmware-esxi-host-configuration-guide/) -
  Backup and restore instructions from Nakivo.
* [Best ESXCLi and ESXi Shell Commands for your VMware Environment](https://www.nakivo.com/blog/most-useful-esxcli-esxi-shell-commands-vmware-environment/) -
  A one of explanation of the most often used ESXi shell commands from Nakivo.
* [How to back up ESXi host configuration (2042141)](https://kb.vmware.com/s/article/2042141)-
  How to back up and restore the ESXi host from VMWare.
* [How to Backup and Restore the VMware ESXi 6.5 Configuration](https://graspingtech.com/backup-vmware-esxi-6-5-configuration/) -
  Another manual instruction to this. This one from graspingtech.
* [Резервное копирование настроек VMWare ESXI](https://linux-admins.ru/article.php?id_article=17) -
  On Russian language.
* [Configure Autostart of VM on VMware ESXi](https://theitbros.com/config-virtual-machine-auto-startup-vmware-esxi/) -
  Some information about starting VM process, and how to do it with cmd
