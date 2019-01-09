---
published: true
layout: single
title: Ansible. Architecture
excerpt: >-
  Ansible is an automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs. What is playbook, inventory and other terms will be described in this post.
categories: devops
tags: ansible devops
toc: true
header:
  teaser: /assets/images/ansible-architecture.png
  og_image: /assets/images/ansible-architecture.png
---

Ansible is an automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs.
This post is inspired an [official documentation][overview-architecture] about architecture of Ansible.
Many thoughts is perfect described in the docs, but I want to explain it in my words and collect all of my experience in one place. I hide details and concentrate on the base.

The main idea of Ansible is one or more the controller server, that needed for sending commands to the remote servers by SSH. It doesn't require agents on remotes.
All commands send throw secure network and the hosts run it. Only one thing is needed is Python interpreter. And it is installed on the every popular distributive of OS (Centos, Debian, etc.) by default.

![ansible-architecture]({{ site.url }}{{ site.baseurl }}/assets/images/ansible/ansible-architecture.jpg){: .align-center}

Ansible based on the next terms: inventory, playbook, task, role, handler, module, template, facts and variables.
This sequence could be continued but I think if you will know mean of the words you could use Ansible in major cases.

## 1. Inventory

Basically inventory is a file (by default, Ansible uses a simple INI format) that describes Hosts and Groups. It stores in file `hosts`. Also inventory can be a [dynamic][dynamic-inventory] to flexible and customizable server infrastructure.

I prefer writing inventory in YAML format. A YAML version would look like:
```yaml
all:
  hosts:
    mail.example.com:
      ansible_host: 192.0.2.50
      ansible_port: 5555
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        two.example.com:
        three.example.com:
```

In this inventory include two groups (`webservers` and `dbservers`) includes six hosts.
There are two default groups: `all` and `ungrouped`.
The parameters `ansible_host` and `ansible_port` explain how to connect to the mail server (by this IP and port).
Also [inventory][inventory-intro] can include variables for groups or individual hosts using in playbooks. And many other magic thing can be making with inventory.

Next command shows the hosts that includes into group `dbservers`:
```
ansible --list-host "dbservers"
```

For showing all hosts use it:
```
ansible --list-host "*"
```

For request what hosts will be using with selected playbook use next:
```
ansible-playbook <playbook-name.yml> --list-hosts
```

## 2. Playbook

[Playbooks][playbook-intro] are the language by which Ansible orchestrates, configures, administers, or deploys systems. At a basic level, playbooks can be used to manage configurations of and deployments to remote machines.

Playbook basically consist of roles and tasks that will execute on the selected hosts.

There is the simplest playbook that ping the hosts in group `webservers`:
```yaml
---
- hosts: webservers
  tasks:
    - name: test connection
      ping:
```

For example this playbook was saved in the file `test-connection-playbook.yml`.
Then we can execute it with next command:
```
ansible-playbook test-connection-playbook.yml
```

## 3. Task

Playbooks exist to run tasks. Tasks combine an action (**a module** and **its arguments**) with a name and optionally some other keywords. Handlers are also tasks, but they are a special kind of task that do not run unless they are notified by name when a task reports an underlying change on a remote system.

The example of a task has showed before in playbook section.

## 4. Role

Roles are ways of automatically loading certain tasks, variables and handlers based on a known file structure. Grouping content by roles also allows easy sharing of roles with other users.

Roles work only if you have prepared project files structure like this one:
```
test-connection-playbook.yml
roles/
   common/
     tasks/
        main.yml
     handlers/
     files/
     templates/
     ...
```

Roles expect files to be in certain directory names.
Firstly is loading the file `tasks/main.yml`. It can load other files to have platform-specific tasks.

For using role in the playbooks need to add `roles` section into a playbook file.
Also common practice for creating reusable playbooks is to define a variable in the playbook. See the example below.
```yaml
---
- hosts: webservers
  roles:
    - role: common
      vars:
         dir: '/opt/a'
         app_port: 5000
```

## 5. Handler

Handler is just like regular task in an playbook. It is only run if the task contains a notify directive and also indicates that it changed something. For example, if a config file is changed, then the task referencing the config file templating operation may notify a service restart handler.

