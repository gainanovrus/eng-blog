---
published: true
layout: single
title: Dockerfile. ARG and ENV statement differences
excerpt: >-
  The both words use for define variables in to Dockerfile.
  It may be difficult to understand the difference, but it will be explained here.
categories: devops
tags: docker dockerfile devops
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-11-22
---

The ARG and ENV statements use for define variables in to Dockerfile.
It may be difficult to understand the difference. The difference is when variables will be utilized.

If you need **build-time** customization, **ARG** is best choice.
From docker [reference][arg]:
> The `ARG` instruction defines a variable that users can pass at build-time to the builder with the docker build command using the --build-arg <varname>=<value> flag.

If you need **run-time** customization (to run the same image with different settings), **ENV** is well-suited.
What you should know from [docs][env]:
> The `ENV` instruction sets the environment variable <key> to the value <value>.
The environment variables set using ENV will persist when a container is run from the resulting image.

To define them as the command line instructions use next (for `ARG var` or ):

ARG (for `ARG var`): `docker build --build-arg var=xxx`

ENV (for `ENV var`): `docker run --env var=yyy`

The best way define them is write a [docker-compose][dcfile] file with defined next section:
```
version: "3"
services:
  ubuntu:
    build:
      context: .
      args:
        var: xxx
    environment:
      var: yyy
```

To run container with build image use next command
```
docker-compose up --build
```

## conclusion

I hope this explanation has helped you to learn a bit more about using dockerfile statements,
and will save you time to understand the basis of Docker.

[dcfile]: https://docs.docker.com/compose/compose-file/
[arg]: https://docs.docker.com/engine/reference/builder/#arg
[env]: https://docs.docker.com/engine/reference/builder/#env
