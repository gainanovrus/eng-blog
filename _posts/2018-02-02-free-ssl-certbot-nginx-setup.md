---
title:  "Add a free SSL certificate to NGINX server"
excerpt: "In this article I will show you how to use the certbot Let's Encrypt client to obtain a free SSL certificate and use it with Nginx on CentOS 7. I will also show you how to automatically renew your SSL certificate."
toc: true
categories: linux
tags: ssl nginx centos letsencrypt
last_modified_at: 2018-03-11T16:00:00+05:00
header:
  teaser: /assets/images/free-ssl-certbot-nginx-setup-teaser.png
  og_image: /assets/images/free-ssl-certbot-nginx-setup-teaser.png
---
[Let's Encrypt][lets-encrypt] provides an easy way to obtain and install free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers. It simplifies the process by providing a software client, Certbot, that attempts to automate most (if not all) of the required steps.

In this article I will show you how to use the certbot Let's Encrypt client to obtain a free SSL certificate and use it with Nginx on CentOS 7. I will also show you how to automatically renew your SSL certificate.

## Prerequisites
Before following this tutorial, you'll need a few things.
* CentOS 7 server.
* DNS A Record that points your domain to the public IP address of your server.

Once you have all of the prerequisites out of the way, let's move on to installing the Let's Encrypt client software.

## Instructions

### Install and configure nginx
```
 yum -y install nginx
 ```

 Add server name to nginx config file. Next a minimal configures that you need to get certificates  stored in file `/etc/nginx/nginx.conf`:
```
http {
    server {
        listen       80 default_server;
        server_name  example.com;

        location ~ ^/(.well-known/acme-challenge/.*)$ {
	        root     /usr/share/nginx/html;
	    }
	    location / {
	       ...
	    }
    }
}
```
### Install and configure Certbot

The official Let's Encrypt client is called Certbot, and it is included in the EPEL repository. Install Certbot with yum:
```
yum -y install certbot certbot-nginx
```

### Obtaining a Certificate

Certbot provides a variety of ways to obtain SSL certificates, through various plugins.
This runs certbot with the `--nginx` plugin, using `-d` to specify the names we'd like the certificate to be valid for. If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let's Encrypt server, then run a challenge to verify that you control the domain you're requesting a certificate for.

I suggest you a simple command that will obtain a certifiacte without any questions and outputs:
```
certbot certonly --webroot           \
-d example.com                       \
--webroot-path /usr/share/nginx/html \
--email your_email@example.com       \
--post-hook "systemctl reload nginx" \
--agree-tos                          \
--quiet
```

But you can yet view logs placed on path `/var/log/letsencrypt/letsencrypt.log`

To show information about obtained certificates use command `certbot certificates`:
```
# certbot certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log
-------------------------------------------------------------------------------
Found the following certs:
  Certificate Name: example.com
    Domains: example.com
    Expiry Date: 2018-05-03 04:20:43+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/example.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/example.com/privkey.pem
-------------------------------------------------------------------------------
```

The certificate is storing on the disk by path `/etc/letsencrypt/live/example.com`:
```
# ll /etc/letsencrypt/live/example.com
total 4
lrwxrwxrwx. 1 root root  37 Feb  2 08:45 cert.pem -> ../../archive/example.com/cert1.pem
lrwxrwxrwx. 1 root root  38 Feb  2 08:45 chain.pem -> ../../archive/example.com/chain1.pem
lrwxrwxrwx. 1 root root  42 Feb  2 08:45 fullchain.pem -> ../../archive/example.com/fullchain1.pem
lrwxrwxrwx. 1 root root  40 Feb  2 08:45 privkey.pem -> ../../archive/example.com/privkey1.pem
-rw-r--r--. 1 root root 543 Feb  2 08:45 README
```

In file `README` consists next description:
```
This directory contains your keys and certificates.

`privkey.pem`  : the private key for your certificate.
`fullchain.pem`: the certificate file used in most server software.
`chain.pem`    : used for OCSP stapling in Nginx >=1.3.7.
`cert.pem`     : will break many server configurations, and should not be used
                 without reading further documentation (see link below).
```

Additional information are available on the [certbot site][certs].

### Revoke a Certificate

If you didn't want use generated certificates any more just revoke it:
```
certbot revoke                                              \
--cert-path /etc/letsencrypt/live/example.com/fullchain.pem \
--delete-after-revoke                                       \
--quiet
```
I use a `--quiet` parameter if you want see a result of `certbot revoke` just run it without this option:
```
# certbot revoke --cert-path /etc/letsencrypt/live/example.com/fullchain.pem
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
-------------------------------------------------------------------------------
Would you like to delete the cert(s) you just revoked?
-------------------------------------------------------------------------------
(Y)es (recommended)/(N)o: y
-------------------------------------------------------------------------------
Deleted all files relating to certificate ems.insyte.ru.
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
Congratulations! You have successfully revoked the certificate that was located
at /etc/letsencrypt/live/example.com/fullchain.pem
-------------------------------------------------------------------------------
```

### Renew a Certificate

Let's Encrypt's certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process. We'll need to set up a regularly run command to check for expiring certificates and renew them automatically.

To run the renewal check daily, we will use cron, a standard system service for running periodic jobs.

We tell cron what to do by opening and editing a file called a crontab `crontab -e`:
```
15 3 * * * /usr/bin/certbot renew --post-hook "systemctl reload nginx" --quiet
```

The line runs the following command at 3:15 am, every day.

The renew command for Certbot will check all certificates installed on the system and update any that are set to expire in less than thirty days. After a renew process has completed a nginx server will been reload.

All installed certificates will be automatically renewed and reloaded when they have thirty days or less before they expire.

Read [docs][cert-command-line] to get information about command line options.

## Additional information
* [How To Secure Nginx with Let's Encrypt on CentOS 7 - DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7)
* [User Guide — Certbot documentation](https://certbot.eff.org/docs/using.html)
* [Nginx on CentOS/RHEL 7 — Certbot](https://certbot.eff.org/#centosrhel7-nginx)



[lets-encrypt]: https://letsencrypt.org/
[certs]: https://certbot.eff.org/docs/using.html#where-are-my-certificates
[cert-command-line]: https://certbot.eff.org/docs/using.html#certbot-command-line-options
