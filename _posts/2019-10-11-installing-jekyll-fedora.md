---
published: true
layout: single
title: Installing Jekyll on Fedora 30
excerpt: >-
  Jekyll is simple static site generator that focuses on blogs, but can be used
  for all kinds of sites. This blog uses Jekyll as base. And this post show
  commands that I use to fresh install it on Fedora Linux.
categories: linux
tags: jekyll linux fedora
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-10-10
---


## Introduction

[Jekyll][jekyll] is simple static site generator that focuses on blogs, but can be used
for all kinds of sites. It takes html templates and posts written in Markdown to
generate static website which is ready to deploy on your favorite web server.
Alternatively you can host it on GitHub and publish it via GitHub Pages,
absolutely for free.

I use Jekyll for my blog. And official instruction doesn't enough for me.
I got some issues, really small things, and the most one is a Jekyll error.
Trying to install Jekyll on the new machine, I got a error.

I need some package installed because some version of ruby binary are not
available for Fedora.

## Environment

In this article I use next version of installed components:
1. Fedora 30
2. [ruby][ruby] 2.6.3p62 (2019-04-16 revision 67580)

## Installation

### dnf install

I install next packages manually. It will pretty enough to install Jekyll.
```
sudo dnf install ruby ruby-devel rubygems-devel \
                 autoconf automake bison gcc-c++ libffi-devel libtool \
                 libyaml-devel readline-devel sqlite-devel zlib-devel \
                 openssl-devel redhat-rpm-config
```
So I am not sure that all of them are needed.

### update system

Update all system gems
```
sudo gem install rubygems-update
```

Then run it
```
sudo gem update --system
```
or (if previous not work)
```
sudo gem update --system --install-dir=/usr/share/gems --bindir /usr/local/bin
```

### install jekyll

Next, install gems for Jekyll and Bundler
```
sudo gem install jekyll bundler
```

Done! You may install new Jekyll site and test it locally :)


## Some notes

Inside Jekyll project directory, run `bundler install` and `bundler update --bundler`
to install and update the gems needed, then use `bundle exec jekyll serve -d public --incremental --verbose --watch` to run Jekyll on localhost. If success open URL:[localhost:4000](http://localhost:4000) in browser.

## Additional information

* [Getting started with GitHub Pages](https://help.github.com/en/articles/getting-started-with-github-pages) -
    Host a static site on GitHub.
* [Quickstart in Jekyll](https://jekyllrb.com/docs/) -
    Let's begin design your static site with Jekyll.
* [Jekyll, rvm, bundler and Fedora](https://robbinespu.gitlab.io/blog/2019/05/08/ruby-fedora-jekyll/)-
    The article that I inspired by.
* [Jekyll on Fedora][https://developer.fedoraproject.org/start/sw/web-app/jekyll.html] -
    Jekyll instruction from Fedora developers.
* [minimal-mistakes](https://github.com/mmistakes/minimal-mistakes) -
    The best theme for Jekyll that I've try. It is used in this blog. 

[jekyll]: https://jekyllrb.com/
[haproxy-package]: https://docs.netgate.com/pfsense/en/latest/packages/haproxy-package.html
