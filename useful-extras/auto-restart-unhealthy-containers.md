---
description: >-
  The following steps will guide you through deploying the
  "willfarrell/autoheal" container, which will restart any containers that
  become unhealthy.
---

# Auto-Restart Unhealthy Containers

[As previously discussed](../foundations/common-tasks-and-info.md#information-on-healthchecks), we try to include [healthchecks](https://docs.docker.com/engine/reference/builder/) in all our images. You should see a health status in the `STATUS` column of each container created in this guide whenever you issue the `docker ps` command.

For example:

```text
STATUS
Up 2 hours (healthy)
```

If a container becomes unhealthy for any reason \(for example: perhaps the SDR hardware "wedges", perhaps a program running in a container becomes deadlocked, etc\), then we would naturally want to restart it. Ideally, we'd want this to happen automatically.

Unfortunately, Docker does not yet offer this functionality. [It has been proposed but not yet implemented](https://github.com/moby/moby/issues/28400).

Thankfully, the container `willfarrell/autoheal` has been created that will regularly check containers and restart any with an unhealthy status.

There are two ways to implement `willfarrell/autoheal`:

1. Monitor all containers and restart any in an unhealthy state
2. Only monitor and restart a specific set of containers

This page will discuss both methods.

## A Word About Security

The `willfarrell/autoheal` image requires that the container has access to the host's docker socket: `/var/run/docker.sock`.

Mounting `/var/run/docker.sock` inside a container effectively gives the container and anything running within it root privileges on the underlying host, since now you can do anything that a root user and a member of the `docker` group can.

For example, you could create a container, mount the host's `/etc`, modify configurations to open an attack vector to take over the host.

The image has been around since 2017, has several hundred stars, [the source code is available](https://github.com/willfarrell/docker-autoheal/blob/main/docker-entrypoint) for public scrutiny and many people run it \(myself included\), so the risk of nefarious behaviour by the author is \(in my opinion\) fairly low. Nevertheless, if you decide to give this container access to the host's `/var/run/docker.sock`, you do so at your own risk.

## Monitor and Restart All Containers

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  autoheal:
    image: willfarrell/autoheal:latest
    container_name: autoheal
    restart: unless-stopped
    environment:
      - AUTOHEAL_CONTAINER_LABEL=all
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

To explain what's going on in this addition:

* We're creating a container called `autoheal`, from the image `willfarrell/autoheal:latest`.
* We're passing several environment variables to the container:
  * `AUTOHEAL_CONTAINER_ALL=all` to inform autoheal to monitor all containers
* We're passing through the docker socket `/var/run/docker.sock` so that autoheal can control docker \(to restart containers\).

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `autoheal` container. You should see the following output:

```text
ultrafeeder is up-to-date
piaware is up-to-date
fr24 is up-to-date
pfclient is up-to-date
rbfeeder is up-to-date
adsbhub is up-to-date
opensky is up-to-date
Creating autoheal
```

The container does not log any output, so check it is running by issuing the command `docker ps` and finding the `autoheal` container. It should have a status of `Up 20 seconds (healthy)` or similar.

The `autoheal` container logs when it restarts an unhealthy container, for example:

```text
15-12-2020 20:17:06 Container /fr24 (51bc3e3511f5) found to be unhealthy - Restarting container now with 10s timeout
```

## Only monitor and restart a specific set of containers

In order for us to inform`autoheal` which containers to monitor, we need to apply a label to them.

### Label Existing Containers

To tell `autoheal` to monitor your `ultrafeeder` container, you would add the following configuration directive under its service definition in your `docker-compose.yml` file:

```text
labels:
  - "autoheal=true"
```

Thus, your updated `ultrafeeder` service may then look like this \(note the added `labels:` section\):

```yaml
version: '3.8'

volumes:
  readsbpb_rrd:
  readsbpb_autogain:

services:
  readsb:
    image: ghcr.io/sdr-enthusiasts/docker-readsb-protobuf:latest
    container_name: readsb
    hostname: readsb
    restart: unless-stopped
    labels:
      - "autoheal=true"
    devices:
      - /dev/bus/usb
    ports:
      - 8080:8080
    environment:
      - TZ=${FEEDER_TZ}
      - READSB_DEVICE_TYPE=rtlsdr
      - READSB_RTLSDR_DEVICE=00001090
      - READSB_GAIN=autogain
      - READSB_LAT=${FEEDER_LAT}
      - READSB_LON=${FEEDER_LONG}
      - READSB_RX_LOCATION_ACCURACY=2
      - READSB_STATS_RANGE=true
      - READSB_NET_ENABLE=true
    volumes:
      - readsbpb_rrd:/run/collectd
      - readsbpb_autogain:/run/autogain
```

Update the service definitions for all of the containers you want `autoheal` to monitor.

### Deploy the `autoheal` container

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  autoheal:
    image: willfarrell/autoheal:latest
    container_name: autoheal
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

To explain what's going on in this addition:

* We're creating a container called `autoheal`, from the image `willfarrell/autoheal:latest`.
* We're passing through the docker socket `/var/run/docker.sock` so that autoheal can control docker \(to restart containers\).

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `autoheal` container. You should see the following output:

```text
Recreating ultrafeeder
piaware is up-to-date
fr24 is up-to-date
pfclient is up-to-date
rbfeeder is up-to-date
adsbhub is up-to-date
opensky is up-to-date
Creating autoheal
```

Note that any containers you added the `label:` directive to were recreated by `docker compose` to reflect the updated configuration. Cool huh?

The container does not log any output, so check it is running by issuing the command `docker ps` and finding the `autoheal` container. It should have a status of `Up 20 seconds (healthy)` or similar.

The `autoheal` container logs messages when it restarts an unhealthy container, for example:

```text
15-12-2020 20:17:06 Container /adsbx (51bc3e3511f5) found to be unhealthy - Restarting container now with 10s timeout
```
