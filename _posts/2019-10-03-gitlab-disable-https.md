---
published: true
layout: single
title: GitLab CE in Docker. Disable HTTPS
excerpt: >-
  GitLab CE enable https and Let's Encrypt certificates by default.
  What to do if it no needs? How to disable listening 443 TCP port.
categories: devops
tags: gitlab devops nginx
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-10-03
---

This post shows options that should setup in GitLab settings to fully disable https.
I needed because I use GitLab with external Web server (NGINX) and this server proxies traffic by local network.
GitLab opens only 80 and 22 ports.

## Environment

I use that I was deployed [before][prev-post]
1. GitLab CE 12.3.0 (Omnibus package for Docker) - gitlab.example.com

It doesn't matter where you host your GitLab instance, it can be self managed as usual service.
But in Cloud GitLab it should be another way to configure.

## Configuration

Full docs are available [here](https://docs.gitlab.com/omnibus/settings/ssl.html).

1. Find and open your `gitlab.rb` configuration file. And add this rows:

```ruby
external_url 'https://gitlab.example.com'
nginx['listen_https'] = false
nginx['listen_port'] = 80
nginx['redirect_http_to_https'] = false
letsencrypt['enable'] = false
```

2. Restart GitLab. Run the command on the host in console

```
# gitlab-ctl reconfigure
```

3. Write a right configuration for you web server to proxy `gitlab.example.com`
to the GitLab instance. I use NGINX and my configuration is:

```
server {
    listen          443 ssl;
    server_name     gitlab.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

    include /etc/nginx/ssl.conf;

    location / {
        proxy_pass           http://192.168.40.205;  # GitLab local network IP
        proxy_set_header     X-Real-IP       $remote_addr;
        proxy_set_header     Host            $http_host;
        proxy_set_header     Upgrade         $http_upgrade;
        proxy_set_header     Connection      "Upgrade";
        proxy_set_header     X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout        2;
        proxy_http_version           1.1;
        proxy_ssl_server_name        on;
        proxy_ignore_client_abort    on;
        fastcgi_ignore_client_abort  on;
        proxy_buffering              off;
        proxy_intercept_errors       on;
        proxy_buffers                16       16k;
        proxy_buffer_size            16k;
        client_max_body_size         0;
    }
}

server {
    listen          80;
    server_name     gitlab.example.com;

    return 301 https://$host$request_uri;
}
```

The disabling https needs only if you have external web proxy (like NGINX).

## Additional information

* [GitLab SSL Configuration](https://docs.gitlab.com/omnibus/settings/ssl.html) -
    Official doc to configure SSL.

[prev-post]: {% post_url 2019-10-02-deploy-gitlab-in-docker %}
