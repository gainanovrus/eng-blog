---
published: true
layout: single
title: ESXi. The best way to build VPN tunnel with pfSense
excerpt: >-
  ESXi is good and reliable tool to run VMs on bare metal. It is configuring by webGUI.
  But the host with ESXi can be accessed only through NAT without any public IP or ports.
  This post shows how to create VPN tunnel with pfSense as VM on ESXi.
categories: sysad
tags: esxi pfsense openvpn
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2020-02-21
---

## Introduction

[ESXi](https://www.vmware.com/products/esxi-and-esx.html) is a type-1 hypervisor, meaning it runs directly on system hardware without the need for an operating system (OS). Type-1 hypervisors are also referred to as bare-metal hypervisors because they run directly on hardware.

Also VMware vSphere Hypervisor is a free product and require a free license for the usage.

[pfSense](https://www.pfsense.org/) is a free and open source firewall and router that also features unified threat management, load balancing, multi WAN, and more.

## Environment

In this article I have next hardware and installed software on them:
1. Supermicro server with two network interfaces (1 - connected to Internet, 2 - to LAN)
2. VMware ESXi 6.5 U3

## Installation

So I have an installed ESXi hypervisor on my bare metal server.
I get access to ESXi through WebGUI by local IP that ESXI have got from DHCP Server.
I just open a browser and insert `10.16.65.12` in address line.

### Prepare network

So after first boot I see a next window.

![001_not_join_to_improvement_programm]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/001_not_join_to_improvement_programm.png){: .align-center}

Set up a new vSwitch. It will use for access for created virtual machines.

![002_add_lan_vswitch]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/002_add_lan_vswitch.png){: .align-center}

Set an obvious name and select second network interface. In vmnic1 connected my laptop.

![003_create_vswitch_lan]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/003_create_vswitch_lan.png){: .align-center}

Now open **Port groups** tab.
![004_add_new_portgroup]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/004_add_new_portgroup.png){: .align-center}

Add new port group that will used to connect our software firewall (pfSense).
Name it **External** as straight external network.

![005_create_external_port_group]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/005_create_external_port_group.png){: .align-center}

Retry it for creation **Internal** port group.
VMs connected to this port group has access to Internet only through a firewall.

![006_create_internal_port_group]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/006_create_internal_port_group.png){: .align-center}

The created port group by default can be removed.

![007_delete_default_port_group]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/007_delete_default_port_group.png){: .align-center}

Allow to delete it.
![008_alllow_delete_default_port_group]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/008_alllow_delete_default_port_group.png){: .align-center}

View a result of these actions.
![009_view_result_port_group_tab]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/009_view_result_port_group_tab.png){: .align-center}

Add new VMKernel interface.
![010_add_new_vm_interface]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/010_add_new_vm_interface.png){: .align-center}

It needs to get access to ESXi WebGUI from our local network (`192.168.112.1/24`)
![011_configure_new_vmnic]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/011_configure_new_vmnic.png){: .align-center}

After that we will have two management interfaces. An old one will be removed at the end.
![012_view_results]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/012_view_results.png){: .align-center}

### Upload ISO

Now we can prepare to make pfSense on ESXi. First of all add an installation ISO image.
![013_open_datastore_page]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/013_open_datastore_page.png){: .align-center}

Create an `ISOs` directory.
![014_create_iso_directory]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/014_create_iso_directory.png){: .align-center}
![015_create_iso_directory]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/015_create_iso_directory.png){: .align-center}

Add a [downloaded](https://www.pfsense.org/download/) ISO with pfSense.
![016_add_pfsense_image]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/016_add_pfsense_image.png){: .align-center}

Wait some time. After that you will see the pfsense image in ESXi.
![017_content_of_iso_directory]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/017_content_of_iso_directory.png){: .align-center}

### Create VM with pfSense

Open **Virtual machines** page in ESXi. Click **Create new VM**.
![018_open_vm_page]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/018_open_vm_page.png){: .align-center}

Yes, create a new virtual machine.
![019_create_new_vm]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/019_create_new_vm.png){: .align-center}

Set name. Select type of VM.
![020_set_name]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/020_set_name.png){: .align-center}

Select where VM will be installed.
![021_vm_place_to_store_files]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/021_vm_place_to_store_files.png){: .align-center}

Add more space of VM disk. Add second network adapter. Join them with Internal and External PG.
![022_configuration_vm]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/022_configuration_vm.png){: .align-center}

Select an installation ISO image uploaded earlier.
![023_select_iso_image]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/023_select_iso_image.png){: .align-center}

Check the result of configuring VM.
![024_check_result_config]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/024_check_result_config.png){: .align-center}

