---
published: false
layout: single
title: Custom OAuth2 provider with Authlib. Flask implementation and Nextcloud tested
excerpt: >-
  One day I've to realize the OAuth2 Server. A main application was written on Python.
  It will be used as a single point to login to many corporate services.
  The one of them is Nextcloud. Nextcloud is a file hosting service.
  I'll show you how to use Authlib Python library for building own OAuth2 Simple Server.
  It will based on Flask framework.
categories: devops
tags: docker python devops
toc: true
header:
  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-09-20
---

## About OAuth2

Some information about OAuth2 procedure, library implementation, etc.

[OAuth2][rfc-oauth2]

## Flask implementation

I'll use a completed example to test a custom OAuth2 on Nextcloud.
This project will run in docker environment with Nextcloud instance.
Just open console and execute next:

## Run project

```shell_module
docker-compose up
```

## Configure applications

Flask app will be accessed by [localhost:5000](http://localhost:5000/),
Nextcloud - [localhost](http://localhost/).

Open browser and enter to the Flask app. Login as `admin` user.
Create client - [create_client](http://localhost:5000/create_client):

|          Parameter         |                         Value                        |
|:--------------------------:|:----------------------------------------------------:|
| Client Name                | nextcloud                                            |
| Client URI                 | http://localhost                                     |
| Allowed Scope              | profile                                              |
| Redirect URIs              | http://localhost/apps/sociallogin/custom_oauth2/test |
| Allowed Grant Types        | authorization_code                                   |
| Allowed Response Types     | code                                                 |
| Token Endpoint Auth Method | client_secret_post                                   |


After that authorize in your [local Nextcloud](http://localhost/) as admin.
Then find and setup `sociallogin` [plugin](https://apps.nextcloud.com/apps/sociallogin) in [App section](http://localhost/settings/apps). And open configure plugin page.

I use next settings to setup our custom OAuth2 provider

|                  Parameter                  |                       Value                      |
|:-------------------------------------------:|:------------------------------------------------:|
| Internal name                               | test                                             |
| Title                                       | test                                             |
| API Base URL                                | http://localhost:5000/                           |
| Authorize url (can be relative to base URL) | http://localhost:5000/oauth/authorize            |
| Token url (can be relative to base URL)     | http://flask-app:5000/oauth/token                |
| Profile url (can be relative to base URL)   | http://flask-app:5000/api/me                     |
| Logout URL (optional)                       |                                                  |
| Client Id                                   | fYKQrkBZABNnxEWEO4HyS1pF                         |
| Client Secret                               | jI9lHDGk0pvUQSiQ3whWwhOJiVoOF7RgLHJZqllVMtoJEHpl |
| Scope (optional)                            | profile                                          |

Some links has include `localhost` and other has `flask-app` as domain name.
It's OK, because some data transfer will taken in background through services.
Services use them names that was configured in docker-compose file.

Client parameters was generated in flask app. Just copy them.

After save configuration log out from Nextcloud and click `Login with test`
in start page.



## Additional information
https://docs.authlib.org/en/latest/index.html

https://github.com/authlib/example-oauth2-server

https://github.com/zorn-v/nextcloud-social-login/blob/master/3rdparty/hybridauth/hybridauth/src/User/Profile.php


* [Getting Started with Ansible][getting-started] - A simple guide to Ansible's community and resources.
* [Ansible Documentation][docs] - Covers all Ansible options in depth. There are few open source projects with documentation as clear and thorough.
* [Ansible for DevOps][book] - this book helps reach an expert level of Ansible using.
* [15 Things You Should Know About Ansible][15-things] - some notes to dive deep into Ansible.

[getting-started]: https://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html
[docs]: https://docs.ansible.com/
[book]: https://leanpub.com/ansible-for-devops
[15-things]: http://codeheaven.io/15-things-you-should-know-about-ansible/

[getting-started]: {{ "" | relative_url }}{% post_url 2019-01-09-ansible-getting-started %}
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
[apt-key-module]: https://docs.ansible.com/ansible/latest/modules/apt_key_module.html
[yum-repository-module]: https://docs.ansible.com/ansible/latest/modules/yum_repository_module.html
[user-module]: https://docs.ansible.com/ansible/latest/modules/user_module.html
[authorized-key-module]: https://docs.ansible.com/ansible/latest/modules/authorized_key_module.html
[blockinfile-module]: https://docs.ansible.com/ansible/latest/modules/blockinfile_module.html
[lineinfile-module]: https://docs.ansible.com/ansible/latest/modules/lineinfile_module.html
[pause-module]: https://docs.ansible.com/ansible/latest/modules/pause_module.html
[debug-module]: https://docs.ansible.com/ansible/latest/modules/debug_module.html
[fail-module]: https://docs.ansible.com/ansible/latest/modules/fail_module.html
