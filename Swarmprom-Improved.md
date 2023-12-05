# SWIM - SWarmprom IMproved

The original [Swarmprom](https://github.com/stefanprodan/swarmprom) is excellent, but it is no longer maintained and has become outdated.
As of writing this post, it has been over 3 years since there was an update to the repository and it has now been archived.

Here is a up to date version of Swarmprom, that I call Swarmprom Improved aka SWIM.
The main idea is to make it compatible with the latest versions (as of September 2023) of Prometheus, Grafana, cAdvisor, Node Exporter, Alert Manager and ~~Unsee~~ Karma, the same programs, or stack, used by the original Swarmprom. Karma is a fork of Unsee by the original developer.

Here is the up to date [swarmprom-improved-compose.yaml](swarmprom-improved-compose.yaml) file.

Deploy it using this command: `docker swarm deploy -c <path/to/swarmprom-improved-compose.yaml> swim`

## Deploy

Default Deploy (without AlertManager and Karma):

```bash
git clone https://github.com/persunde/swarmprom-improved.git
cd swarmprom-improved
docker stack deploy -c swarmprom-improved-compose.yaml swim
```

If you want to include AlertManager and Karma, then:

1. Modify the [swarmprom-improved-compose.yaml](swarmprom-improved-compose.yaml) file, and set the `replicas` to 1 for both AlertManager and Karma.
2. Get a slack token/hook-URL.
3. Deploy using the below command:

```bash
$ git clone https://github.com/persunde/swarmprom-improved.git
$ cd swarmprom-improved
$ SLACK_URL=https://hooks.slack.com/services/<TOKEN> \
SLACK_CHANNEL=devops-alerts \
SLACK_USER=alertmanager \
docker stack deploy -c swarmprom-improved-compose.yaml swim
```

### Verify that it works

- Check that all the SWIM services are working by running: `docker stack ps swim`  
- Access Grafana on from your browser on `http://<swarm-ip>:3000`

See the original Swarmporm instructions for more detailed information about how to configure Alertmanager: <https://github.com/stefanprodan/swarmprom#instructions>

### Prerequisites

- Docker 20.10.17 or higher
  - Probably works with 17+ versions as well, but I have only tested on Docker 20.10 and higher
- Swarm cluster with one manager and 0 or more workers
  - The swarm cluster must have 576 MB or more available memory that can be reserved by the SWIM services

### Services

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
- Added the amazing Grafana dashboard [Node Exporter Full (1860)](https://grafana.com/grafana/dashboards/1860-node-exporter-full/) (REMOVE?)
- Updated and modified previous Grafana dashboards
- <TODO>:: add a list of Grafana dashboards the user should add, either just add as a list or somehow embedded it into the yaml file or something

## Persistence storage

The `swarmprom-improved-compose.yaml` deployment will not keep the data between container restarts. To do that you need to handle it yourself.

See the comments in the [swarmprom-improved-compose.yaml](swarmprom-improved-compose.yaml) file for how to add data persistence to Prometheus and Grafana.

Assuming you run a multi-machine setup, the Grafana configuration and the Prometheus data needs to be saved in a persistent storage that is independent of the container, so that the data will be not be lost even when the machine or container restarts. You probably also want to save the Grafana configuration and dashboards between deployments or container restarts.

Here are some suggestions on how to achieve persistent storage for Prometheus and Grafana:

- Set Placement constraints on the Grafana and Prometheus containers, so that they always run on the same node, and use that node's local persistent storage.
- Use a shared network drive (NFS), or whatever distributed storage your cloud provider provides.
  - [Longhorn](https://longhorn.io/) can be used in K8S to provide distributed storage
  - AWS offers [EFS](https://aws.amazon.com/efs/) and it can [handle up to 500 MB/s](https://docs.aws.amazon.com/efs/latest/ug/performance.html#performance-overview)
  - Or something like [Thanos](https://thanos.io/) if your service generates lots of data