### Install pfSense

After that the VM has to be created. And we can click to button **Power on** for start installation.
![025_run_created_vm]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/025_run_created_vm.png){: .align-center}

In many of screens I've just clicked **Next** and haven't change anything.
![026_accept_copyright]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/026_accept_copyright.png){: .align-center}
![027_install_pfsense]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/027_install_pfsense.png){: .align-center}
![028_use_default_keymap]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/028_use_default_keymap.png){: .align-center}
![029_use_default_partition]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/029_use_default_partition.png){: .align-center}
![030_wait_processing]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/030_wait_processing.png){: .align-center}
![031_skip_manual]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/031_skip_manual.png){: .align-center}
![032_reboot_system]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/032_reboot_system.png){: .align-center}

### Configure pfSense

After reboot system will offer to setup network interfaces
![033_setup_interfaces]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/033_setup_interfaces.png){: .align-center}

Open VM setting in ESXi to be right in this choose. And view network interface MACs.
![034_view_mac_interfaces]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/034_view_mac_interfaces.png){: .align-center}

The WAN interface should be the virtual interface connected to External port group.
![035_select_wan]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/035_select_wan.png){: .align-center}

Another interface will be LAN.
![036_select_lan]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/036_select_lan.png){: .align-center}

Next see the results and continue the configuring in browser by `192.168.1.1`.
![037_view_results]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/037_view_results.png){: .align-center}

Open browser and login in pfSense web configurator with default credentials (`admin`:`pfsense`)
![038_pfsense_first_login]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/038_pfsense_first_login.png){: .align-center}

Skip welcome page. And read Netgear support information if you need.
![039_view_welcome_page]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/039_view_welcome_page.png){: .align-center}
![040_read_netgear_support]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/040_read_netgear_support.png){: .align-center}

Set hostname, NTP, timezone.
![041_set_hostname]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/041_set_hostname.png){: .align-center}
![042_set_ntp]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/042_set_ntp.png){: .align-center}

Configure WAN interface. My instance get IP by DHCP.
![043_configure_wan]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/043_configure_wan.png){: .align-center}

Configure LAN network. Set a new network address `192.168.112.1/24`.
Remember WebGUI will have a new IP after reboot.
![043_configure_lan]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/043_configure_lan.png){: .align-center}

Set new admin password.
![045_set_admin_password]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/045_set_admin_password.png){: .align-center}

Click **Reload** button.
![046_reload_system]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/046_reload_system.png){: .align-center}
![047_finish_configuration]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/047_finish_configuration.png){: .align-center}

Open new browser tab with new WebGUI instance accessed by IP `192.168.112.1`.
![048_dashboard_pfsense]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/048_dashboard_pfsense.png){: .align-center}

### Setup OpenVPN

VMs and ESXi don't have any public IP. But we can get full access to them if we will install OpenVPN.
I have another pfSense server with public IP. It will be **OpenVPN Server**.
This ESXi host and pfSense instance will connect to server and share their LAN (`192.168.112.1`).
We name pfSense that we configurating in this article as **OpenVPN Client**.
So for this purposes we need to do some manipulation with **OpenVPN Server**.

Open VPN page in WebGUI. Click **Add** button and create new OpenVPN Server.
Select `Peer to peer (shared key)` server mode. Set listen port `1201` (it can be any other).
Edit another options if it needs.
![049_create_openvpnserver]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/049_create_openvpnserver.png){: .align-center}

After we click **Save** the shared key of the server will be generated.
Open created server.  
![050_open_created_server]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/050_open_created_server.png){: .align-center}

Copy generated key. It will need to connect client to this server.
![051_copy_shared_key]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/051_copy_shared_key.png){: .align-center}

Now open WebGUI of another pfSense that will **OpenVPN Client**. Open VPN page. Click **Add**.
![052_add_new_client]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/052_add_new_client.png){: .align-center}

Setup OpenVPN client by my example. I highlight options that I modified.
![053_create_openvpn_client]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/053_create_openvpn_client.png){: .align-center}

Click save and apply changes.
![054_after_save_client]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/054_after_save_client.png){: .align-center}

Open status page of VPN connection
![055_check_status_vpn]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/055_check_status_vpn.png){: .align-center}

The status will be `up`.
![056_status_ok]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/056_status_ok.png){: .align-center}

Open Firewall configuration page. Follow to **OpenVPN** tab.
![057_open_firewall_rules]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/057_open_firewall_rules.png){: .align-center}

Create new rule with allow traffic through this adapter to LAN net.
![058_pass_traffic_by_openvpn]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/058_pass_traffic_by_openvpn.png){: .align-center}

