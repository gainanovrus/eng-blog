---
published: true
layout: single
title: Ansible. My preferred modules
excerpt: >-
  Ansible ships with a number of modules (module library) that can be executed directly on remote hosts or through Playbooks. Modules are the ones that do the actual work in Ansible. Ansible provides over 2000 modules. There is less 40 most used from them. I explain about my preferred modules in this post.
categories: devops
tags: ansible devops
toc: true
header:
  teaser: /assets/images/ansible-preferred-modules.png
  og_image: /assets/images/ansible-preferred-modules.png
last_modified_at: 2019-01-21
---

[Modules][all-modules] are the main building blocks of Ansible and are basically reusable scripts that are used by Ansible playbooks.
Ansible comes with a number of reusable modules.
These include functionality for controlling services, software package installation, working with files and directories etc.

In the [previous][getting-started] article was showed about basic actions with Ansible like a execution a task, a playbook.
This one explains modules that I often use in playbooks. It covers over 80% cases of using Ansible.
I want to write it for storing information in one page.
It expected you know how to write and custom Ansible playbook.

For start see the next simple task that used module `reboot` for [rebooting a server][reboot-module]:
```
- name: Reboot a slow machine that might have lots of updates to apply
  reboot:
    reboot_timeout: 3600
```

In there the field `name` is a some description that helps to understand what doing this task.
The keyword `reboot` is a name of used module. And the module could have a parameters that write with new row begins with a indention.
The `reboot_timeout` is a name of parameter that need to setup the maximum seconds to wait for machine to reboot, `3600` - value of the parameter.
Also you could write the parameters in one row (i.e. `reboot: reboot_timeout=3600`) but I don't recommend do it because it has reduce readability.

Ok, now I list the my list of 'must have' modules.

## copy

The [`copy`][copy-module] module copies a file from the local or remote machine to a location on the remote machine.

```
- name: Copying file with owner and permissions
  copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: 0644

- name: Copy modules.d directory to remote hosts
  copy:
    src:  config/filebeat/modules.d/
    dest: /etc/filebeat/modules.d/
    directory_mode: yes
    owner: root
    group: root
    mode:  0644
```

## fetch

The [`fetch`][fetch-module] module works like `copy`, but in reverse.
It is used for fetching files from remote machines and storing them locally in a file tree, organized by hostname.
It works with only ONE file (not directory).

```
- name: Copy file into local storage in /tmp/fetched/host.example.com/tmp/somefile
- fetch:
    src: /tmp/somefile
    dest: /tmp/fetched

- name: Copy ssh_key.pub to master machine
  fetch:
    src: /root/.ssh/id_rsa.pub
    dest: "./"

- name: Copy postgresql config file to local machine
  fetch:
    src: /usr/local/pgsql/data/postgresql.conf
    dest: backup/pgconf/{{ inventory_hostname }}/
    flat: yes
```

## template

