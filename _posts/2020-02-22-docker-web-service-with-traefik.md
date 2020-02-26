---
published: true
layout: single
title: Docker. Create secure web service using Let's Encrypt with Traefik
excerpt: >-
  Deploying web services to public network usually requires to set up secure connections
  using SSL certificates. With Docker you can easily make it using another container as reverse proxy.
  This service named Traefik. It has big capabilities about I will explain in the post.
categories: devops
tags: docker devops ssl letsencrypt traefik
toc: true
header:
  teaser: /assets/images/traefik/traefik_docker_ssl_logo.png
  og_image: /assets/images/traefik/traefik_docker_ssl_logo.png
---

In my [previous][previous] post I speak about [Traefik][traefik] concepts and designs. I explain the main words used in Traefik as Endpoint, Router, Rule and etc. So in here I will concentrate only on practice.

[![traefik-docker-scheme]({{ site.url }}{{ site.baseurl }}/assets/images/traefik/treafik_docker.png)]({{ site.url }}{{ site.baseurl }}/assets/images/traefik/treafik_docker.png){: .align-center}

## What We’ll Do
* We’ll use a pre-made container — [`containous/whoami`][whoami] — capable of telling you where it is hosted and what it receives when you call it.
* We’ll deploy that container through traefik proxy using docker-compose.
* We'll define a dashboard that shows us all deployed services.
* We'll setup letsencrypt configures that will automatically get and update free SSL certificates.
* We'll configure proxy to redirect all insecure requests to https scheme.

## Prerequisite
You have installed a Docker and docker-compose. We will use Traefik 2.1

## Repository
All presented compose files has stored in [this][traefik-my-repo] Github Repository.

## 1. Use traefik as proxy

This basic configuration will be a start point to run traefik with docker.

```yaml
version: '3'

services:
  traefik:
    image: traefik:v2.1
    container_name: traefik
    command:
      # Enable dashboard
      - --api.dashboard=true
      # Allow use API and dashboard though insecure
      - --api.insecure=true
      # Use docker as provider
      - --providers.docker=true
      # Enable log income requsts
      - --accesslog=true
      # Set log level (default ERROR)
      - --log.level=ERROR
      # Define entrypoint that listens 80 port
      - --entryPoints.web.address=:80
    ports:
      # The HTTP port
      - "80:80"
      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"
    volumes:
      # Mount docker socker from host machine
      - /var/run/docker.sock:/var/run/docker.sock:ro

  whoami:
    # A container that exposes an API to show its IP address
    image: containous/whoami
    container_name: whoami
    labels:
      # Create a route `whoami` and bound with defined entrypoint
      - traefik.http.routers.whoami.entrypoints=web
      # Create rule
      - traefik.http.routers.whoami.rule=Host(`whoami.example.com`)
```

We declare two services. First service is our traefik. Second one is a service response information about itself on web requests. Also we've enabled API along with the dashboard. And we can see it on [localhost:8080/dashboard/](http://localhost:8080/dashboard).

