---
title: "Monitoring system for Docker SLURM cluster"
excerpt: "Design and deploy a system for collect, store and visualize data from SLURM cluster realized on Docker environment."
header:
  image: assets/images/influxdb-slurm-monitoring/influxdb-slurm-monitoring-banner.png
  teaser: assets/images/influxdb-slurm-monitoring/influxdb-slurm-monitoring-teaser.png
sidebar:
  - title: "Role"
    image: assets/images/influxdb-slurm-monitoring/icmm_logo-min.png
    image_alt: "logo"
    text: "DevOps engineer"
  - title: "Responsibilities"
    text: "Design a system for monitoring tasks queue, node loads and system statics of SLURM cluster in Institute of Continuous Media Mechanics (ICMM UB RAS)"
gallery:
  - url: assets/images/influxdb-slurm-monitoring/influxdb-slurm-monitoring-scheme.png
    image_path: assets/images/influxdb-slurm-monitoring/influxdb-slurm-monitoring-scheme.png
    alt: "Scheme of SLURM cluster and monitoring system"
  - url: assets/images/influxdb-slurm-monitoring/grafana-all-panels.png
    image_path: assets/images/influxdb-slurm-monitoring/grafana-all-panels.png
    alt: "An example dashboard to view system statistics"
---
Institute of Continuous Media Mechanics ([ICMM UB RAS][ICMM UB RAS]) has a high-performance cluster consist of about 50 nodes with >400 CPUs and more than 1 TB RAM.
It's needed to make models of new materials and calculate its properties.
And in the future, the Institute has plans to upgrade his infrastructure and virtualize his resources to get less dependencies of a hardware layer.
The cluster uses [Slurm workload manager][SLURM] to run programs on compute nodes.
With SLURM many scientist can easy run his programs on the cluster in parallel and independent.

And I should work with these systems.
My primary task is to make scripts for deploying virtual compute infrastructure with [Docker][Docker].
My second task is to design and configure a monitoring system to collect and view system statics in the real-time.

Has been developed a script that deploying a simple SLURM configuration with docker-compose.
Instead of docker-compose could be used [docker stack][stack] command with Docker [Swarm][Swarm] to deploy containers to a compute nodes with included SLURM software.
All information is described on [GitHub][docker-slurmbase].

I have used a stack [InfluxDB](http://influxdata.com)+[Grafana](http://grafana.com), I've known good from my [previous][visualisation-jinr-cloud] work, to realize a monitoring system.
I use HDF5 profiling plugin for collecting data about runned tasks.
This plugin was based of work of the [cfenoy][cfenoy] user.
I've modified it to work with the last SLURM version and add gathering some new fields.
Also was configured a Grafana dashboard that helps view all statistics about task that have been ever run on the HPC cluster.

----------------------------------------------------------------------
### docker-slurmbase:

[https://github.com/GRomR1/docker-slurmbase](https://github.com/GRomR1/docker-slurmbase)


### influxdb-slurm-monitoring:

[https://github.com/GRomR1/influxdb-slurm-monitoring](https://github.com/GRomR1/influxdb-slurm-monitoring)

----------------------------------------------------------------------

{% include gallery %}

[ICMM UB RAS]: https://www.icmm.ru/en/about-institute
[SLURM]: https://slurm.schedmd.com/overview.html
[Docker]: https://www.docker.com/
[visualisation-jinr-cloud]: _portfolio/visualisation-jinr-cloud.md
[Swarm]: https://docs.docker.com/engine/swarm/
[stack]: https://docs.docker.com/engine/reference/commandline/stack/
[docker-slurmbase]: https://github.com/GRomR1/docker-slurmbase
[influxdb-slurm-monitoring]: https://github.com/GRomR1/influxdb-slurm-monitoring
[cfenoy]: https://github.com/cfenoy/influxdb-slurm-monitoring
