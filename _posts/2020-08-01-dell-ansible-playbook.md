---
published: true
layout: single
title: DELL. Configure Dell 10 gigabit switch with Ansible
excerpt: >-
  Use Ansible playbooks to easy configure Dell Networking OS9 system.
  The post contains many practical examples of using dellos9 modules
categories: devops
tags: dell networks ansible devops
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
---

Today I would like to talk how to easy configure a network switch with Ansible.
In the past when no one listens about "Infrastructure as a Code" (IaaC) methodology we setuping our devices throw console.
We type command by command successively. We copy configure scripts and edit them on device. It's good, but when you configure many devices you often will mistakes.

Today with a very beautiful utility as Ansible. We could automate many actions, that we make manually earlier.
Futhermore we can configure a network switches like Dell and Cisco too. Ansible has a specific modules for these things.

If you already familiar with Ansible you will see how it come easy.
Soo in this post I want to share my experience about it. It will include many practical examples.
All code sketches has been tested by me on [Dell S4048][s4048-specs] 10-gigabit switch.  

## Prerequisites

These examples requires the following:
* Ansible 2.5 (or higher) installed. See Installing Ansible for more information.
* Dell OS9 platform device.
* Basic understanding of YAML Syntax.
* Basic Linux command line use.
* Basic knowledge of network switch & router configurations.

You should have enabled SSH-server. See my [previous post][ssh-server] about configuring it.
It is not difficult.
Also you should append the connection rate by SSH on the network device.
Use the command below to configure the maximum number of incoming SSH connections per minute.
```
# ip ssh connection-rate-limit 60
```

## Tested Environment

* Dell Switch S4048 (OS9 Network System v.9.14)
* Ansible 2.9.11 with Python 3.8 runs on MacOS

## Inventory

