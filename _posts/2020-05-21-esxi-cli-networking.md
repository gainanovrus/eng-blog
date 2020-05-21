---
published: true
layout: single
title: ESXi. Create Management Network (vmk0) VMkernel interface using ESXi Command line
excerpt: >-
  This quick post about ESX CLI networking basics. Here is should be useful to recover the Management Network functionality on your ESXi host.
categories: sysad
tags: esxi vm networks
# toc: true
# header:
#  teaser: /assets/images/traefik/traefik_docker_ssl_logo.png
#  og_image: /assets/images/traefik/traefik_docker_ssl_logo.png
---

I want to continue my [last post][previous] about `esxcli` tool. Here is I'll describe some useful console commands that will helps you to create a management network adapter (`VMkernel`) and configure them. In addition commands to work with Port Groups, vSwitches, NetStack, Uplinks will be placed here in the one quick post.

### Network device (Uplink)

1. List network devices

```
# esxcli network nic list

Name    PCI Device    Driver  Admin Status  Link Status  Speed  Duplex  MAC Address         MTU  Description
------  ------------  ------  ------------  -----------  -----  ------  -----------------  ----  ----------------------------------------------------------------
vmnic0  0000:81:00.0  ixgben  Up            Up           10000  Full    7c:d3:0a:00:00:00  1500  Intel Corporation 82599EB 10-Gigabit SFI/SFP+ Network Connection
vmnic1  0000:81:00.1  ixgben  Up            Down             0  Half    7c:d3:0a:00:00:00  1500  Intel Corporation 82599EB 10-Gigabit SFI/SFP+ Network Connection
vmnic2  0000:01:00.0  igbn    Up            Up            1000  Full    a0:36:9f:00:00:00  1500  Intel Corporation Ethernet Server Adapter I350-T2
vmnic3  0000:01:00.1  igbn    Up            Down             0  Half    a0:36:9f:00:00:00  1500  Intel Corporation Ethernet Server Adapter I350-T2
```

### Virtual Switch (vSwitch)

1. List standard virtual switches:

```
# esxcli network vswitch standard list

vSwitch0
   Name: vSwitch0
   Class: cswitch
   Num Ports: 10496
   Used Ports: 6
   Configured Ports: 128
   MTU: 1500
   CDP Status: listen
   Beacon Enabled: false
   Beacon Interval: 1
   Beacon Threshold: 3
   Beacon Required By:
   Uplinks: vmnic2
   Portgroups: WAN network, Management
```

2. Add an uplink (`remove` - the same)

```
esxcli network vswitch standard uplink add -v vSwitch0 -u vmnic0
```

3. Create a new vSwitch (`remove` - the same)

```
esxcli network vswitch standard add -v vSwitch_New
```

### Network Stack (NetStack)

1. List an existing network stack

```
# esxcli network ip netstack list

defaultTcpipStack
   Key: defaultTcpipStack
   Name: defaultTcpipStack
   State: 4660
```

2. Add a new stack (`remove` - the same)

```
esxcli network ip netstack add -N New_Stack
```

### Port Group

1. List port groups

```
# esxcli network vswitch standard portgroup list

Name           Virtual Switch  Active Clients  VLAN ID
-------------  --------------  --------------  -------
Management     vSwitch0                     1        0
VM network     vSwitchLAN                   1        0
WAN network    vSwitch0                     1       98
```

2. Add a new port group (`remove` - the same)

```
esxcli network vswitch standard portgroup add -p New_Management -v vSwitch_New
```

### Network Interface (vmk nic)

1. List VMkernel interfaces

```
# esxcli network ip interface list

vmk0
   Name: vmk0
   MAC Address: 00:50:56:6c:01:11
   Enabled: true
   Portset: vSwitch0
   Portgroup: Management
   Netstack Instance: defaultTcpipStack
   VDS Name: N/A
   VDS UUID: N/A
   VDS Port: N/A
   VDS Connection: -1
   Opaque Network ID: N/A
   Opaque Network Type: N/A
   External ID: N/A
   MTU: 1500
   TSO MSS: 65535
   RXDispQueue Size: 1
   Port ID: 33554443
```

2. List them IP settings

```
# esxcli network ip interface ipv4 get

Name  IPv4 Address  IPv4 Netmask   IPv4 Broadcast  Address Type  Gateway      DHCP DNS
----  ------------  -------------  --------------  ------------  -----------  --------
vmk0  192.168.1.11  255.255.255.0  192.168.1.255   STATIC        192.168.1.1     false
```

3. Add a new vmkernel interface

```
esxcli network ip interface add -i vmk2 -p New_Management -N New_Stack
```

4. Configure the new vmkernel interface

```
esxcli network ip interface ipv4 set -i vmk2 -I 172.16.1.111 -N 255.255.255.0 -t static -g 172.16.1.25
```

5. Set default Gateway

```
esxcfg-route -a default 172.16.1.25
```

6. Mark `vmk2` for Management traffic

```
esxcli network ip interface tag add -i vmk2 -t Management
```

### Search Domains (DNS)

1. List a DNS servers

```
# esxcli network ip dns search list

   DNSSearch Domains: 192.168.1.1
```

2. Add a new DNS

```
esxcli network ip dns search -d 8.8.8.8 -N New_Stack
```

### Network Troubleshooting commands

`vmkping` is one of them. This command allows us to use the IP stack of VMkernel interface to send a ping command to another VMkernel interface. Like this you can check if the remote site (remote host) reply on this VMkernel.

Let's describe our scenario. I'm connected via Putty to my esxi host on `192.168.1.11` and I'll be pinging by using `vmk0` another host in LAN on `192.168.1.22`.
```
# vmkping -I vmk0 192.168.1.22

PING 192.168.1.22 (192.168.1.22): 56 data bytes
64 bytes from 192.168.1.22: icmp_seq=0 ttl=64 time=0.583 ms
64 bytes from 192.168.1.22: icmp_seq=1 ttl=64 time=0.406 ms

--- 192.168.1.22 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.406/0.494/0.583 ms
```

## Additional information

* [Виртуализация - Некоторые полезные команды ESXCLI в VMware ESXi 5.0, чтобы побольше узнать о хосте и окружении](https://www.vmgu.ru/news/vmware-esxi-50-esxcli)
* [Remove and Re-Create Management Network (vmk0) VMkernel interface using ESXi Command line](https://thevirtualist.org/remove-re-create-management-network-vmkernel-interface-using-esxi-command-line/)
* [Utilize vSphere CLI Commands to Troubleshoot ESXi Network Configurations](https://buildvirtual.net/utilize-vsphere-cli-commands-to-troubleshoot-esxi-network-configurations/)
* [ESXi Commands List - networking commands](https://www.vladan.fr/esxi-commands-list-networking-commands/)


[previous]: {% post_url 2019-10-25-esxi-shell-commands %}
