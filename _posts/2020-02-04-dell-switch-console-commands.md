---
published: true
layout: single
title: DELL. CLI commands
excerpt: >-
  Here is a list of basic CLI commands which will help you manage your Dell PowerConnect series switches.
categories: sysad
tags: dell cisco networks
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
---

I recently have experience in configuring a DELL PowerSwitch [S4048-ON][s4048-specs].
It is 10/40GbE top-of-rack (ToR) switch built for applications in high performance data center and computing environments.

Dell’s OS9 (S & Z series) switches have a minimal differences from Cisco IOS. Many commands is similar.
In this quick reference I want to share a basic commands you’ll need to rely on when handling various configuration and troubleshooting tasks. The output of something commands will included.

So, here is a list of basic CLI commands which will help you manage your DELL PowerConnect series switches.


## The “?”
You can use the command `?` in many ways. First, use it when you don’t know what command to type.
For example, type `?` At the command line for a list of all possible commands.

```
SWP_UP_10G>?
clock                           Manage the system clock                 
crypto                          Crypto Commands                         
disable                         Turn off privileged commands            
dot1x                           802.1x commands                         
enable                          Turn on privileged commands             
ethernet                        Ethernet commands                       
exit                            Exit from the EXEC                      
fsck                            Filesystem check utility                
ip                              Global IP subcommands                   
ipv6                            Global IPv6 subcommands                 
monitor                         Monitoring feature                      
mtrace                          Trace reverse multicast path from destination to source
package                         Automation package related commands     
ping                            Send echo messages                      
quit                            Exit from the EXEC                      
show                            Show running system information         
ssh                             Open a SSH connection                   
ssh-peer-stack-unit             Open a SSH connection to the peer stack-unit
start                           Start shell                             
support-assist                  SupportAssist service                   
telnet                          Open a telnet connection                
telnet-peer-stack-unit          Open a telnet connection to the peer stack-unit
terminal                        Set terminal line parameters            
--More--
```

You can also use `?` When don’t know what a command’s next parameter should be. For example, you might type `interface`?
If the router requires no other parameters for the command, the router will offer `CR` as the only option.
Type a `?` at the prompt or after a keyword. There must always be a space before the `?`.

```
SWP_UP_10G(conf)#interface ?
fortyGigE               FortyGigabit Ethernet interface         
gigabitethernet         Gigabit Ethernet interface              
group                   Configure interface group               
loopback                Loopback interface                      
managementethernet      Management Ethernet interface           
null                    Null interface                          
port-channel            Port-channel interface                  
range                   Configure interface range               
tengigabitethernet      TenGigabit Ethernet interface           
tunnel                  Tunnel interface                        
vlan                    VLAN interface   
```

To see all commands that start with a particular letter. For example, show `c?`

```
SWP_UP_10G#c?
cd
clear
clock
configure
copy
crypto
```

Will return a list of commands that start with the letter `c`.

## show running-config

The `show running-config` command shows the router, switch, or firewall’s current configuration.
The running-configuration is the config that is in the router’s memory.
You change this config when you make changes to the router.
Keep in mind that that config is not saved until you do copy it to startup.

```
SWP_UP_10G#show running-config
Current Configuration ...
! Version 9.11(0.0P3)
! Last configuration change at Mon Feb  3 18:09:01 2020 by admin
!
boot system stack-unit 1 primary system: A:
boot system stack-unit 1 secondary system: B:
boot system stack-unit 1 default system: A:
!
logging coredump stack-unit  1
logging coredump stack-unit  2
logging coredump stack-unit  3
logging coredump stack-unit  4
logging coredump stack-unit  5
logging coredump stack-unit  6
!
hostname SWP_UP_10G
!
--More--
```

## copy running-config startup-config

This command will save the configuration that is currently being modified (in RAM),
also known as the running-configuration, to the nonvolatile RAM (NVRAM).
If the power is lost, the NVRAM will preserve this configuration.
In other words, if you edit the router’s configuration, don’t use this command and reboot the router–those changes will be lost.
This command can be abbreviated `copy run start`.

```
SWP_UP_10G#copy running-config startup-config
```

## command modes

The DELL switch like Cisco has different command modes with distinctive prompts.
These modes execute different switch commands. Each mode has a set of specific commands.

To navigate and launch various CLI modes, use specific commands

```
Prompt         CLI Command Mode
Dell>          EXEC
Dell#          EXEC Privilege
Dell(conf)#    CONFIGURATION
```

Enter EXEC Privilege mode or any other privilege level configured - `enable`.
After entering this command, you may need to enter a password.

```
SWP_UP_10G>enable
Password:
SWP_UP_10G#
```