After we append a hostname `whoami.example.com` to local DNS (i.e. to `/etc/hosts`) we can access to a created `whoami` container from browser by link [http://whoami.example.com](http://whoami.example.com).

Without any changing DNS service we also could send request to `whoami` with curl:
```
# curl -H Host:whoami.example.com http://localhost

Hostname: 3b64e9ee3e38
IP: 127.0.0.1
IP: 172.28.0.2
RemoteAddr: 172.28.0.3:55870
GET / HTTP/1.1
Host: whoami.example.com
User-Agent: curl/7.54.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.28.0.1
X-Forwarded-Host: whoami.example.com
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: a9dfe96d5877
X-Real-Ip: 172.28.0.1
```

## 2. HTTPS with Let's Encrypt

Today almost all services are accessed by secure https connection. Installing an SSL certificate is the most common work.
It also could be done with Traefik. Let's create a compose file with next content:

```yaml
version: '3'

services:
  traefik:
    image: traefik:v2.1
    container_name: traefik
    command:
      - --api.dashboard=true
      - --api.insecure=true
      - --providers.docker=true
      - --accesslog=true
      - --entryPoints.web.address=:80
      - --entryPoints.websecure.address=:443
      # Enable ACME (Let's Encrypt): automatic SSL.
      - --certificatesResolvers.letsencrypt.acme.email=user@example.com
      # File or key used for certificates storage.
      - --certificatesResolvers.letsencrypt.acme.storage=/acme/acme.json
      # Uncomment the line to use Let's Encrypt's staging server,
      - --certificatesResolvers.letsencrypt.acme.caServer=https://acme-staging-v02.api.letsencrypt.org/directory
      # Use a TLS-ALPN-01 ACME challenge.
      # - --certificatesResolvers.letsencrypt.acme.tlsChallenge=true
      # Use a HTTP-01 ACME challenge.
      - --certificatesResolvers.letsencrypt.acme.httpChallenge=true
      # EntryPoint to use for the HTTP-01 challenges.
      - --certificatesResolvers.letsencrypt.acme.httpChallenge.entryPoint=web
    ports:
      # The HTTP port
      - "80:80"
      # The HTTPS port
      - "443:443"
      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      # ACME certificates can be stored in a JSON file that needs to have a 600 file mode
      - ./acme:/acme

  whoami:
    # A container that exposes an API to show its IP address
    image: containous/whoami
    container_name: whoami
    labels:
      # Handle secure and insecure traffic from websecure,web entry points
      - traefik.http.routers.whoami.entrypoints=websecure,web
      # Accept requests that matched specific Host
      - traefik.http.routers.whoami.rule=Host(`whoami.example.com`)
      # Enable TLS/SSL
      - traefik.http.routers.whoami.tls=true
      # Bind with created certresolver `letsencrypt`
      - traefik.http.routers.whoami.tls.certresolver=letsencrypt
```

A traefik service will call [Let's encrypt][letsencrypt] HTTP challenge to create a free SSL certificate.
The server have to be explored from Internet by 80 port and specified DNS `whoami.example.com`.
In this example to test purposes was setup a staging letsencrypt server. Comment the line with `acme.caServer` to use production server by default. Also don't forget to change a mail address `user@example.com` to your.

## 3. Add BasicAuth to Dashboard

Once Traefik has found a match for the request, it can process it before forwarding it to the service.
In the following example, we’ll add a [BasicAuth][basicauth] mechanism for our route.
This is done with a few additional labels on `traefik` service. After that you could open [Dashboard](https://docs.traefik.io/operations/dashboard/) by name `dashboard.example.com`.

```yaml
version: '3'

services:
  traefik:
    image: traefik:v2.1
    container_name: traefik
    command:
      - --api=true
      - --api.dashboard=true
      - --providers.docker=true
      - --providers.docker.exposedbydefault=true
      - --accesslog=true
      - --entryPoints.web.address=:80
      - --entryPoints.websecure.address=:443
      - --certificatesResolvers.letsencrypt.acme.email=user@example.com
      - --certificatesResolvers.letsencrypt.acme.storage=/acme/acme.json
      # Uncomment the line to use Let's Encrypt's staging server,
      - --certificatesResolvers.letsencrypt.acme.caServer=https://acme-staging-v02.api.letsencrypt.org/directory
      - --certificatesResolvers.letsencrypt.acme.httpChallenge=true
      - --certificatesResolvers.letsencrypt.acme.httpChallenge.entryPoint=web
    labels:
      - traefik.http.routers.api.entrypoints=websecure,web
      - traefik.http.routers.api.rule=Host(`dashboard.example.com`)
      - traefik.http.routers.api.tls=true
      - traefik.http.routers.api.tls.certresolver=letsencrypt
      # Connect the router `api` to internal service 'api'
      - traefik.http.routers.api.service=api@internal
      # Declaring a middleware with name `auth`
      # Declaring the user list - echo $(htpasswd -nb admin password) | sed -e s/\\$/\\$\\$/g
      - "traefik.http.middlewares.auth.basicauth.users=admin:$$apr1$$mW/l73Bf$$WsprkCzl5.QbLdY9c4kdB0"
      # Referencing an `auth` middleware
      - traefik.http.routers.api.middlewares=auth
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./acme:/acme

  whoami:
    # A container that exposes an API to show its IP address
    image: containous/whoami
    container_name: whoami
    labels:
      - traefik.http.routers.whoami.entrypoints=websecure,web
      - traefik.http.routers.whoami.rule=Host(`whoami.example.com`)
      - traefik.http.routers.whoami.tls=true
      - traefik.http.routers.whoami.tls.certresolver=letsencrypt
```

Let's explain what we made upper. First, we remove the insecure api (specifying `--api` instead of `--api.insecure`).
We declare a tls connection to `api` router that we made in previous part.
After that we bound the router with internal `api` service. Declaring a [Middleware][middleware] with name `auth` (`traefik.http.middlewares.auth.basicauth.users`). The value of it was include a string in format `username:xxxx`.
It could be generated with shell command - `echo $(htpasswd -nb username password) | sed -e s/\\$/\\$\\$/g`.
And at the end, join the middleware to the router `api` (`traefik.http.routers.api.middlewares=auth`).

[![traefik-dashboard]({{ site.url }}{{ site.baseurl }}/assets/images/traefik/dashboard_screenshot.png)]({{ site.url }}{{ site.baseurl }}/assets/images/traefik/dashboard_screenshot.png){: .align-center}

## 4. HTTPS Redirection

Now that we have HTTPS routes, let’s redirect every non-https requests to their https equivalent.
For that, we’ll reuse the previous trick and add just 3 labels to declare a redirect middleware. [RedirectScheme](https://docs.traefik.io/middlewares/redirectscheme/) will help us. It redirects request from a scheme to another. We will catch requests only for specific domain - `whoami.example.com`. See the example below:

```yaml
version: '3'

services:
  traefik:
    image: traefik:v2.1
    container_name: traefik
    command:
      - --api=true
      - --api.dashboard=true
      - --providers.docker=true
      - --providers.docker.exposedbydefault=true
      - --accesslog=true
      - --entryPoints.web.address=:80
      - --entryPoints.websecure.address=:443
      - --certificatesResolvers.myresolver.acme.email=r.gainanov@skoltech.ru
      - --certificatesResolvers.myresolver.acme.storage=/acme/acme.json
      # Uncomment the line to use Let's Encrypt's staging server,
      - --certificatesResolvers.myresolver.acme.caServer=https://acme-staging-v02.api.letsencrypt.org/directory
      - --certificatesResolvers.myresolver.acme.httpChallenge=true
      - --certificatesResolvers.myresolver.acme.httpChallenge.entryPoint=web
    labels:
      - traefik.http.routers.api.entrypoints=websecure,web
      - traefik.http.routers.api.rule=Host(`dashboard.example.com`)
      - traefik.http.routers.api.tls=true
      - traefik.http.routers.api.tls.certresolver=letsencrypt
      - traefik.http.routers.api.service=api@internal
      - traefik.http.routers.api.middlewares=auth
      - "traefik.http.middlewares.auth.basicauth.users=admin:$$apr1$$mW/l73Bf$$WsprkCzl5.QbLdY9c4kdB0"
      # Declaring a middleware with name `https_redirect` uses Redirecting the Client to a `https` Scheme
      - traefik.http.middlewares.https_redirect.redirectscheme.scheme=https
      # Set the permanent option to true to apply a permanent redirection.
      - traefik.http.middlewares.https_redirect.redirectscheme.permanent=true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./acme:/acme

  whoami:
    # A container that exposes an API to show its IP address
    image: containous/whoami
    container_name: whoami
    labels:
      traefik.http.routers.app.rule: Host(`whoami.example.com`)
      traefik.http.routers.app.entrypoints: web
      # Set the middleware `https_redirect` to `app` router to apply a redirection to https.
      traefik.http.routers.app.middlewares: https_redirect

      traefik.http.routers.appsecured.rule: Host(`whoami.example.com`)
      traefik.http.routers.appsecured.entrypoints: websecure
      traefik.http.routers.appsecured.tls: true
      traefik.http.routers.appsecured.tls.certresolver: myresolver
```

We use the previous example and add `middlewares.https_redirect` as traefik service label. After we bind this middleware to a router defined in whoami - `traefik.http.routers.app.middlewares: https_redirect`.

Follow to next advice if want a global redirect rule for requests to all insecured hosts. Move 2 rows from `whoami` service into traefik label's section `.app.rule`, `.app.entrypoints` and `.app.middlewares`. And change the value of the [Rule][rule] ```.rule: Host(`whoami.example.com`)``` to ```HostRegexp(`{host:.+}`)```

## Conclusion

Hopefully, I’ve gone through important questions you’ll have when dealing with Traefik 2.0 in a Docker setup, and I hope this examples get you a start point to explore more complex configurations.

## Additional information

* [Official Documentation](https://docs.traefik.io/) - the start point to know more about Traefik
* [Basic Example - Traefik](https://docs.traefik.io/user-guides/docker-compose/basic-example/) - docker-compose basic example by Traefik
* [Docker Configuration Reference](https://docs.traefik.io/reference/dynamic-configuration/docker/) - configure Traefik with Docker Labels
* [Traefik 2.0 & Docker 101](https://containo.us/blog/traefik-2-0-docker-101-fc2893944b9d/) - step by step with Traefik in an author blog


[previous]: {% post_url 2020-02-21-docker-traefik-concept-terms-explanation %}
[traefik]: https://docs.traefik.io/
[traefik-my-repo]: https://github.com/GRomR1/docker-traefik
[whoami]: https://github.com/containous/whoami
[letsencrypt]: https://letsencrypt.org/
[basicauth]: https://en.wikipedia.org/wiki/Basic_access_authentication
[rule]: https://docs.traefik.io/routing/routers/#rule
[service]: https://docs.traefik.io/routing/services/
[middleware]: https://docs.traefik.io/middlewares/overview/
