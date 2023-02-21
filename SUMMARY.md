# Table of contents

* [ADS-B Reception, Decoding & Sharing with Docker](README.md)

## Intro

* [Changes From the Old Guide](intro/changes-from-the-old-guide.md)
* [Overview](intro/overview.md)
* [How to Get Help](intro/how-to-get-help.md)
* [Why Docker?](intro/why-docker.md)
* [Equipment Needed](intro/equipment-needed.md)
* [Information Needed](intro/information-needed.md)

## Setting Up The Host System

* [Install Docker](setting-up-the-host-system/install-docker.md)
* [Configure Docker](setting-up-the-host-system/configure-docker.md)
* [Install Docker Compose](setting-up-the-host-system/install-docker-compose.md)

## Setting Up RTL-SDRs

* [Blacklist Kernel Modules](setting-up-rtl-sdrs/blacklist-kernel-modules.md)
* [Re-Serialise SDRs](setting-up-rtl-sdrs/re-serialise-sdrs.md)

## Foundations

* [Prepare the Application Environment](foundations/prepare-the-project-environment.md)
* [Deploy "readsb"](foundations/deploy-readsb-container.md)
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
* [Feeding "new" aggregators](feeder-containers/feeding-new-aggregators.md)
* [Feeding ADS-B Exchange](feeder-containers/feeding-ads-b-exchange.md)

## Useful Extras

* [Improved Visualisation with tar1090](useful-extras/improved-visualisation-with-tar1090.md)
* [Alternative Graphing with Influx and Grafana](useful-extras/alternative-graphing-with-prometheus-grafana.md)
* [Alternative Graphing with Prometheus and Grafana](useful-extras/alternative-graphing-with-influx-grafana.md)
* [Auto-Restart Unhealthy Containers](useful-extras/auto-restart-unhealthy-containers.md)
* [Auto-Upgrade Containers](useful-extras/auto-upgrade-containers.md)
* [Managing a remote station using ZeroTier](useful-extras/managing-a-remote-station.md)
