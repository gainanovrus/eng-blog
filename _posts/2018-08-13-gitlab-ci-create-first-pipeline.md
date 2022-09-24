---
published: true
layout: single
title: GitLab CI/CD. Create simple pipeline
excerpt: >-
  With using CI/CD pipelines in GitLab project you can easily build, test and
  deploy your code. This post will describe how to make it step by step.
categories: devops
tags: gitlab devops java
toc: true
header:
  teaser: /assets/images/gitlab-ci/gitlab-ci-simple-pipeline.png
  og_image: /assets/images/gitlab-ci/gitlab-ci-simple-pipeline.png

change-runner-type:
  - url: /assets/images/gitlab-ci/change-runner-type.png
    image_path: /assets/images/gitlab-ci/change-runner-type.png
    title: "Determine the landing project with a current Runner (nginx)"
---

Today I want to show you how to build and run a simple web application with
pipelines. I wrote an example Java application to demonstrate a possibility of CI.
The application code is placed [here][java-servlet-hello].

For using it in GitLab just clone and add as new project.
![create-new-project]({{ site.baseurl }}/assets/images/gitlab-ci/create-new-project.png){: .align-center}
![java-project-info]({{ site.baseurl }}/assets/images/gitlab-ci/java-project-info.png){: .align-center}


## About project

This is a typical and simple web application that shows a web page with
**"Hello world"** when a user gets
[`http://localhost:8081/`](http://localhost:8081/) from browser.
Firstly you should compile a project with Maven. In the result of it you have
got a file `hello.war`. For running the app copy this war file into a webapps
folder and run Tomcat WebServer.


## Runner host preparing

It supposes you have [configured][install-runner-post] a shell Runner and have
installed Docker.

After that for allowing to manage docker containers by Runner you should add
a gitlab-runner user into docker group.
```shell
usermod -a -G docker gitlab-runner
```

In addition I setup gitlab-runner as sudo user to run protected linux commands
without user passwords. Just add a next row into a file with a command
`sudo visudo -f /etc/sudoers.d/gitlab-runner`
```
gitlab-runner ALL=(ALL) NOPASSWD: ALL
```

In the end, enable a specific Runner to this project if you didn't do that yet.

![configure-stand-runner]({{ site.baseurl }}/assets/images/gitlab-ci/configure-stand-runner.png){: .align-center}

## Writing pipeline

In this project I want to use two stages to build and deploy an application.

![hello-pipeline]({{ site.baseurl }}/assets/images/gitlab-ci/hello-pipeline.png){: .align-center}

The stage `deploy` has two jobs to deploy the app into a stand and production
servers. Runners on these servers have tags `prod-shell` and `stand-shell`.
The deploy to the production server requires a manual action. To build an app
are used a docker maven container. The app is deploying with tomcat container.

Thus the next code will configure the defined pipeline.
It requires to be inputted in `.gitlab-ci.yml`:
```yaml
stages:
  - build
  - deploy

build_app:
  stage: build
  dependencies: []
  tags:
  - stand-shell
  script:
  - docker run -i --rm --name hello-maven -v ${PWD}:/hello -w /hello maven
      mvn clean install
  - cp target/hello.war hello.war
  - docker run -i --rm --name hello-maven -v ${PWD}:/hello -w /hello maven
      mvn clean
  artifacts:
    paths:
    - hello.war
    expire_in: 1 week

deploy:stand:
  stage: deploy
  dependencies:
  - build_app
  tags:
  - stand-shell
  script:
  - docker run -d --rm --name hello-tomcat-${CI_COMMIT_SHA:0:8} -P
      -v ${PWD}/hello.war:/usr/local/tomcat/webapps/hello.war   
      tomcat:9.0-jre8-alpine
  - docker ps -f "name=hello-tomcat-${CI_COMMIT_SHA:0:8}" --format '{% raw %}{{.Ports}}{% endraw %}'

deploy:prod:
  stage: deploy
  when: manual
  dependencies:
  - build_app
  tags:
  - prod-shell
  script:
  - docker run -d --rm --name hello-tomcat-${CI_COMMIT_SHA:0:8} -P
      -v ${PWD}/hello.war:/usr/local/tomcat/webapps/hello.war   
      tomcat:9.0-jre8-alpine
  - docker ps -f "name=hello-tomcat-${CI_COMMIT_SHA:0:8}" --format '{% raw %}{{.Ports}}{% endraw %}'
```



In the result we have an automation process that builds and deploy the web app.
The app is deployed to stand host for test purposes after every commit into a repo.

![hello-gitlab-pipelines]({{ site.baseurl }}/assets/images/gitlab-ci/hello-gitlab-pipelines.png){: .align-center}

The result pipeline is:

![java-project-info]({{ site.baseurl }}/assets/images/gitlab-ci/java-project-info.png){: .align-center}
![hello-gitlab-pipeline]({{ site.baseurl }}/assets/images/gitlab-ci/hello-gitlab-pipeline.png){: .align-center}

The app can be accessed by IP and PORT.  The PORT are dynamically generated and
are showed in the `deploy:stand` job result:

![hello-deploy-job-result]({{ site.baseurl }}/assets/images/gitlab-ci/hello-deploy-job-result.png){: .align-center}

In my job output port is `32768`. And the web app can be accessed by:

![hello-result]({{ site.baseurl }}/assets/images/gitlab-ci/hello-result.png){: .align-center}

With this configuration you have tried to use CI/CD process with GitLab. Don't
forget this simple pipeline doesn't provide jobs to stop stands. You should are
stopping unused docker containers yourself.

Next you can change it to your complex configuration for own purposes.

In the future I will explain how to use [Ansible](https://www.ansible.com/)
for CI/CD in the script block. It useful if you have a complex pipeline and you
want write readable scripts.

[java-servlet-hello]:https://github.com/GRomR1/java-servlet-hello

[install-runner-post]:{% post_url 2018-08-12-gitlab-ci-runner-install %}
