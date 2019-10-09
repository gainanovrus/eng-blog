---
published: false
layout: single
title: GitLab CE in Docker. Deploying
excerpt: >-
  A good team of software development doesn't exists without a code repository
  manager system. GitLab is the most popular and reach functionality system.
  This post will explain how to fast deploy GitLab as container in Docker.
categories: devops
tags: docker gitlab devops
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-10-02
---

## About GitLab CE

GitLab CE is an open source, cloud-based Git repository and version control
system used by thousands of organizations worldwide. Written in Ruby, GitLab CE
includes a host of features that enable software development teams to consolidate
source code, track and manage releases, increase code quality, deploy code
changes, and track the evolution of software over time.

Also included in the GitLab CE is a fully functional Continuous Integration
and Delivery (CI/CD) system that can build, test, and deploy software updates
as your team produces new code. Supporting the CI/CD functionality of GitLab
is a private registry for Docker containers, enabling teams to streamline
updates for production deployments that are running on a microservices
architecture.

You can install and self-manage GitLab under and MIT license or use GitLab's
commercial software built on top of the open source edition with additional
features.

Next will showed how to deploy GitLab CE quickly. The Docker will help us with it.
This can be allowed because GitLab maintains a set of official Docker images
based on the Omnibus GitLab package.

[Omnibus GitLab](omnibus) is a way to package different services and tools required to
run GitLab, so that most users can install it without laborious configuration.

## The requirements

The GitLab has a minimum [system requirements](system-req):
1 Cores, 4 GB RAM, 10 GB Storage. For a big workload read the docs.

The docker installation require installed [Docker](docker-install) software.

## The installation

Full docs are available [here](https://docs.gitlab.com/omnibus/docker/).
I will show only main parts of it to run it quickly.

1. There is `docker-compose.yml` file has included the basic configuration
to start GitLab in Docker:

```yaml
web:
  image: 'gitlab/gitlab-ce:latest'
  name: gitlab
  restart: always
  hostname: 'gitlab.example.com'
  environment:
    GITLAB_OMNIBUS_CONFIG: |
      external_url 'https://gitlab.example.com'
  ports:
    - '80:80'
    - '443:443'
    - '22:22'
  volumes:
    - '/srv/gitlab/config:/etc/gitlab'
    - '/srv/gitlab/logs:/var/log/gitlab'
    - '/srv/gitlab/data:/var/opt/gitlab'
```
Don't forget to change the `gitlab.example.com` to desired hostname.
GitLab has allow to set variables with `GITLAB_OMNIBUS_CONFIG` for configuration
GitLab.

> Note: The settings contained in GITLAB_OMNIBUS_CONFIG will not be written to the gitlab.rb configuration file, they’re evaluated on load.

It's reason then I preferable to write any customization in `gitlab.rb`.
I show my config in [next](next-topic) topic.

2. To start the installation run the command:
```
docker-compose up -d
```

3. To view logs after starting use next:
```
docker logs gitlab -f
```

4. After success starting open browser and setup new admin password.

## Some notes

I can't start GitLab with mounted volumes in NFS Storage. The error will be:
```
Multiple failures occurred:
* RuntimeError occurred in chef run: ruby_block[verify_chown_persisted_on_redis] (/opt/gitlab/embedded/cookbooks/cache/cookbooks/runit/libraries/provider_runit_service.rb line 148) had an error: RuntimeError: Unable to persist filesystem ownership changes of /var/log/gitlab/redis/config. See https://docs.gitlab.com/ee/administration/high_availability/nfs.html#recommended-options for guidance.
* RuntimeError occurred in delayed notification: ruby_block[restart_log_service] (/opt/gitlab/embedded/cookbooks/cache/cookbooks/runit/libraries/provider_runit_service.rb line 69) had an error: RuntimeError: ruby_block[verify_chown_persisted_on_redis] (/opt/gitlab/embedded/cookbooks/cache/cookbooks/runit/libraries/provider_runit_service.rb line 148) had an error: RuntimeError: Unable to persist filesystem ownership changes of /var/log/gitlab/redis/config. See https://docs.gitlab.com/ee/administration/high_availability/nfs.html#recommended-options for guidance.
```
I found the simple solution and delete a row `'/srv/gitlab/logs:/var/log/gitlab'` from yuml-file.

Without it gitlab logs will not save in host machine but you still view
logs with `docker logs` command.

Also you can use the next advice from docs - [GitLab - Disable storage directories management](fix-nfs).

## Additional information

* [Dockerfile of official GitLab image][https://gitlab.com/gitlab-org/omnibus-gitlab/tree/master/docker] -
    To view what actions was doing after you start an container.
* [Omnibus GitLab Docs - GitLab Docker images][https://docs.gitlab.com/omnibus/docker/] -
    The full information about installation GitLab in Docker.
* [Download and install GitLab][https://about.gitlab.com/install/] -
    The start page of installation process for everywhere.
* [GitLab Feature Comparison][https://about.gitlab.com/pricing/self-managed/feature-comparison/] -
    Find the difference between version of self-hosted GitLab.

[system-req]: https://docs.gitlab.com/ee/install/requirements.html
[docker-install]: https://docs.docker.com/install/
[omnibus]: https://docs.gitlab.com/omnibus/README.html
[next-topic]: {% post_url 2019-01-09-ansible-getting-started %}
[fix-nfs]: https://docs.gitlab.com/omnibus/settings/configuration.html#disable-storage-directories-management