Templates are processed by the [Jinja2](http://jinja.pocoo.org/docs/) templating language.
[This module][template-module] like `copy` module - copies files to remote hosts.
But it has provide autocomplete special files - templates, that include variables in them.
For instance the template has contents `This is a test file on {{ ansible_hostname }}`.
After run playbook with `template` task, the file will be copied and included `This is a test file on some_hostname`.

```
- name: Copy a template /mytemplates/foo.j2 to master machine
  template:
    src: /mytemplates/foo.j2
    dest: /etc/file.conf
    owner: root
    group: wheel
    mode: 0644
```

## file

The [`file`][file-module] module creates or removes files, symlinks or directories.
Also, set attributes of files, symlinks or directories.

```
- name: Change file ownership, group and permissions
  file:
    path: /etc/foo.conf
    owner: foo
    group: foo
    mode: 0644

- name: Creates borgmatic directory if it does not exist
  file:
    path: /etc/borgmatic
    state: directory
    mode: 0755
    owner: root
    group: root

- name: Create file /etc/sudoers.d/telegraf
  file:
    path: /etc/sudoers.d/telegraf
    state: touch
    owner: root
    group: root
    mode: "u=rw,g=,o="

- name: Remove file web-master.war
  file:
    path: /foo/bar/web-master.war
    state: absent
```

## synchronize

The `copy` and `fetch` modules copies only the defined files not full directory.
I use the [synchronize][synchronize-module] module for copy directory recursive with files from control to remote machine.
It is a wrapper around `rsync` command. `rsync` must be installed on both the local and remote host.

```
- name: Synchronization of src on the control machine to dest on the remote hosts
  synchronize:
    src: some/relative/path
    dest: /some/absolute/path

# cp -r some/relative/path/* /second/absolute/path/
- name: Synchronize two directories on one remote host
  synchronize:
    src: /first/absolute/path/
    dest: /second/absolute/path/
  delegate_to: "{{ inventory_hostname }}"

- name: Synchronization of two paths both on the control machine
  synchronize:
    src: some/relative/path
    dest: /some/absolute/path
  delegate_to: localhost
```

## command

The [`command`][command-module] module takes the command name followed by a list of space-delimited arguments.
It will not be processed through the shell, so variables like $HOME and operations like "<", ">", "|", ";" and "&" will not work (use the `shell` module if you need these features).

```
- name: Return somefile to registered var - somefile_content
  command: cat somefile
  args:
    chdir: somedir/
  register: somefile_content

- name: Run the command if the specified file does not exist.
  command: /usr/bin/make_database.sh arg1 arg2
  args:
    creates: /path/to/database

- name: Run the command if the specified file already exists.
  command: /usr/bin/remove_database.sh arg1 arg2
  args:
    removes: /path/to/database
```

## shell

The [`shell`][shell-module] module takes the command name followed by a list of space-delimited arguments.
It is almost exactly like the command module but runs the command through a shell (`/bin/sh`) on the remote node.
It also has the `creates` and `removes` parameters.

```
- name: Run liquibase
  shell: |
    docker-compose up -d db       &&
    docker-compose run liquibase  &&
    docker-compose down
  args:
    chdir: "{{ project_dir }}"

- name: Show replication slots
  shell: |
    psql -U postgres -c 'SELECT * FROM pg_replication_slots;'
  register: out

- name: Run a command using a templated variable (always use quote filter to avoid injection)
  shell: cat {{ myfile|quote }}
```

## raw

The [`raw`][raw-module] module allows execute command on the remote hosts that haven't got installed Python.
It is works when any other modules doesn't.

```
- name: Install python2 for Ansible on Debian host
   raw: bash -c "test -e /usr/bin/python || (apt -qqy update && apt install -qqy python-minimal python-simplejson)"
   register: output
   changed_when: output.stdout != ""
```

## service

The [`service`][service-module] module controls services on remote hosts.

```
- name: Start and enable service httpd, if not started
  service:
    name: httpd
    state: started
    enabled: yes

- name: Stop service httpd, if started
  service:
    name: httpd
    state: stopped

- name: Restart service httpd, in all cases
  service:
    name: httpd
    state: restarted
```

## yum

The [`yum`][yum-module] module installs, upgrades, downgrades, removes, and lists packages and groups with the `yum` package manager (such as for RedHat/CentOS).
This module only works on Python 2. If you require Python 3 support see the `dnf` module.

```
- name: Install the latest version of Apache
  yum:
    name: httpd
    state: latest

- name: Remove the Apache package
  yum:
    name: httpd
    state: absent

- name: Upgrade all packages
  yum:
    name: '*'
    state: latest

- name: Install KVM packages
  vars:
    ansible_python_interpreter: /usr/bin/python
  yum:
    name: "{{ item }}"
    state: present
  with_items:
    - qemu-kvm
    - libvirt
    - libvirt-python
    - libguestfs-tools
    - virt-install
    - bridge-utils
```

## apt

The [`apt`][apt-module] module manages `apt` packages (such as for Debian/Ubuntu).

```
- name: Update repositories cache and install Apache package
  apt:
    name: apache2
    update_cache: yes

- name: Remove Apache package
  apt:
    name: apache2
    state: absent

- name: Upgrade all packages to the latest version
  apt:
    name: "*"
    state: latest
    update_cache: yes

- name: Update all packages to the latest version
  apt:
    upgrade: dist

- name: Install KVM packages
  apt:
    name: "{{ item }}"
  with_items:
    - qemu-kvm
    - bridge-utils
    - libvirt-daemon
    - libvirt-daemon-system
    - libvirt-clients

```

## pkgng

The [`pkgng`][pkgng-module] module manages binary packages for FreeBSD after 9.0.

```
- name: Install package telegraf | FreeBSD
  pkgng:
    name: telegraf
    state: present

- name: Remove package beats
  pkgng:
    name: beats
    state: absent

- name: Upgrade package telegraf
  pkgng:
    name: telegraf
    state: latest
```

## apt_repository

The [`apt_repository`][apt-repository-module] module another useful module for working with package lists.
It adds or removes an APT repositories in Ubuntu and Debian.
It often is used with [`apt_key`][apt-key-module] module.

```
- name: Download apt key
  apt_key:
    url: https://artifacts.elastic.co/GPG-KEY-elasticsearch

- name: Add Elastic repository
  apt_repository:
    repo: deb https://artifacts.elastic.co/packages/6.x/apt stable main

- name: Remove specified repository from sources list
  apt_repository:
    repo: deb http://archive.canonical.com/ubuntu hardy partner
    state: absent
```

## yum_repository

The [`yum_repository`][yum-repository-module] module like `apt_repository` adds or removes repositories in RPM-based Linux distributions.

```
- name: Add EPEL repository
  yum_repository:
    name: epel
    description: EPEL YUM repo
    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/

- name: Remove repository
  yum_repository:
    name: epel
    state: absent

- name: Add OpenNebula repository
  when: ansible_os_family|lower == 'redhat'
  yum_repository:
    name: opennebula
    description: OpenNebula Repository - RHEL
    baseurl: https://downloads.opennebula.org/repo/5.6/CentOS/7/x86_64
    gpgcheck: yes
    gpgkey: https://downloads.opennebula.org/repo/repo.key
```

## user

The [`user`][user-module] module manages user accounts and user attributes.

```
- name: Create a backup user
  user:
    name: backup
    home: /nfsstore/backup/

- name: Remove the user 'johnd'
  user:
    name: johnd
    state: absent
    remove: yes

- name: Create a 2048-bit SSH key for user 'jsmith'
  user:
    name: jsmith
    generate_ssh_key: yes
    ssh_key_bits: 2048
```

## authorized-key-module

The [`authorized_key`][authorized-key-module] module adds or removes SSH authorized keys for particular user accounts.

```
- name: Set authorized key taken from file on control machine
  authorized_key:
    user: backup
    state: present
    key: "{{ lookup('file', 'backup/id_rsa.pub') }}"


- name: Set multiple authorized key taken from selected files
  authorized_key:
    user: root
    state: present
    key: "{{ lookup('file', item + '_id_rsa.pub') }}"
  loop:
    - jsmith
    - sjobs
    - rgreen
```

## lineinfile

The [`lineinfile`][lineinfile-module] module ensures a particular line is in a file, or replace an existing line using a back-referenced regular expression.
This is primarily useful when you want to change a single line in a file only.

```
- name: Disable SElinux (config) | RedHat
  lineinfile:
    path: /etc/selinux/config
    regexp: '^SELINUX=enforcing'
    line: 'SELINUX=disabled'

# Disable selinux immediately
- name: Disable SElinux (setenforce) | RedHat
  shell: setenforce 0 || true

- name: Add node IP to /etc/hosts
  lineinfile:
    path: /etc/hosts
    regexp: '{{ item.name }}$'
    line: '{{ item.ip }} {{ item.name }}'
  with_items:
    - ip: 10.0.0.1
      name: srv1
    - ip: 10.0.0.2
      name: srv2
```

## blockinfile

The [`blockinfile`][blockinfile-module] module will insert/update/remove a block of multi-line text surrounded by customizable marker lines.

```
- name: Add parameters for connection to host into ssh-config
  blockinfile:
    path: ~/.ssh/config
    marker_begin: "BEGIN {{ item.name }}"
    marker_end: "END {{ item.name }}"
    marker: "# {mark}"
    block: |
      Host {{ item.name }}
      HostName {{ item.name }}
      User oneadmin
    with_items:
      - ip: 10.0.0.1
        name: srv1
      - ip: 10.0.0.2
        name: srv2
```

## debug

The [`debug`][debug-module] module prints statements during execution.
It can be useful for debugging variables or expressions.
Useful for debugging together with the ‘when:’ directive.

```
- debug:
    msg: "System {{ inventory_hostname }} has gateway {{ ansible_default_ipv4.gateway }}"
  when: ansible_default_ipv4.gateway is defined

- shell: /usr/bin/uptime
  register: result

- debug:
    var: result
    verbosity: 2
```

## pause

The [`pause`][pause-module] module pauses playbook execution for a set amount of time, or until a prompt is acknowledged.
Use `ctrl+c` to advance a pause earlier than it is set to expire or if you need to abort a playbook run entirely.
To continue early press `ctrl+c` and then `c`. To abort a playbook press `ctrl+c` and then `a`.

```
- name: Pause for 5 minutes
  pause:
    minutes: 5

- name: Pause for 30 seconds
  pause:
    seconds: 5
```

## fail

The [`fail`][fail-module] module fails the progress with a custom message.
It can be useful with flag `ingnore_error` in previous task. It's often used with some condition.

```
- name: build app
  docker_container:
    image: maven
    volumes:
     - "{{ project_dir }}:/pd"
    working_dir: /pd
    command: mvn clean install
  register: app_build
  ignore_errors: yes

- debug:
    var=app_build.ansible_facts.docker_container.Output.split('\n')

- name: fail exit when app_build failed
  fail:
    msg: |
      app build failed!
  when: app_build.failed|default(false)|bool == true
```

In this article I show you how to use some modules in Ansible playbooks.
This list has included my preferred modules.
I hope, it will be a good start to meet full features of Ansible.
Also, these notes will help me to remember forgotten things.

## Additional information

* [Getting Started with Ansible][getting-started] - A simple guide to Ansible's community and resources.
* [Ansible Documentation][docs] - Covers all Ansible options in depth. There are few open source projects with documentation as clear and thorough.
* [Ansible for DevOps][book] - this book helps reach an expert level of Ansible using.
* [15 Things You Should Know About Ansible][15-things] - some notes to dive deep into Ansible.

[getting-started]: https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html
[docs]: https://docs.ansible.com/
[book]: https://leanpub.com/ansible-for-devops
[15-things]: http://codeheaven.io/15-things-you-should-know-about-ansible/

[getting-started]: {% post_url 2019-01-09-ansible-getting-started %}
[all-modules]: https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html
[reboot-module]: https://docs.ansible.com/ansible/latest/modules/reboot_module.html
[copy-module]: https://docs.ansible.com/ansible/latest/modules/copy_module.html
[fetch-module]: https://docs.ansible.com/ansible/latest/modules/fetch_module.html
[template-module]: https://docs.ansible.com/ansible/latest/modules/template_module.html
[file-module]: https://docs.ansible.com/ansible/latest/modules/file_module.html
[command-module]: https://docs.ansible.com/ansible/latest/modules/command_module.html
[shell-module]: https://docs.ansible.com/ansible/latest/modules/shell_module.html
[synchronize-module]: https://docs.ansible.com/ansible/latest/modules/synchronize_module.html
[service-module]: https://docs.ansible.com/ansible/latest/modules/service_module.html
[raw-module]: https://docs.ansible.com/ansible/latest/modules/raw_module.html
[yum-module]: https://docs.ansible.com/ansible/latest/modules/yum_module.html
[apt-module]: https://docs.ansible.com/ansible/latest/modules/apt_module.html
[pkgng-module]: https://docs.ansible.com/ansible/latest/modules/pkgng_module.html
[apt-repository-module]: https://docs.ansible.com/ansible/latest/modules/apt_repository_module.html
[apt-key-modules]: https://docs.ansible.com/ansible/latest/modules/apt_key_module.html
[yum-repository-module]: https://docs.ansible.com/ansible/latest/modules/yum_repository_module.html
[user-module]: https://docs.ansible.com/ansible/latest/modules/user_module.html
[authorized-key-module]: https://docs.ansible.com/ansible/latest/modules/authorized_key_module.html
[blockinfile-module]: https://docs.ansible.com/ansible/latest/modules/blockinfile_module.html
[lineinfile-module]: https://docs.ansible.com/ansible/latest/modules/lineinfile_module.html
[pause-module]: https://docs.ansible.com/ansible/latest/modules/pause_module.html
[debug-module]: https://docs.ansible.com/ansible/latest/modules/debug_module.html
[fail-module]: https://docs.ansible.com/ansible/latest/modules/fail_module.html
