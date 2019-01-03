---
published: true
layout: single
title:  "GitLab CI/CD"
excerpt: >-
  GitLab CI/CD takes many benefits for companies that want to automate software
  development workflow. I use it in my practice for a long time and want to
  documents my experience.
categories: devops
tags: gitlab devops
toc: false
header:
  teaser: /assets/images/gitlab-ci-teaser.png
  og_image: /assets/images/gitlab-ci-teaser.png
---

Today I want to start writing a cycle of articles about CI/CD process with GitLab.
Many developers use GitLab to store code and work together on one project (write
and review code, make issues, user management, etc.). But it has many other
built-in tools includes automate test and deploy process.

With Gitlab you can easily build, test and deploy your code. In our company we
are using Gitlab CI/CD for a long time. I will share my experience with you.

So these topics will be described in next articles:
1. [GitLab CI/CD. Main concepts and terms][main-concepts]
2. [GitLab CI/CD. Install and configure Runner][install-runner-post]
3. [GitLab CI/CD. Create simple pipeline][create-first-pipeline]
4. GitLab CI/CD. Tips and tricks

----
__Official documentation__
* [GitLab CI/CD Documentation](https://docs.gitlab.com/ce/ci/)
* [Getting started with GitLab CI/CD](https://docs.gitlab.com/ce/ci/quick_start/README.html)
* [Configuration of your jobs with .gitlab-ci.yml](https://docs.gitlab.com/ce/ci/yaml/README.html)

[main-concepts]:{% post_url 2018-08-11-gitlab-ci-main-concepts %}
[install-runner-post]:{% post_url 2018-08-12-gitlab-ci-runner-install %}
[create-first-pipeline]:{% post_url 2018-08-13-gitlab-ci-create-first-pipeline %}
