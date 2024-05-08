---
description: >-
  If you wish to deploy Grafana and Prometheus for alternative graphs, follow the steps below.
---

# Alternative graphs with Grafana using Prometheus

[`Grafana`](https://grafana.com/) is an analytics platform that can provide alternative graphs for `ultrafeeder`. It reads data from database; here we will be using [`Prometheus`](https://prometheus.io/) as the database.

Using Grafana and Prometheus in this configuration does not require a plan, account, or credentials for their respective cloud offerings.

At the bottom of this page, you can see an example of what a Grafana dashboard can look like.

## Setting up and using Prometheus and Grafana

As a prerequisite, we will assume that you already have a working deployment of the [docker-tar1090](https://github.com/sdr-enthusiasts/docker-tar1090) or [docker-adsb-ultrafeeder](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder) container to receive ADSB data. Optionally, if you are using [docker-dump978](https://github.com/sdr-enthusiasts/docker-dump978) to receive UAT data, you can also include this in your Grafana setup.

For Grafana to work, you will need to install and configure a few extra things:

- ensuring that your `docker-tar1090`, `docker-adsb-ultrafeeder`, and/or `docker-dump978` containers are set up so they make data available to Prometheus
- a (containerized) Prometheus database instance that reads data from `docker-tar1090`, `docker-adsb-ultrafeeder`, and/or `docker-dump978`
- a (containerized) Grafana instance that is the platform for creating and hosting the graphs
- a Grafana Dashboard that contains the actual graphs

For step-by-step instructions on how to implement this, please see the [Grafana-specific README](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder/blob/main/README-grafana.md) in the [docker-adsb-ultrafeeder](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder) repository.

![image](https://user-images.githubusercontent.com/15090643/234161588-69cd1888-6d9c-42f2-90d9-8eb108b0dce5.png)
![image](https://user-images.githubusercontent.com/15090643/234161718-845d3836-005e-4d38-ba45-9c59873c8db9.png)
![image](https://user-images.githubusercontent.com/15090643/234161841-fde61d66-2f64-43f6-8e71-4152eef76f72.png)
