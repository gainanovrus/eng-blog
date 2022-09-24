---
published: true
layout: single
title: Ansible. Why I use it?
excerpt: >-
  As a sysadmin I often connect to servers that run the same commands by SSH.
  This is could be update packages, edit configures, copy files or some other things.
  And I should do it many times to every server.
  But with tools named configuration management tools this is no need any more.
  Also have not required complex shell-scripts.
  Ansible is the tool that helps me easy maintain many servers and save my time.
categories: devops
tags: ansible devops
toc: true
header:
  teaser: /assets/images/ansible-why-i-use-it.png
  og_image: /assets/images/ansible-why-i-use-it.png
---

As a sysadmin, I often connect to servers that run the same commands by SSH.
This is could be updated packages, edit configures, copy files or some other things.
And I should do it many times to every server.
But with tools named configuration management tools, this is no need anymore.
Also, have not required complex shell-scripts.
Ansible is the tool that helps me easy maintain many servers and save my time.

Like [Chef][chef], [Puppet][puppet], [CFEngine][cfe] Ansible helps for people involved in DevOps automate them regular actions. Ansible has a great opportunities and big community.
And below I describe why to choose it and why you should to try it if you work manually yet.

## 1. Simplicity

The first reason is Ansible easy to use. Nothing additional configures requires on remote machines. Ansible manages machines in an agent-less manner. Thus no additional custom security infrastructure, so it's easy to deploy. If you already have access to the server by ssh (of course you have it) you could use Ansible.

After add the information (name and IP) about servers into an [inventory file][inventory] could possible run ansible commands like this (reboot all servers):
```
ansible all -a "reboot" -s
```

## 2. Human readable

Ansible uses a very simple language ([YAML][yaml]) that allow you to describe your automation jobs in a way that approaches plain English. After shell-scripting uses breckets, quotes, commas and other signs I love it. It uses indentions in programs that beauty and easy to read.
And if you not familiar with any programing language (Python, Ruby, etc..) you also read it with ease.

Next config (in Ansible it names a [playbook][playbook]) will copy local config file (`sshd_config`) of SSH daemon and restart the daemon.

```yaml
- name: Copy SSH configure file
  copy:
    src: remote_ssh/sshd_config
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: 0644

- name: Restart sshd
  service:
    name: sshd
    state: restarted
```

And after run the playbook we have readable output log like this one:
![ansible-yum-all]({{ site.baseurl }}/assets/images/ansible/ansible-yum-all.jpg){: .align-center}

## 3. Many modules

Ansible contains a giant toolbox of built-in [modules][modules], well over 750 of them.
Modules are discrete units of code that can be used from the command line or in a playbook task.
Also it possible to write own modules or execute any command from console in [shell][shell].

Executing a command that creating a replication slot on a remote host with psql console utility.

```yaml
- name: create replication slot
  shell: |
    psql -U postgres -c "SELECT * FROM pg_create_physical_replication_slot('{% raw %}{{ replica_host }}{% endraw %}');"
```

## 4. Rich documentation

I enjoy reading the documentation of Ansible.
All functions are understanding quickly and better with many examples in the docs.
And if I have a question [StackOverflow][stackoverflow] have an answer on it often.


Thus, with this instrument my work was better and I can easily deploy many tasks on servers and repeat it many times with minimal additions.

## Additional information

* [Ansible Documentation][docs] - Covers all Ansible options in depth. There are few open source projects with documentation as clear and thorough.
* [Ansible Glossary][glossary] - If there is ever a term in this book you don't seem to fully understand, check the glossary.
* [Getting Started with Ansible][getting-started] - A simple guide to Ansible's community and resources.
* [Ansible for DevOps][book] - this book helps reach an expert level of Ansible using.
* [Ansible - A Beginner's Tutorial][bens-lessons] - youtube lessons about the basics of Ansible.

[bens-lessons]: https://www.youtube.com/playlist?list=PLFiccIuLB0OiWh7cbryhCaGPoqjQ62NpU
[inventory]: https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-inventory
[modules]: https://docs.ansible.com/ansible/latest/modules/modules_by_category.html
[playbook]: https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-playbooks
[getting-started]: https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html
[docs]: https://docs.ansible.com/
[glossary]: https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html
[cfe]: http://cfengine.com/
[puppet]: http://puppetlabs.com/
[chef]: http://www.getchef.com/chef/
[shell]: https://docs.ansible.com/ansible/2.5/modules/shell_module.html#shell-module
[stackoverflow]: https://stackoverflow.com/questions/tagged/ansible
[yaml]: https://wikipedia.org/wiki/YAML
[book]: https://leanpub.com/ansible-for-devops
