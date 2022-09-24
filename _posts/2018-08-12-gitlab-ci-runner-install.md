---
published: true
title: GitLab CI/CD. Install and configure Runner
excerpt: >-
  GitLab uses Runners for a running scripts that defined in your CI pipeline.
  A Runner is a daemon that hosted in your servers.  It waits commands from
  GitLab and activates when a job executed. It requires some manual actions to
  install and configure.
categories: devops
tags: gitlab devops centos
toc: true
header:
  teaser: /assets/images/gitlab-ci/gitlab-ci-runner-centos.png
  og_image: /assets/images/gitlab-ci/gitlab-ci-runner-centos.png
---

In GitLab Runners run the jobs that you define in [.gitlab-ci.yml][gitlab-ci.yml].
A Runner can be a virtual machine, a VPS, a bare-metal machine, a docker container
or even a cluster of containers. GitLab and the Runners communicate through an API,
so the only requirement is that the Runner's machine has Internet access.

The official Runner supported by GitLab is written in Go and its documentation
can be found at [https://docs.gitlab.com/runner/][runner].

In order to have a functional Runner you need to follow two steps:
1. Install it
2. Configure it

Next I'll show you how to install and configure the latest GitLab Runner with
shell executor in CentOS system. Additional steps you can find in
[official docs](https://docs.gitlab.com/runner/install/index.html).

## Install Runner

1. Add GitLab's official [repository][runner-repository] (for RHEL/CentOS/Fedora):
```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
```

2. Install the Runner:
```
yum install -y gitlab-runner
```


## Configure Runner

### Register Runner

[Registering][register-runner] a Runner is the process that binds the Runner
with a GitLab instance.

You can register a Runner in interactive mode with a command:
```
gitlab-runner register
```

And reply to the questions of the wizard.

But I would prefer to register it with inserting answers in parameters:
```shell
gitlab-runner register -n \
  --url "https://YOUR_GITLAB_URL/" \
  --registration-token "XXXXXXXXXXXXXXXXXXXXXXX" \
  --description "nginx" \
  --tag-list "nginx-shell" \
  --executor "shell" \
  --limit 1
```

Don't forget place your toker. It stored in Admin Area ➔ Overview ➔ Runners:

`https://YOUR_GITLAB_URL/admin/runners`

![runners-admin]({{ site.baseurl }}/assets/images/gitlab-ci/runners-admin.png){: .align-center}

And set the correct description and tags. I prefer to use a hostname as a description.
I named it `nginx` because I run it on Nginx server. And set up tags.
I suggest use `host + executor` as a tag.

### Start Runner
```
gitlab-runner start
```

Check a status of a service:
```
# gitlab-runner status
gitlab-runner: Service is running!
```

Show configured runners on the host:
```
# gitlab-runner list
Listing configured runners                          ConfigFile=/etc/gitlab-runner/config.toml
host1                                                 Executor=shell Token=XXXXXXXXXXXXXXXXXXXXXXX URL=https://YOUR_GITLAB_URL/
```

After that I usually change a type of a Runner from a shared into a specific.

### Converting Runner type

Set up the current Runner as a specific one to deny to use a Runner of all projects.
Just choose our new Runner in Admin Area and determine some project or projects with it.

Once the Runner has been set up, you should see it on the Runners page of your
project, following Settings ➔ CI/CD:

![change-runner-type]({{ site.baseurl }}/assets/images/gitlab-ci/change-runner-type.png){: .align-center}



After that you can configure your CI process in a .gitlab-ci.yml file with
using tags to select specific Runners. For example:
```yaml
example_job:
  tags:
    - nginx-shell
  script:
    - execute_something_script
```

Next [post][create-first-pipeline] will help you to write a basic CI/CD configuration.

[gitlab-ci.yml]:https://docs.gitlab.com/ce/ci/yaml/README.html
[runner]:https://docs.gitlab.com/runner
[runner-repository]:https://docs.gitlab.com/runner/install/linux-repository.html
[register-runner]:https://docs.gitlab.com/runner/register

[create-first-pipeline]:{% post_url 2018-08-13-gitlab-ci-create-first-pipeline %}
