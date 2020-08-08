---
published: true
layout: single
title: DELL. Setup SSH server on OS9 switch (S4048 10G switch)
excerpt: >-
  Setup and configure ssh server on Dell Networking system with RSA and password authentication.
categories: sysad
tags: dell networks security
toc: true
last_modified_at: 2020-08-08T00:00:00-00:00

# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
---

Yet another technical post about DELL PowerSwitch [S4048-ON][s4048-specs] that I am configuring.

Secure shell (SSH) is a protocol for secure remote login and other secure network services over an insecure network.
Dell Networking OS is compatible with SSH versions 1.5 and 2, in both the client and server modes.
SSH sessions are encrypted and use authentication.

The RSA-SSH authentication permits the user to establish an SSH session without entering a password.
This allows the user to use a custom script, for example, to automatically launch an SSH session to a Force10 switch, eliminating the potential security vulnerability of including a password in the script.

## Prerequisites

If you just unpack you will get an empty switch that does not listen any network connection.
Firstly you should get a console cable. My switch has a micro USB interface on [front-panel](https://www.dell.com/support/manuals/us/en/04/networking-s4048t-on/s4048t-on-ig-pub/micro-usb-b-console-port-access?guid=guid-e873c8e3-5514-4cc5-b457-35f074d44fa2&lang=en-us) for console connection.
So I'm plug-in a usual USB cable into this port, then install a [drivers](https://www.dell.com/support/home/en-us/drivers/driversdetails/?driverid=r5k9d) for COM-ports and connect to the switch with this settings:
```
115200-baud rate
No parity
8 data bits
1 stop bits
No flow control
```

If you have seen an input invitation like this:
```
DellEMC#
```
You can start to configure a network interfaces and SSH server.

## Create user

Enter to `CONF` mode with `conf` command.

Create a user with name `admin` and privilege level `15` (`<password>` enter the strong password here).
```
Dell(config)# username admin password <password> privilege 15
```
The [privilege level](https://www.dell.com/support/manuals/us/en/19/force10-s4048-on/s4048_on_9.9.0.0_config_pub-v1/configuring-privilege-levels?guid=guid-2fa480e0-1660-41d8-8ac2-45aa90d5bb71&lang=en-us) need to restrict access to commands.
This level should between from 0 to 15. The default privilege level is 1.  
The user with privilege level 15 is like `root` without any restrictions.
It allows to access to the system begins at EXEC `Privilege` mode, and all commands are available.


## Configure Management interface

A Management interface need to remote configure and run commands on the switch.
It use static or dynamic IP address.

Let's add an IP `10.0.0.1/24` to the first management interface. Then enable it and exit.
```
Dell(config)# interface managementethernet 1/1
Dell(conf-if-ma-0/0)# ip add 10.0.0.1/24
Dell(conf-if-ma-0/0)# no shutdown
Dell(conf-if-ma-0/0)# exit
```

## Configure SSH server


### Enable SSH

Just input this command
```
Dell(config)# ip ssh server enable
Dell(config)# ip ssh server version 2
```

### Generate keys

Generate keys for the SSH server
```
Dell(config)# crypto key generate rsa
```

### Enable password auth

Enable a password authentication
```
Dell(config)# ip ssh password-authentication enable
```

### Enable SSH-RSA password auth


Enable a RSA authentication
```
Dell(config)# ip ssh rsa-authentication enable
```

Copy your public part of RSA-key (`.ssh/id_rsa.pub`) into internal `flash://` memory from remote host (`10.0.0.3`)
```
Dell# copy scp://username:password@10.0.0.3/.ssh/id_rsa.pub flash://id_rsa.pub
```

Add authentication with this key
```
Dell# ip ssh rsa-authentication my-authorized-keys flash://id_rsa.pub

Creating ADMIN_DIR/ssh
RSA keys added to user's list of authorized-keys.
Delete the file flash://id_rsa.pub : (yes/no) ? yes
```

### Try to connect

So we configure an management IP of switch - `10.0.0.1`.
You can now SSH to the switch using RSA authentication.
The user will encrypt the initial connection using the private key, which the switch will authenticate using the public key; no password exchange is necessary.
```
$ ssh admin@10.0.0.1
```
If it will successfully you see the hostname of device - `Dell#`.

### Display SSH connection information

In the end of configuring I want to show a status of SSH with command `show ip ssh`
```
Dell# show ip ssh

SSH server                : enabled.
SSH server version        : v2.
SSH server vrf            : default.
SSH server ciphers        : aes256-ctr,aes256-cbc,aes192-ctr,aes192-cbc,aes128-ctr,aes128-cbc,3des-cbc.
SSH server macs           : hmac-sha2-256,hmac-sha1,hmac-sha1-96,hmac-md5,hmac-md5-96.
SSH server kex algorithms : diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1,diffie-hellman-group14-sha1.
Password Authentication   : enabled.
Hostbased Authentication  : disabled.
RSA       Authentication  : enabled.
Challenge Response Auth   : disabled.
   Vty          Encryption      HMAC            Remote IP
*  0            aes128-ctr      hmac-sha2-256   10.0.0.2
```



## Conclusion

In this article I want to show how to enable SSH-server on Dell Networking system.
Also this setup make possible to secure communicate with the device without any passwords.
It uses RSA-SSH authentication and connection with imported public (`id_rsa`) keys.

## Additional information

* [SSH Key Auth on Dell PowerConnect Switches](https://www.netwrix.com/cisco_commands_cheat_sheet.html) - another blog post with the similar problem
* [Dell Command Line Reference Guide for the S4048–ON System](https://www.dell.com/support/manuals/us/en/19/force10-s4048-on/s4048-on-9.14.0.0-cli/about-this-guide) - dell man page included full command list


[s4048-specs]: https://i.dell.com/sites/doccontent/shared-content/data-sheets/en/Documents/Dell-EMC-Networking-S4048-ON-Spec-Sheet.pdf