```yaml
---
- hosts: webservers
  tasks:
  - name: write the apache config file
    template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf
    notify:
    - restart apache
  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```


## 6. Module

Modules are the units of work that Ansible ships out to remote machines.
Once modules are executed on remote machines, they are removed, so no long running daemons are used. Ansible refers to the collection of available modules as a library.

Ansible contains a giant toolbox of built-in [modules][all-modules]. And you can write you own module in any language, including Perl, Bash, or Ruby. Modules just have to return JSON.

Before running an action module check the current state of a host, service, file. And if it is in the required state the module wouldn't change anything and returns the `changed: false` flag.

For example, the copy module can be used to copy files – and shows that files are only transferred if they are not already there:
```
$ ansible example.com -m copy -a "src=some_config.conf dest=config.conf"
example.com | success >> {
    "changed": false,
    ...
...
```

## 7. Template

Ansible can easily transfer files to remote systems but often it is desirable to substitute variables in other files. [Templates][template-intro] use the Jinja2 template engine and can also include logical constructs like loops and if statements.
It used a `template` module in playbooks. It prefers to name template files with extension `j2`.

## 8. Facts

Before make changes Ansible connects to remote hosts and gather information about it: network interfaces, host states, operating system, and etc.
Thus facts are simply things that are discovered about remote nodes.
Facts are automatically discovered by Ansible when running plays by executing the internal setup module on the remote nodes.
Also it can be manually disabled in the playbook with `gather_facts: false` parameters.
And if you want only gather facts on selected hosts just run next command with `setup` [module][setup-module]:
```
ansible all -m setup
```

## 9. Variables

As opposed to Facts, variables are names of values (they can be simple scalar value's integers, booleans, strings) or complex ones (dictionaries/hashes, lists) that can be used in templates and playbooks. They are declared things, not things that are inferred from the remote system's current state or nature (which is what Facts are).

Ansible uses variables to help deal with differences between systems.
Vars need to make things repeatable.
YAML supports dictionaries which map keys to values. For instance:
```
foo:
  field1: one
  field2: two
```

And for getting a value of the field2 used next reference `foo.field2`.

Once you have defined variables, you can use them in your playbooks using the Jinja2 templating system. Here is a simple Jinja2 template:
```
My amp goes to {{ max_amp_value }}
```
The same syntax is using in playbooks. More examples you can find in the [docs][vars-intro].


So in the end I can only recommend Ansible to anyone who is dealing with configuration management and deployments. It is a certainly helpful tool. And it requires minimal actions in your servers to try it and use it.


## Additional information

* [Ansible Documentation][docs] - Covers all Ansible options in depth. There are few open source projects with documentation as clear and thorough.
* [Ansible Glossary][glossary] - If there is ever a term in this book you don't seem to fully understand, check the glossary.
* [Getting Started with Ansible][getting-started] - A simple guide to Ansible's community and resources.
* [Ansible for DevOps][book] - this book helps reach an expert level of Ansible using.
* [Ansible - A Beginner's Tutorial][bens-lessons] - youtube lessons about the basics of Ansible.
* [Ansible Architecture][arch-youtube] - an youtube video that explain the base of Ansible


[arch-youtube]: https://www.youtube.com/watch?v=fiZkd1aK2OQ
[bens-lessons]: https://www.youtube.com/playlist?list=PLFiccIuLB0OiWh7cbryhCaGPoqjQ62NpU
[getting-started]: https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html
[docs]: https://docs.ansible.com/
[glossary]: https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html
[book]: https://leanpub.com/ansible-for-devops
[overview-architecture]: https://docs.ansible.com/ansible/latest/dev_guide/overview_architecture.html
[dynamic-inventory]: https://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html
[inventory-intro]: https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html
[playbook-intro]: https://docs.ansible.com/ansible/latest/user_guide/playbooks.html
[role-intro]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html
[all-modules]: https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html
[template-intro]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html
[setup-module]: https://docs.ansible.com/ansible/latest/modules/setup_module.html#setup-module
[vars-intro]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html
