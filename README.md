# SWarmprom-IMproved aka SWIM

SWarmprom-IMproved, also known as SWIM, is an updated version of the original [Swarmprom project](https://github.com/stefanprodan/swarmprom).

SWIM uses the up to date versions (as of September 2023) of Prometheus, Grafana, cAdvisor, Node Exporter, Alert Manager and ~~Unsee~~ Karma. This is the same original stack that is used in Swarmprom. Unsee is replaced by Karma, which is a fork of Unsee by the original developer.

## About SWIM

The aim for SWIM is to be as easy to deploy and use as the original Swarmprom, but use up to date versions of the software used, be easier to maintain, configure and extend for different situations.

SWIM uses the official distributed container images. This makes upgrading or downgrading to different versions is easy to do, as as switching to custom built images, and also to mix the different versions of the different programs depending on your needs.

The original Swarmprom used custom built container images that only supported AMD64 architecture and was locked to one specific version for all the programs used. SWIM makes it easier to deploy on ARM64 as well as AMD64 hardware. It should also be easier to deploy on new and less popular hardware, such as various ARM versions, RISC-V and other non-AMD64 architectures that are increasingly gaining a lot of popularity.

To deploy a different image than the default, you need to modify the compose file to use different container image(s). If the version of the software does not exist as an container image, or your hardware architecture is not supported, then you can in theory build them yourself from source since all the software used by SWIM is OSS with the Apache License, Version 2.0.

Please help improve SWIM by submitting PRs and/or add write suggestions on how to improve SWIM. For suggestions please use the GitHub discussion forum or create an GitHub issue.

Have a good day or evening wherever you are.

## Prerequisites

- Docker 20.10.17 or higher
  - Probably works with 17+ versions as well, but I have only tested on Docker 20.10 and higher
- Docker Swarm cluster with one manager and 0 or more workers
  - The swarm cluster must have 576 MB or more available memory that can be reserved by the SWIM services


## Deploy

Here is the up to date [swarmprom-improved-compose.yaml](swarmprom-improved-compose.yaml) file. Modify it to your liking before deploying, or use it as is. To deploy run the below command:

```bash
git clone https://github.com/persunde/swarmprom-improved.git
cd swarmprom-improved
docker stack deploy -c swarmprom-improved-compose.yaml swim
```

If you want AlertManager to send slack notifications, then deploy like this:

```bash
$ git clone https://github.com/persunde/swarmprom-improved.git
$ cd swarmprom-improved
$ SLACK_URL=https://hooks.slack.com/services/<TOKEN> \
SLACK_CHANNEL=devops-alerts \
SLACK_USER=alertmanager \
docker stack deploy -c swarmprom-improved-compose.yaml swim
```

See the original Swarmporm instructions for more detailed information about how to configure Alertmanager: <https://github.com/stefanprodan/swarmprom#instructions>

### Verify that the deployment works

- Check that all the SWIM services are working by running: `docker stack ps swim`  
- Access Grafana on from your browser on `http://<swarm-ip>:3000`
  - Check that you can see live data on the dashboards

### Services deployed with SWIM

- prometheus (metrics database) `http://<swarm-ip>:9090`
- grafana (visualize metrics) `http://<swarm-ip>:3000`
- node-exporter (host metrics collector)
- cadvisor (containers metrics collector)
- dockerd-exporter (Docker daemon metrics collector)
- caddy (reverse proxy and basic auth provider for prometheus, alertmanager and karma)
- alertmanager (alerts dispatcher) `http://<swarm-ip>:9093`
- Karma (alert manager dashboard) `http://<swarm-ip>:9094`

## Changes from Swarmprom

These are the main changes from the original Swarmprom that are in SWarmprom IMproved:

- The yaml file to includes the **latest stable versions** (as of September 2023) of
  - Prometheus
  - Grafana
  - cAdvisor
  - Node Exporter
  - Alert Manager
  - ~~Unsee~~ replaced by [Karma](https://github.com/prymitive/karma)
- Uses the official images from each project, to make it easier to change the versions of each program the user wants to use.
- **ARM64 compatability**. 
  - Tested on on AWS Graviton instances.
  - All the images have ARM64/ARMv8 and AMD64 releases

TODO:
- Added the amazing Grafana dashboard [Node Exporter Full (1860)](https://grafana.com/grafana/dashboards/1860-node-exporter-full/)
- Updated Grafana dashboards
- Add a list of Grafana dashboards the user should add, either just add as a list or somehow embedded it into the yaml file or something

## Persistence storage

The `swarmprom-improved-compose.yaml` deployment will not keep the data between container restarts. To do that you need to handle it yourself.

Assuming you run a multi-machine setup, the Grafana configuration and the Prometheus data needs to be saved in a persistent storage that is independent of the container, so that the data will be not be lost even when the machine or container restarts. You probably also want to save the Grafana configuration and dashboards.

Here are some suggestions on how to achieve persistent storage for Prometheus and Grafana:

- Set Placement constraints on the Grafana and Prometheus containers, so that they always run on the same node, and use that node's local persistent storage.
- Use a shared network drive (NFS), or whatever distributed storage your cloud provider provides.
  - [Longhorn](https://longhorn.io/) can be used in K8S to provide distributed storage
  - AWS offers [EFS](https://aws.amazon.com/efs/) and it can [handle up to 500 MB/s](https://docs.aws.amazon.com/efs/latest/ug/performance.html#performance-overview)
  - Or something like [Thanos](https://thanos.io/) if your service generates lots of data