Enter CONFIGURATION mode from EXEC Privilege mode - `configure`.
```
SWP_UP_10G#configure
SWP_UP_10G(conf)#
```

Return to the lower command mode - `exit`.
```
SWP_UP_10G(conf)#exit
SWP_UP_10G#
```

## command history

Display a buffered log of all commands all users enter along with a time stamp
```
#show command-history
```

## device version

Some useful information about Software version and device.
Display the current Dell Networking Operating System (OS) version information on the system - `show version`.

```
SWP_UP_10G#show version
Dell Real Time Operating System Software
Dell Operating System Version:  2.0
Dell Application Software Version:  9.11(0.0P3)
Copyright (c) 1999-2017 by Dell Inc. All Rights Reserved.
Build Time: Fri Jan 27 10:54:03 2017
Build Path: /build/build02/SW/SRC
Dell Networking OS uptime is 6 week(s), 1 day(s), 23 hour(s), 13 minute(s)

System image file is "system://A"

System Type: S4048-ON
Control Processor: Intel Rangeley with 3 Gbytes (3201302528 bytes) of memory, core(s) 2.

8G bytes of boot flash memory.

  1 54-port TE/FG (SK-ON)
 48 Ten GigabitEthernet/IEEE 802.3 interface(s)
  6 Forty GigabitEthernet/IEEE 802.3 interface(s)
```

## show interfaces

Display information on a specific physical interface or virtual interface.
```
SWP_UP_10G#show interfaces tengigabitethernet 1/1
TenGigabitEthernet 1/1 is up, line protocol is up
Description:
Hardware is DellEth, address is 14:18:77:80:11:00
    Current address is 14:18:77:80:11:00
Pluggable media present, SFP+ type is 10GBASE-SR
    Medium is MultiRate, Wavelength is 850nm
    SFP+ receive power reading is -2.7344dBm
    SFP+ transmit power reading is -2.2105dBm
Interface index is 2097156
Internet address is not set
Mode of IPv4 Address Assignment : NONE
DHCP Client-ID :1418778022e1
MTU 1554 bytes, IP MTU 1500 bytes
LineSpeed 10000 Mbit
Flowcontrol rx off tx off
ARP type: ARPA, ARP Timeout 04:00:00
Last clearing of "show interface" counters 6w1d23h
Queueing strategy: fifo
Input Statistics:
     12063517275 packets, 16273367119685 bytes
     452219597 64-byte pkts, 3344945 over 64-byte pkts, 123343574 over 127-byte pkts
     994638054 over 255-byte pkts, 4755930 over 511-byte pkts, 10485215175 over 1023-byte pkts
     0 Multicasts, 6533 Broadcasts, 12063510742 Unicasts
```


To display status information on a specific interface only,
display a summary of interface information or specify a stack-unit slot and interface.

```
SWP_UP_10G#show interfaces status
Port                 Description  Status Speed        Duplex Vlan  
Te 1/1                            Up     10000 Mbit   Full   3
Te 1/2                            Up     10000 Mbit   Full   3
Te 1/3                            Up     10000 Mbit   Full   3
Te 1/4                            Up     10000 Mbit   Full   3
Te 1/5                            Up     10000 Mbit   Full   3
Te 1/6                            Up     10000 Mbit   Full   3
Te 1/7                            Up     10000 Mbit   Full   3
--More--
```

## monitor interface

Monitor counters on a single interface or all interfaces.
The screen is refreshed every five seconds and the CLI prompt disappears.
```
SWP_UP_10G#monitor interface te 1/1

 Interface: Te 1/1, Enabled, Link is Up, Linespeed is 10000 Mbit

  Traffic statistics:              Current           Rate                Delta
         Input bytes:       16273367122653          0 Bps                    0
        Output bytes:       13745541955640          0 Bps                    0
       Input packets:          12063517303          0 pps                    0
      Output packets:          10342234622          0 pps                    0
         64B packets:            452219597          0 pps                    0
    Over 64B packets:              3344973          0 pps                    0
   Over 127B packets:            123343574          0 pps                    0
   Over 255B packets:            994638054          0 pps                    0
   Over 511B packets:              4755930          0 pps                    0
  Over 1023B packets:          10485215175          0 pps                    0
 Error statistics:
     Input underruns:                    0          0 pps                    0
        Input giants:                    0          0 pps                    0
     Input throttles:                    0          0 pps                    0
           Input CRC:                    0          0 pps                    0
   Input IP checksum:                    0          0 pps                    0
       Input overrun:                    0          0 pps                    0
    Output underruns:                    0          0 pps                    0
    Output throttles:                    0          0 pps                    0

       m - Change mode                            c - Clear screen
       l - Page up                                a - Page down
       T - Increase refresh interval              t - Decrease refresh interval
       q - Quit
```

