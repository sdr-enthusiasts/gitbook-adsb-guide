# Table of contents

* [ADS-B Reception, Decoding & Sharing with Docker](README.md)

## Intro

* [Overview](intro/overview.md)
* [How to Get Help](intro/how-to-get-help.md)
* [What is Docker?](intro/what-is-docker.md)
* [Why Docker?](intro/why-docker.md)
* [Equipment Needed](intro/equipment-needed.md)
* [Information Needed](intro/information-needed.md)

## Setting Up The Host System

* [Preparing Your System](setting-up-the-host-system/preparing-your-system.md)
* [Install Docker](setting-up-the-host-system/running-docker-install.md)

## Setting Up RTL-SDRs

* [Blacklist Kernel Modules](setting-up-rtl-sdrs/blacklist-kernel-modules.md)
* [Re-Serialise SDRs](setting-up-rtl-sdrs/re-serialise-sdrs.md)

## Foundations

* [Prepare the Application Environment](foundations/prepare-the-project-environment.md)
* [Deploy "ultrafeeder"](foundations/deploy-ultrafeeder-container.md)
* [Deploy "dump978" \(USA Only\)](foundations/deploy-dump978-usa-only.md)
* [Container Monitoring and Management](foundations/common-tasks-and-info.md)

## Feeder Containers

* [Feeding Plane.watch](feeder-containers/feeding-plane-watch.md)
* [Feeding FlightAware \(piaware\)](feeder-containers/feeding-flightaware-piaware.md)
* [Feeding FlightRadar24](feeder-containers/feeding-flightradar24.md)
* [Feeding RadarBox](feeder-containers/feeding-radarbox.md)
* [Feeding PlaneFinder](feeder-containers/feeding-planefinder.md)
* [Feeding ADSBHub](feeder-containers/feeding-adsbhub.md)
* [Feeding OpenSky Network](feeder-containers/feeding-opensky-network.md)
* [Feeding RadarVirtuel](feeder-containers/feeding-radarvirtuel.md)
* [Feeding ADS-B Exchange](feeder-containers/feeding-ads-b-exchange.md)

## Useful Extras

* [Improved Visualisation with tar1090](useful-extras/improved-visualisation-with-tar1090.md)
* [Storing Data and Metrics in a Time Series Database](useful-extras/storing-time-series-data.md)
* [Graphing Data and Metrics with Influx and Grafana](useful-extras/alternative-graphing-with-influx-grafana.md)
* [Graphing Data and Metrics with Prometheus and Grafana](useful-extras/alternative-graphing-with-prometheus-grafana.md)
* [Auto-Restart Unhealthy Containers](useful-extras/auto-restart-unhealthy-containers.md)
* [Auto-Upgrade Containers](useful-extras/auto-upgrade-containers.md)
* [Managing a remote station using ZeroTier](useful-extras/managing-a-remote-station.md)