Create another pass rule for LAN interface allows anything.
![059_pass_traffic_by_lan]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/059_pass_traffic_by_lan.png){: .align-center}

Apply the changes.
![060_apply_changes]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/060_apply_changes.png){: .align-center}

Now we can connect to pfSense (that configuring as OpenVPN client) by local IP (`192.168.112.1`).
ESXi server can be hosted in any place. If it connected to Internet also through NAT.
OpenVPN will auto connect to our public VPN server.  

But to access to ESXi webGUI we should make some addition.

### Setup ESXi on private LAN

We have an additional management virtual interface that connected to vSwitchLAN - `vmk1`. We made it before.
![061_open_vmk1_interface]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/061_open_vmk1_interface.png){: .align-center}

Open the interface, and make sure that it has a right IP adress - `192.168.112.1`.
![062_check_adapter_ip]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/062_check_adapter_ip.png){: .align-center}

Open TCP/IP stack tab. Select **Default TCP/IP stack**.
![063_open_ip_stack_tab]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/063_open_ip_stack_tab.png){: .align-center}

Click **Edit settings** button.
![064_click_edit_settings]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/064_click_edit_settings.png){: .align-center}

Manually configure it. Set pfSense IP as default gateway.
![065_manually_configure]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/065_manually_configure.png){: .align-center}

We will lose the access to ESXi WebGUI. It's OK. Just open it in new IP - `192.168.112.2`
![066_open_esxi_webgui_by_lan_ip]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/066_open_esxi_webgui_by_lan_ip.png){: .align-center}

Open VMKernel NIC's tab.
![067_open_vmk_tab]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/067_open_vmk_tab.png){: .align-center}

Remove the management interface connected to External network to `vSwitch0`.
![068_remove_external_mgmt_interface]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/068_remove_external_mgmt_interface.png){: .align-center}

Confirm remove.
![069_confirm_remove]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/069_confirm_remove.png){: .align-center}

Now we have access only from OpenVPN tunnel or LAN network.
![070_view_results]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/070_view_results.png){: .align-center}

Also we can delete unused `Management network` port group.
![071_open_port_group_tab]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/071_open_port_group_tab.png){: .align-center}
![072_remove_unused_mgmt_network]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/072_remove_unused_mgmt_network.png){: .align-center}

### Enable autorun pfSense VM

ESXi doesn't run any VMs after power on by default. You should enable this manually.
It's a simple. Open **Virtual machines** page and select `fw`.
![075_select_fw_vm]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/075_select_fw_vm.png){: .align-center}

Enable autostart for selected VM.
![076_enable_autostart]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/076_enable_autostart.png){: .align-center}

### Install VM tools

ESXi will show you the notice message until you install VM tools.
>  VMware Tools is not installed in this virtual machine. VMware Tools allows detailed guest information to be displayed as well as allowing you to perform operations on the guest OS, e.g. graceful shutdown, reboot, etc. You should install VMware Tools.

![077_view_notice_about_vm_tools]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/077_view_notice_about_vm_tools.png){: .align-center}

VM tools is very helpful instrument. And VMWare haven't it for FreeBSD.
But exists [Open VM tools package](https://docs.netgate.com/pfsense/en/latest/packages/open-vm-tools-package.html).
It can be easily installed on pfSense **Package manager** page.
![078_open_package_manager]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/078_open_package_manager.png){: .align-center}

Search and install Open VM tools.
![079_search_open_vm_tools]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/079_search_open_vm_tools.png){: .align-center}

View success result of installation the package.
![080_view_result_of_installation]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/080_view_result_of_installation.png){: .align-center}

Done! Now you can create any virtual machines on ESXi host and get full access to them by secure VPN tunnel :)

## Some notes

After I install **Open VM Tools** ESXi will show a warning message:
> The configured guest OS (FreeBSD 12 or later versions (64-bit)) for this virtual machine does not match the guest that is currently running (FreeBSD 11 (64-bit)). You should specify the correct guest OS to allow for guest-specific optimizations.

![081_view_warning_about_version_os]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/081_view_warning_about_version_os.png){: .align-center}

To avoid this just change the **Guest OS Version** to `FreeBSD 11 (64-bit)` and will be happy.
![082_change_guest_os_to_freebsd11]({{ site.url }}{{ site.baseurl }}/assets/images/installing-pfsense-on-esxi/082_change_guest_os_to_freebsd11.png){: .align-center}


## Additional information

* [VMware Virtual Networking Concepts](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/techpaper/virtual_networking_concepts.pdf) -
    The information guide to VMware Networks.