## show ip interface

The command provides useful information about the configuration and status of the IP protocol and its services, on all interfaces, including their IP address, Layer 2 status, and Layer 3 status.
```
SWP_UP_10G#show ip interface
TenGigabitEthernet 1/1 is up, line protocol is up
Description:
Internet address is not set
IP MTU is 1500 bytes
Directed broadcast forwarding is disabled
Proxy ARP is enabled
Split Horizon is enabled
Poison Reverse is disabled
ICMP redirects are not sent
ICMP unreachables are not sent
IP unicast RPF check is not supported

--More--
```

## configure interface

Configure a physical or virtual interface on the switch.
```
SWP_UP_10G#conf
SWP_UP_10G(conf)#interface tengigabitethernet 1/1
SWP_UP_10G(conf-if-te-1/1)#
```

Before configuring should be useful view a current settings for the interface
```
SWP_UP_10G(conf-if-te-1/1)#show config
!
interface TenGigabitEthernet 1/1
 no ip address
 switchport
 no shutdown
```

## no shutdown

To disable, delete or return to default values, use the `no` form of the commands.
The `no shutdown` command enables an interface (brings it up).
This command must be used in `interface` configuration mode.
It is useful for new interfaces and for troubleshooting.
When you’re having trouble with an interface, you may want to try a `shut` and `no shut`.

## arp & mac table

For displaying the ARP table use `show arp` or `show arp interface te 1/1` for specific interface.
```
SWP_UP_10G#show arp

Protocol    Address         Age(min)  Hardware Address    Interface      VLAN             CPU
---------------------------------------------------------------------------------------------
Internet    172.16.1.1            6   64:00:6a:d8:00:00   Po 1           Vl 4             CP
Internet    172.16.1.2            -   14:18:77:80:00:01        -         Vl 4             CP
Internet    172.16.1.3            0   a0:36:9f:d8:00:02   Po 1           Vl 4             CP
```

Display the MAC address table.
```
SW_DWN_1G#show mac-address-table
Codes: *N - VLT Peer Synced MAC
*I - Internal MAC Address used for Inter Process Communication

VlanId     Mac Address           Type          Interface        State
 3      00:0c:29:41:01:00       Dynamic         Gi 1/9          Active
 3      00:0c:29:dd:01:00       Dynamic         Gi 1/9          Active
 3      00:1e:67:f0:02:00       Dynamic         Gi 1/40         Active
--More--
```

## show vlan

Display the current VLAN configurations on the switch - `show vlan`.
```
SWP_UP_10G#show vlan

Codes: * - Default VLAN, G - GVRP VLANs, R - Remote Port Mirroring VLANs, P - Primary, C - Community, I - Isolated
       O - Openflow, Vx - Vxlan
Q: U - Untagged, T - Tagged
   x - Dot1x untagged, X - Dot1x tagged
   o - OpenFlow untagged, O - OpenFlow tagged
   G - GVRP tagged, M - Vlan-stack
   i - Internal untagged, I - Internal tagged, v - VLT untagged, V - VLT tagged

    NUM    Status    Description                     Q Ports
*   1      Inactive                                  
    3      Active    C-MGMT                          T Po1(Te 1/47-1/48)
    4      Active    C-DATA                          T Po1(Te 1/47-1/48)
                                                     U Te 1/1-1/28,1/30-1/46
```

## current time

Display the current clock settings.
```
SWP_UP_10G#show clock detail
18:04:16.792 UTC Mon Feb 3 2020
Time source is RTC hardware
```

## Conclusion

In summary, the key differences between Dell and Cisco are the approach they take to where the untagging and tagging happens (Cisco: interface, Dell: VLAN), how the trunks allow VLANs and how dell OS9 does not have any form of Spanning tree by default.
But other than that the CLI commands are very similar and any engineer familiar with Cisco will no problem using Dell OS9.


## Additional information

* [Cisco Commands Cheat Sheet](https://www.netwrix.com/cisco_commands_cheat_sheet.html) - cisco commands in one page
* [Dell Command Line Reference Guide for the S4048–ON System 9.14.0.0](https://www.dell.com/support/manuals/us/en/19/force10-s4048-on/s4048-on-9.14.0.0-cli/about-this-guide) - dell man page included full command list
* [DELL EMC NETWORKING S4048-ON SWITCH](https://i.dell.com/sites/doccontent/shared-content/data-sheets/en/Documents/Dell-EMC-Networking-S4048-ON-Spec-Sheet.pdf) - dell spec sheet for S4048–ON switch


[s4048-specs]: https://i.dell.com/sites/doccontent/shared-content/data-sheets/en/Documents/Dell-EMC-Networking-S4048-ON-Spec-Sheet.pdf
