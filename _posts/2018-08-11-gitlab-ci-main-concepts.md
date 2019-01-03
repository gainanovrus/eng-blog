---
published: true
layout: single
title:  GitLab CI/CD. Main concepts and terms
excerpt: >-
  Before we will create the first pipeline with GitLab CI we should understand
  what pipeline is it. I want to explain main terms in this post.
categories: devops
tags: gitlab devops
toc: true
header:
  teaser: /assets/images/gitlab-ci/gitlab-ci-devops.png
  og_image: /assets/images/gitlab-ci/gitlab-ci-devops.png
---

If you want to use Continuous Integration (CI) and Continuous Delivery (CD)
services you need to know basic terms using in GitLab. I selected the foremost
ones and set links to the description and additional information about them.

## Configuration file
For using a CI/CD service you need to create a [.gitlab-ci.yml][gitlab-ci.yml]
file to the root directory of your repository. This file is used by GitLab
Runner to manage your project's jobs and stored in a
[YAML](https://en.wikipedia.org/wiki/YAML) format.

## Runner  
[GitLab Runner][runner] is a daemon in a host machine that is used to running
your jobs and send the results back to GitLab. It permanently holds a connect
with Gitlab. When a user runs job by pipeline Runner executes commands from
.gitlab-ci.yml on the host.

## Job
[Jobs][job] are set of commands that stored in section `script` in a config file.
GitLab Runner runs jobs in a host machine. Each job is run independently of each
other. The most popular jobs are a build_web_app1, build_web_app2, prepare, test,
deploy, and etc.

## Stage
A [stage][stage] allows to group jobs, and jobs of the same stage are executed
in parallel. Jobs of the next stage are run after the jobs from the previous
stage complete successfully.

## Pipeline
A [pipeline][pipeline] is a group of jobs that get executed in stages (batches). All of the
jobs in a stage are executed in parallel and if they all succeed the pipeline
moves on to the next stage. If one of the jobs fails the next stage is not
(usually) executed. You can access the pipelines page in your project's
Pipelines tab.

## Artifact
An [artifact][artifact] is a list of files and directories which are attached
to a job after it completes successfully. The uploaded artifacts will be kept
in GitLab during expiry period. You can download the artifacts archive or
browse its contents in Job Info page.

## Environment
GitLab provides a full history of your deployments per every environment.
[Environments][environment] are like tags for your CI jobs, describing where code gets deployed.
GitLab keeps track of your deployments, so you always know what is currently
being deployed on your servers. For example, you can create and use typical
environments - production and staging.

## Dependency
When defined jobs exist in a [`dependency`][dependency] block of your .gitlab-ci.yml the
Runner should be download all artifacts before start the current job.
It can be used to divide build and deploy jobs for example.
The deploy job often uses generated artifacts on previous stages.

Next post will describe how to [install and configure a GitLab Runner][install-runner] for using
it in your first pipeline.

[gitlab-ci.yml]:https://docs.gitlab.com/ce/ci/yaml/README.html
[runner]:https://docs.gitlab.com/runner
[job]:https://docs.gitlab.com/ee/ci/pipelines.html#jobs
[job-add]:https://docs.gitlab.com/ee/ci/yaml/README.html#jobs
[stage]:https://docs.gitlab.com/ee/ci/yaml/README.html#stages
[pipeline]:https://docs.gitlab.com/ee/ci/pipelines.html#pipelines
[artifact]:https://docs.gitlab.com/ce/user/project/pipelines/job_artifacts.html#defining-artifacts-in-gitlab-ci-yml
[environment]:https://docs.gitlab.com/ee/ci/environments.html
[dependency]:https://docs.gitlab.com/ee/ci/yaml/README.html#dependencies

[install-runner]:{% post_url 2018-08-12-gitlab-ci-runner-install %}
