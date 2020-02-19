---
published: true
layout: single
title: CentOS 8. Podman installation and basic usage
excerpt: >-
  Now that official support for the Docker container runtime has been dropped by RHEL 8/CentOS 8, what are container admins to do? The solution is called Podman which functions without requiring a container daemon, as all containers and pods are created as child processes.
categories: linux
tags: centos linux podman
toc: true
header:
  teaser: /assets/images/podman-logo.png
  og_image: /assets/images/podman-logo.png
---

[Podman][podman] is a free and open-source daemonless container platform that was built to develop, manage and deploy containers and pods on a Linux environment.
Pods are group of containers which are usually deployed on the same host system.
Podman is gradually replacing docker which is another containerization platform that developers use to deploy their applications together with dependencies and libraries.
The main difference between the two is that while docker is a daemon that can be started, enabled, stopped and restarted, podman is not.
Podman is considered more secure due to its audit logging capability in containers.
The auditing plays a very crucial role in monitoring the processes that are running in a container.


## Docker vs Podman

The major difference between Docker and Podman is that there is no daemon in Podman.
It uses container runtimes as well for example runc but the launched containers are direct descendants of the podman process.

This kind of architecture has its advantages such as the following:
* Applied cgroups or security constraints still control the container:
  * whatever cgroup constraints you apply on the podman command, the containers launched will receive those same constraints directly.
* Advanced features of systemd can be utilized using this model:
  * This can be done by placing podman into a systemd unit file and hence achieving more.


## Installing podman on CentOS 8

To install podman on CentOS 8, simply log in as the root user and run the command:

```
# dnf install podman -y
```

## Installing podman on RHEL 8

Run below command to install Podman on RHEL 8 System
```
# dnf module install container-tools -y
```

## View a version

After the successful installation process , check the version of podman using the command:
```
$ podman --version
podman version 1.4.2-stable2
```

## Podman info

After the installation, you can display information pertaining to the host, current storage stats, and build of podman.
```
$ podman info
host:
  BuildahVersion: 1.9.0
  Conmon:
    package: podman-1.4.2-5.module_el8.1.0+237+63e26edc.x86_64
    path: /usr/libexec/podman/conmon
    version: 'conmon version 2.0.1-dev, commit: unknown'
  Distribution:
    distribution: '"centos"'
    version: "8"
  MemFree: 6229364736
  MemTotal: 10473271296
  OCIRuntime:
    package: runc-1.0.0-60.rc8.module_el8.1.0+237+63e26edc.x86_64
    path: /usr/bin/runc
    version: 'runc version spec: 1.0.1-dev'
  SwapFree: 0
  SwapTotal: 0
  arch: amd64
  cpus: 6
  hostname: centos
  kernel: 4.18.0-147.3.1.el8_1.x86_64
  os: linux
  rootless: false
  uptime: 26m 26.07s
registries:
  blocked: null
  insecure: null
  search:
  - registry.redhat.io
  - registry.access.redhat.com
  - quay.io
  - docker.io
store:
  ConfigFile: /etc/containers/storage.conf
  ContainerStore:
    number: 5
  GraphDriverName: overlay
  GraphOptions: null
  GraphRoot: /var/lib/containers/storage
  GraphStatus:
    Backing Filesystem: xfs
    Native Overlay Diff: "true"
    Supports d_type: "true"
    Using metacopy: "false"
  ImageStore:
    number: 5
  RunRoot: /var/run/containers/storage
  VolumePath: /var/lib/containers/storage/volumes
```

## Help page

To check the help page, run the command:
```
$ podman --help
```

Also it works with subcommands:
```
$ podman build --help
Build an image using instructions from Dockerfiles

Description:
  Builds an OCI or Docker image using instructions from one or more Dockerfiles and a specified build context directory.

Usage:
  podman build [flags] CONTEXT
....
```


## Run container

Connect to the interactive session of a Container with [i] and [t] option like follows.
If [exit] from the Container session, the process of a Container finishes.
```
podman run -it centos /bin/bash
```

## Migrating from Docker to Podman

Migrating from Docker to Podman is very easy:
* You need to install Podman instead of Docker. You do not need to start or manage a daemon process like the Docker daemon.
* The commands that you use with Docker will be the same for Podman.
* Images of Docker is compatible with Podman.
* Podman stores its containers and images in a different place than Docker.

## Conclusion

I hope this article has been useful and will help you migrate to using Podman confidently and successfully.

[podman]: https://podman.io/