All Ansible tasks started with adding a new host into [Inventory file](https://gainanov.pro/eng-blog/devops/ansible-architecture/#1-inventory).

I prefer to use inventory stored in YAML format. This is mine (stored in `hosts` file):
```yaml
...
  hosts:
    switch10g_new:
      ansible_host: 192.168.1.10
      ansible_user: admin
      ansible_network_os: dellos9
```

This piece has told Ansible that the host named `switch10g_new` should be accessible by selected IP and user name.

Before continue writing a playbook try to connect with your credentials to the device.
You should connecting throw ssh with command (`ssh admin@192.168.1.10`) from your local device without any problem.

## Playbook

My playbook (saved on `dell-switch-playbook.yml` file) consists of the next rows:
```yaml
- hosts:
    - switch10g_new
  gather_facts: no
  connection: local
  roles:
    - role: dell
```

I have just run a role named `dell`. But in the real life do not use this name for the role.

The role `dell` include many tasks.

## Tasks

Let's run some simple tasks. Open Ansible documentation and found a part about network modules.

Modules for [Dell OS9 series](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#dellos9) devices includes these three modules:
* `dellos9_command` – Run commands on remote devices running Dell OS9
* `dellos9_config` – Manage Dell EMC Networking OS9 configuration sections
* `dellos9_facts` – Collect facts from remote devices running Dell EMC Networking OS9

Try to use them and see that we can make with. I will start from the end of the list.

### dellos9_facts

This [module](https://docs.ansible.com/ansible/latest/modules/dellos9_facts_module.html#dellos9-facts-module) collect a base set of device facts from a remote device. All fact keys prepends with `ansible_net_<fact>`.


* Task to see on your local console the facts of remote device

```yaml
- name: Collect all facts from the device
  dellos9_facts:
    # all, hardware, config, and interfaces
    # '!' to specify that a specific subset should not be collected.
    # gather_subset: ["all"]
    # gather_subset: ["config", "hardware"]
    gather_subset: ["all", "!interfaces"]
  register: dellos9_facts

- name: Show dellos9_facts
  debug:
    var: dellos9_facts
```

  * The result of the command will presented below:
```
ok: [switch10g_new] => {
    "dellos9_facts": {
        "ansible_facts": {
            "ansible_net_config": "Current Configuration ...",
            "ansible_net_filesystems": [
                "flash",
                "fcmfs",
                "nfsmount",
                "ftp",
                "tftp",
                "scp",
                "http",
                "https"
            ],
            "ansible_net_gather_subset": [
                "hardware",
                "default",
                "config"
            ],
            "ansible_net_hostname": "DellEMC",
            "ansible_net_image": "system://A",
            "ansible_net_memfree_mb": 3123970,
            "ansible_net_memtotal_mb": 3126272,
            "ansible_net_model": "S4048-ON ",
            "ansible_net_serialnum": "NA",
            "ansible_net_version": "9.14(2.7)"
        },
        "changed": false,
        "failed": false
    }
}
```

* Task related with previous to save a switch configuration to your local directory `files/`

```yaml
- name: Save an active config from the device to localhost
  vars:
    running_config: "{% raw %}{{ dellos9_facts['ansible_facts']['ansible_net_config'] }}{% endraw %}"
  local_action:
    module: copy
    content: "{% raw %}{{ running_config }}{% endraw %}"
    dest: "files/running_config_{% raw %}{{ inventory_hostname }}{% endraw %}"
```

### dellos9_command

The module [`dellos9_command`](https://docs.ansible.com/ansible/latest/modules/dellos9_command_module.html#dellos9-command-module) sends commands to the host and returns the results read from the device.
You should know that this module does not support running commands in CONF mode.

It useful to show something about your device.

* Task for getting a device version

```yaml
- name: Detect a device version
  dellos9_command:
    commands: show version
  register: show_ver

- name: show version command output
  debug:
    var: show_ver.stdout_lines
```

* The result will be like:

```
TASK [dell : show version command output]***************************************
ok: [switch10g_new] => {
    "show_ver.stdout_lines": [
        [
            "Dell EMC Real Time Operating System Software",
            "Dell EMC Operating System Version:  2.0",
            "Dell EMC Application Software Version:  9.14(2.7)",
            "Dell EMC Networking OS uptime is 56 minute(s)",
            "",
            "System Type: S4048-ON ",
            "",
            "8G bytes of boot flash memory.",
            "",
            "  1 54-port TE/FG (SK-ON)",
            " 48 Ten GigabitEthernet/IEEE 802.3 interface(s)",
            "  6 Forty GigabitEthernet/IEEE 802.3 interface(s)"
        ]
    ]
}
```  

* Or task to see a device time

```yaml
- name: Get device clock
  dellos9_command:
    commands:
      - show clock
  register: show_clock
- debug: var=show_clock.stdout
```

Now we ready using a module that will change a state, that will edit a configuration of the device

### dellos9_config

The module [`dellos9_config`](https://docs.ansible.com/ansible/latest/modules/dellos9_config_module.html#dellos9-config-module) writes a blocks of code into your device configuration. It has `before` and `after` parameters to exactly point the place where your code should saved.

* Task for changing an interface configuration

```yaml
- name: "Enable interface and dhcp client on port te1/1"
  dellos9_config:
    lines:
      - 'ip address dhcp'
      - 'no shutdown'
    parents:
      - 'interface TenGigabitEthernet 1/1'
  register: enable_interface
- debug: var=enable_interface
```

* The result of the task:

```
TASK [dell : Enable interface and dhcp client on port te1/1] *******************
changed: [switch10g_new]

TASK [dell : debug] ************************************************************
ok: [switch10g_new] => {
    "enable_interface": {
        "changed": true,
        "commands": [
            "interface TenGigabitEthernet 1/1",
            "ip address dhcp",
            "no shutdown"
        ],
        "failed": false,
        "saved": false,
        "updates": [
            "interface TenGigabitEthernet 1/1",
            "ip address dhcp",
            "no shutdown"
        ]
    }
}
```

Also it useful to run some actions that acquire to input answer to a prompt.

* Task for saving running_config to startup_config

```yaml
- name: "Copy running-config to startup-config for the Dell EMC OS9 Device"
  dellos9_config:
    lines:
      - command: do copy running-config startup-config
        prompt: '\[confirm yes/no\]:\s?$'
        answer: "yes"
```

## Galaxy roles

Dell EMC Open-Source Community in [Github](https://github.com/Dell-Networking) provides many Galaxy roles.
But it is just a wrapper on the modules explained earlier.
It provide a good interface to edit a network configure as structured YAML variables stored in files.
See a full list of the Galaxy roles [here](https://ansible-dellos-docs.readthedocs.io/en/latest/roles.html)

To use these Dell Galaxy roles you need to download them. Define all roles that you want download into a file like [this mine][dellemc_roles]. I prefer to save an all external roles into a local directory `./ansible_roles`.
So this file I will save into this directory. After that run a command to download them:
```
ansible-galaxy install -r ansible-roles/dellemc_roles.yml
```

Next write a new playbook to run some tasks defined in the downloaded Galaxy roles.

* Playbook - `dell-switch-configure.yml`

```yaml
- hosts:
    - switch10g_new
  gather_facts: no
  # become: yes
  # become_method: enable
  connection: local
  # connection: network_cli

  roles:
    - role: ansible-roles/Dell-Networking.dellos-system
    - role: ansible-roles/Dell-Networking.dellos-interface
    - role: ansible-roles/Dell-Networking.dellos-vlan
```

* Host variables - `host_vars/switch10g_new.yaml`

```yaml
# https://galaxy.ansible.com/Dell-Networking/dellos-system
dellos_system:
  hostname: "{% raw %}{{ inventory_hostname }}{% endraw %}"

# https://galaxy.ansible.com/Dell-Networking/dellos-interface
dellos_interface:
    # interface TenGigabitEthernet 1/22
    #   description T-GPU1
    #   no ip address
    #   portmode hybrid
    #   switchport
    #   no shutdown
    TenGigabitEthernet 1/22:
      desc: "T-GPU1"
      ip_and_mask:  # set up `no ip address`
      portmode: hybrid  # pass both untagged and tagged VLANs
      switchport: true
      # ip_type_dynamic: true # Configures IP address DHCP if set to true (ip_and_mask is ignored if set to true)
      # ip_and_mask: 192.168.23.22/24 # Configures the specified IP address (192.168.11.1/24 format)
      admin: up  # enables the interface

    # interface Port-channel 1
    #   no ip address
    #   switchport
    #   no shutdown
    port-channel 1:
      ip_and_mask:
      switchport: true
      admin: up

    # interface Vlan 3
    #  description C-DATA
    #  no ip address
    #  tagged Port-channel 1
    #  untagged TenGigabitEthernet 1/22
    #  no shutdown
    vlan 3:
      desc: "C-DATA"
      ip_and_mask:
      # to configure vlan use dellos-vlan role (see next)
      admin: up

# https://galaxy.ansible.com/Dell-Networking/dellos-vlan
dellos_vlan:
    vlan 3:
      description: "C-DATA"
      tagged_members:
        - port: "Port-channel 1"
          state: present
      untagged_members:
        - port: "TenGigabitEthernet 1/22"
          state: present
      state: present
```

* Explanation of tasks
This playbook set up a hostname of a device as inventory hostname. After that it configures interfaces `te1/22`, `po1` and `vlan 3`. If it does not existing it will create. Next we add members to created `vlan 3`.

This YAML-file easy to read and it explains capability and power of these galaxy roles. Read more about them in [Documentation pages](https://ansible-dellos-docs.readthedocs.io/en/latest/roles.html).

## Conclusion

In this post I explain the configuration process of network device without console.
With Ansible you can make changes with many devices in one time.
File of configuration easy to read (because it stored in YAML). All tasks could be saved into Git repository.
Futhermore it could possible to update and apply changes with your CI/CD tool.

Also I save all sources into [GitHub][github]. You can read all practical examples and clone it.

## Additional information

* [Dell-Networking page on Ansible Galaxy](https://galaxy.ansible.com/Dell-Networking) - where placed all galaxy roles written by Community
* [Dell Command Line Reference Guide for the S4048–ON System](https://ansible-dellos-docs.readthedocs.io/en/latest/intro.html) - Dell documentation explains Dell EMC Ansible integration
* [Dellos9 page on Ansible Modules](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#dellos9) - read more about explained modules
* [GitHub - Dell-Networking - ansible-dellos-examples](https://github.com/Dell-Networking/ansible-dellos-examples) - Sample Ansible playbooks to understand how the Dell EMC Networking Anisble Module works.


[s4048-specs]: https://i.dell.com/sites/doccontent/shared-content/data-sheets/en/Documents/Dell-EMC-Networking-S4048-ON-Spec-Sheet.pdf
[ssh-server]: {% post_url 2020-07-19-dell-ssh-server-configure %}
[dellemc_roles]: {{ site.baseurl }}/assets/files/dell/dellemc_roles.yml
[github]: https://github.com/GRomR1/dell-ansible-playbook
