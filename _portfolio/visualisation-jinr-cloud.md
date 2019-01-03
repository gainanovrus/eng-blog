---
title: "Visualization of the JINR cloud resources utilization"
excerpt: "A system that gathering data from a cloud infrastructure based on OpenNebula and showing it with Grafana."
header:
  image: assets/images/visualisation-jinr-cloud/visualisation-jinr-cloud-scheme.png
  teaser: assets/images/visualisation-jinr-cloud/visualisation-jinr-cloud-scheme-th-min.png
sidebar:
  - title: "Role"
    image: assets/images/visualisation-jinr-cloud/jinr-logo.png
    image_alt: "logo"
    text: "Analyst, Architect, Developer"
  - title: "Responsibilities"
    text: "Design a system to get statistics of the JINR cloud resources utilization"
gallery:
  - url: assets/images/visualisation-jinr-cloud/visualisation-jinr-cloud-1-min.png
    image_path: assets/images/visualisation-jinr-cloud/visualisation-jinr-cloud-1-min.png
    alt: "Resource allocation between clusters"
  - url: assets/images/visualisation-jinr-cloud/visualisation-jinr-cloud-2-min.png
    image_path: assets/images/visualisation-jinr-cloud/visualisation-jinr-cloud-2-min.png
    alt: "Resource allocation by Department"
  - url: assets/images/visualisation-jinr-cloud/visualisation-jinr-cloud-3-min.png
    image_path: assets/images/visualisation-jinr-cloud/visualisation-jinr-cloud-3-min.png
    alt: "Resource usage statistics"
---

To visualize statistics on a distribution of resources of the [JINR](http://jinr.ru) cloud the [Grafana](http://grafana.com) system was chosen. 
It provides a user-friendly interface through a web browser displaying various kinds of statistical metrics in real-time, 
gives flexible and functional ways to customize the layout of charts and graphs.

As data-storage to store gathering information from OpenNebula was chosen [InfluxDB](http://influxdata.com). 
It is an open source database specifically to handle time series data with high availability and high performance requirements. 
InfluxDB is meant to be used as a backend storage for many use cases involving large amounts of timestamped data, 
including DevOps monitoring, application metrics and real-time analytics. 
It has a simple, high performing write and query HTTP(S) APIs. 

{% include gallery caption="Developed dashboards" %}
