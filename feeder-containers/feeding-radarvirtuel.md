---
description: 'If you wish to feed RadarVirtuel, follow the steps below.'
---

# Feeding RadarVirtuel

The main goal of [RadarVirtuel](https://www.radarvirtuel.com/) is to collect data about flights. Although RadarVirtuel welcomes feeding stations from all over the world, their differentiator is to collect information about traffic around smaller airports around the world.

The docker image [`ghcr.io/sdr-enthusiasts/docker-radarvirtuel`](https://github.com/sdr-enthusiasts/docker-radarvirtuel) contains the required feeder software and all required prerequisites and libraries. This needs to run in conjunction with `readsb`, `tar1090`, or another RAW provider.

## Setting up Your Station

### Obtaining an RadarVirtuel Feeder Key

First-time users should obtain a RadarVirtuel Feeder key. To request one, email support@adsbnetwork.com with the following information:

* Your name
* The Lat/Lon and nearest airport of your station
* Your Raspberry Pi model (or other hardware if not Raspberry Pi)
* Mention that you will feed using a Docker container.

### Update `.env` file with RadarVirtuel Feeder Key

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favorite text editor. Beginners may find the editor `nano` easy to use:

```text
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our RadarVirtuel Feeder Key to this file. Add the following line to the file:

```text
RV_FEEDER_KEY=YOURFEEDERKEY
```

* Replace `YOURFEEDERKEY` with the key you received in response to your email.

For example:

```text
RV_FEEDER_KEY=xxxx:432143214473214732017432014747382140723
```

### Deploying feeder container

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file \(inside the `services:` section\).

```yaml
  radarvirtuel:
    image: ghcr.io/sdr-enthusiasts/docker-radarvirtuel:latest
    tty: true
    container_name: radarvirtuel
    hostname: radarvirtuel
    restart: always
    environment:
      - FEEDER_KEY=${RV_FEEDER_KEY}
      - SOURCE_HOST=readsb:30002
      - RV_SERVER=mg22.adsbnetwork.com:50050
      - VERBOSE=OFF
      - MLAT_SERVER=mlat.adsbnetwork.com:50000
      - MLAT_HOST=readsb:30005
      - LAT=${FEEDER_LAT}
      - LON=${FEEDER_LONG}
      - ALT=${FEEDER_ALT_M}
    depends_on:
      - readsb
    tmpfs:
      - /tmp:rw,nosuid,nodev,noexec,relatime,size=128M
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
```

To explain what's going on in this addition:

* We're creating a container called `radarvirtuel`, from the image `ghcr.io/sdr-enthusiasts/docker-radarvirtuel`.
* We're passing several environment variables to the container:
  * `FEEDER_KEY` contains the key that you added to `.env` as per the instructions above
  * `SOURCE_HOST` indicates where to get the RAW data from
  * `RV_SERVER` is the address of the RadarVirtuel server where your data will be sent. Please do not change this unless you're specifically instructed to
  * `VERBOSE` can be `ON` (meaning: show lots of information in the docker logs) or `OFF` (show only errors in the docker logs)
  * Enabling receiving MLAT RAW data and sending latitude, longitude and altitude from the .env file
* The mounted volumes make sure that the container will use the same timezone as your host system

Once the file has been updated, issue the command `docker compose pull radarvirtuel && docker compose up -d` in the application directory to apply the changes and bring up the `radarvirtuel` container. You should see the following output:

```text
readsb is up-to-date
adsbx is up-to-date
piaware is up-to-date
fr24 is up-to-date
pfclient is up-to-date
Creating radarvirtuel...
```

We can view the logs for the environment with the command `docker logs radarvirtuel`, or continually "tail" them with `docker logs -f radarvirtuel`. The logs will be fairly unexciting and look like this:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] done.
[services.d] starting services
[radarvirtuel/radarvirtuel][Tue May  4 17:06:32 EDT 2021] RadarVirtuel was started as an s6 service
[services.d] done.
[radarvirtuel/imalive][Tue May  4 17:06:32 EDT 2021] Started as an s6 service
```

Once running, you can visit [https://alpha.radarvirtuel.com/stations/xxxx](https://alpha.radarvirtuel.com/stations/xxxx) (replace "xxxx" with the name of your station, which is the first part of the Feeder Key you received) to view the data you are feeding to RadarVirtuel. For example: [https://alpha.radarvirtuel.com/stations/KBOS](https://alpha.radarvirtuel.com/stations/KBOS).

## Troubleshooting

Most log messages are self-explanatory and have suggestions on how to trouble-shoot your issue. Here is some additional information that may help:

* Sometimes, the logs may show error messages that it cannot connect to your `SOURCE_HOST`. If these messages show every few seconds, you have a problem (read below). If there are no new messages after a bit, it means that your station finally connected to the `SOURCE_HOST`. This connection delay is often caused by RadarVirtuel becoming "up and running" before `tar1090` or `readsb` do. This will fix itself within less than a minute.
* This message keeps on scrolling and it doesn't stop after a while. In that case, `tar1090` or `readsb` cannot be reached.
  * If you configured `readsb`, try adding this parameter to the `environment:` section in `docker-compose.yml`:
    `- READSB_NET_RAW_OUTPUT_PORT=30002`
  * If you configured `tar1090`, there's nothing else to configure. Make sure the `tar1090` container is up and running and is receiving data!
* You see log messages about the Feeder Key being incorrect. This is quite self-explanatory: check your feeder key.
* You see messages about not being able to reach the RadarVirtuel Server. This may be a temporary outage. If the message consists for several hours, please contact support@adsbnetwork.com to see if there's something going on.

## More information and support

* There is extensive documentation available on the container's [GitHub](https://github.com/sdr-enthusiasts/docker-radarvirtuel) page.
* RadarVirtuel and ADSBNetwork are owned and operated by Laurent Duval, who can be reached at support@adsbnetwork.com
* You can always find help on the #adsb-containers channel on the [SDR Enthusiasts Discord server](https://discord.gg/m42azbZydy). This channel is meant for Noobs (beginners) and Experts alike.
