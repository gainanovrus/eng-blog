---
published: true
layout: single
title: Installing HAProxy on pfSense with SSL access to web server
excerpt: >-
  When you use pfSense as firewall often you want to protect you local resources
  form external threats. Also pfSense used as router to transfer local and external
  web servers traffic. HAProxy with SSL provides secure and performance access to
  many web sites hosted on multiple hosts connected with pfSense LAN.
categories: linux
tags: pfsense linux ssl haproxy
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-10-09
---


## Introduction

HAProxy is a small but powerful reverse proxy, and allows for loadbalancing
between multiple (web)servers, but also acl (Access Control Lists) allow for
selecting a specific backend or action depending on flexible criteria. And
though many features are available through webGUI. For all possible options
please look at the official [HAProxy documentation][haproxy-docs].

I have an nginx service in an Linux VM as a web server with a site
`nginx.example.com`. In next time will be second VM with another web-service.
I want to utilize HAProxy on my edge router (pfSense-2.4) to proxy to their
appropriate backend VMs.

## Environment

In this article I’ll be showing you how to do this with next version of components:
1. pfSense 2.4.4
2. [haproxy package][haproxy-package] 0.59_19 (with included haproxy 1.7.11)
3. Linux VM with NGINX accessed by IP `192.168.100.3`
4. Domain `nginx.example.com` directed on WAN pfSense.

## Installation

So this post will describe how to open a web server from Internet by HTTPS and DNS name (nginx.example.com).
Now it is accessible by local ip (`192.168.100.3`). The pfSense is edge router.
And it already has free LetsEncrypt SSL certificates (how to get them - read
[previous post][pfsense-ssl]). HAproxy will help to make it easy.

### haproxy package

Under **System / Package Manager / Available Packages** find a package `haproxy`.
Click the install button and allow it to complete.

![haproxy_package_install]({{ site.baseurl }}/assets/images/pfsense/haproxy_package_install.png){: .align-center}

### configure haproxy

Once the package is installed navigate to **Services > HAProxy > Settings**
 and configure the settings how you wish, make sure `Enable` is checked, click `Save`.

My setup is like so:

![haproxy_configure]({{ site.baseurl }}/assets/images/pfsense/haproxy_configure.png){: .align-center}

### create backend

> Backends are what HAProxy calls the actual connecting servers, this is known as
"upstreams" in NGINX.

The next step is to create an HAProxy backend for a host.

I have one host with NGINX-service listened `80` port on local IP `192.168.102.3`.

![haproxy_backend]({{ site.baseurl }}/assets/images/pfsense/haproxy_backend.png){: .align-center}

Repeat this for each of your seperate backend "apps" or "servers",
a tip is you can copy one interface to duplicated it, then edit it as needed.

### create frontend

> Frontends are what HAProxy uses to map something to a backend, in this case were
mapping the hostname to a string and sending that matching traffic to the
appropriate backend.

The first step is to create a frontend that will routing all user requests to
right backend based on hostname information. ACL provides many functions to
optimize administrate all frontends in one place.

Use secure (https) connection with LetsEncrypt SSL certificates.

My frontend configuration looks like this:

![haproxy_frontend]({{ site.baseurl }}/assets/images/pfsense/haproxy_frontend.png){: .align-center}

### create firewall rule

Now create two firewall rules (**Firewall / Rules /WAN**).
It is open TCP-ports `80` and `443` through WAN interface  for opening our HAProxy to the external world.

![haproxy_firewall_edit]({{ site.baseurl }}/assets/images/pfsense/haproxy_firewall_edit.png){: .align-center}

![haproxy_firewall_full]({{ site.baseurl }}/assets/images/pfsense/haproxy_firewall_full.png){: .align-center}

### test everything

You should now be able to hit `http://nginx.example.com` and have it redirect to
`https://nginx.example.com` and also correctly go to the right backend server.

The site should have a "green" status because is used a secured connection.

![haproxy_test]({{ site.baseurl }}/assets/images/pfsense/haproxy_test.png){: .align-center}

## Some notes

After enable fronted with HTTPS I've got next warning message:
```
Starting haproxy:
[WARNING] 279/113603 (85707) : Setting tune.ssl.default-dh-param to 1024 by default, if your workload permits it you should set it to at least 2048. Please set a value >= 1024 to make this warning disappear.
```

This is not error in the first place.
To fix it change DH value throw HAproxy settings webGUI (**Services / HAProxy / Settings**).
`Tuning -> Max SSL Diffie-Hellman size = 2048`

![dh_tunning]({{ site.baseurl }}/assets/images/pfsense/dh_tunning.png){: .align-center}

The solution was founded in the [forum](https://forum.netgate.com/topic/111505/how-do-i-fix-this-error-in-haproxy)

## Additional information

* [A walkthrough on how to proxy https traffic to multiple sites](https://blog.devita.co/pfsense-to-proxy-traffic-for-websites-using-pfsense/#step1installthehaproxypackage) -
    A blog post with screenshots with similar idea.
* [pfsense-haproxy-package-doc](https://github.com/PiBa-NL/pfsense-haproxy-package-doc/wiki#https-for-multiple-domains-using-sni-from-1-frontend) -
    HAProxy pfSense package, howto.
* [HAProxy package - pfSense](https://docs.netgate.com/pfsense/en/latest/packages/haproxy-package.html) -
    Official documentation of HAProxy on pfSense site.
* [Another instruction for using HAProxy with pfSense](https://github.com/PiBa-NL/pfsense-haproxy-package-doc/wiki/Single-frontend-serving-multiple-different-domains-using-http) -
    Single frontend serving multiple different domains using http.

[pfsense-ssl]: {% post_url 2019-10-07-installing-lets-encrypt-pfsense %}
[haproxy-docs]: http://www.haproxy.org/#docs
[haproxy-package]: https://docs.netgate.com/pfsense/en/latest/packages/haproxy-package.html
