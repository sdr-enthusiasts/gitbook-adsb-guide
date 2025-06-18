---
description: 'If you wish to feed Radar1090 UK, follow the steps below.'
---

# Feeding Radar1090 UK

[`Radar1090 UK`] (https://www.1090mhz.uk/) is an aggregator based in the UK. They are mostly interested in getting data feeds from the UK, the Republic of Ireland, and its direct neighboring countries, so if you are located in their operating area, feel free to start feeding them.

The docker image [`ghcr.io/sdr-enthusiasts/docker-radar1090`](https://github.com/sdr-enthusiasts/docker-radar1090) contains the required feeder software and all required prerequisites and libraries. This needs to run in conjunction with `ultrafeeder`, `tar1090`, or another RAW provider.

## Setting up Your Station

### Obtaining an Radar1090 UK Feeder Key

First-time users should obtain a Radar1090 UK Feeder key. To request one, email [info@1090mhz.uk](mailto:info@1090mhz.uk) with the following information:

* Your UUID:
* A station name (3-12 characters):
* Your antenna location (Latitude, Longitude) and height:

### Update `.env` file with Radar1090 UK Feeder Key

Inside your application directory (`/opt/adsb`), edit the `.env` file using your favorite text editor. Beginners may find the editor `nano` easy to use:

```shell
nano /opt/adsb/.env
```

This file holds all of the commonly used variables (such as our latitude, longitude and altitude). We're going to add our radar1090 Feeder Key to this file. Add the following line to the file:

```shell
RADAR1090_KEY=YOURFEEDERKEY
```

* Replace `YOURFEEDERKEY` with the key you received in response to your email.

For example:

```shell
RADAR1090_KEY=0x7A3DF151D95F3E9A
```

### Deploying feeder container

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file (inside the `services:` section).

```yaml
  radar1090:
    image: ghcr.io/sdr-enthusiasts/docker-radar1090:latest
    container_name: radar1090
    hostname: radar1090
    restart: always
    environment:
      - TZ=${FEEDER_TZ}
      - RADAR1090_KEY=${RADAR1090_KEY}
      - VERBOSE=false
      - BEASTHOST=ultrafeeder
    tmpfs:
      - /run:exec,size=256M
      - /tmp:size=128M
      - /var/log:size=32M
```

To explain what's going on in this addition:

* We're creating a container called `radar1090`, from the image `ghcr.io/sdr-enthusiasts/docker-radar1090`.
* We're passing several environment variables to the container:
  * `TZ` contains the timezone of your container host to sync time with the server that you added to `.env` previously
  * `RADAR1090_KEY` contains the key that you added to `.env` as per the instructions above
  * `BEASTHOST` indicates where to get the RAW data from
  * `RV_SERVER` is the address of the radar1090 server where your data will be sent. Please do not change this unless you're specifically instructed to
  * `VERBOSE` can be `TRUE` (meaning: show lots of information in the docker logs) or `FALSE` (show only errors in the docker logs)
* The mounted volumes make sure that the container will use the same timezone as your host system


## Refresh running containers

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `radar1090` container. You should see the following output:

```text
 ✔ Container ultrafeeder  Running
 ✔ Container piaware      Running
 ✔ Container fr24         Running
 ✔ Container adsbhub      Running
 ✔ Container radar1090    Started
```

We can view the logs for the environment with the command `docker logs radar1090`, or continually "tail" them with `docker logs -f radar1090`. The logs will be fairly unexciting and look like this:

```text
[2025-06-18 23:09:08.790][radar1090-log] starting as a service...
[2025-06-18 23:09:08.914][radar1090] INFO: Waiting for BEASTHOST (ultrafeeder) to come online
[2025-06-18 23:09:10.067][radar1090] INFO: BEASTHOST (ultrafeeder) is now online
[2025-06-18 23:09:10.076][radar1090] invoking: stdbuf -oL /usr/sbin/radar -k 0x7A3DF151D95F3E9A   -l ultrafeeder -f
[2025-06-18 23:14:25.839][radar1090-log] ------------------------
[2025-06-18 23:14:25.840][radar1090-log] Traffic statistics over the previous 300 seconds:
[2025-06-18 23:14:25.841][radar1090-log] - Total packets received from ultrafeeder: 20608
[2025-06-18 23:14:25.842][radar1090-log] - Duplicate packets discarded: 13960
[2025-06-18 23:14:25.843][radar1090-log] - Average bandwidth used (excluding overhead): 3435 bytes/sec
```

Once running, you can visit <https://www.1090mhz.uk/mystatus.php?key=YOURKEYHERE> (replace "YOURKEYHERE" with the key that you acquired in the previous step) to view the data you are feeding to radar1090. For example: <https://www.1090mhz.uk/mystatus.php?key=0x7A3DF151D95F3E9A>.

## Troubleshooting

Most log messages are self-explanatory and have suggestions on how to trouble-shoot your issue. Please contact [info@1090mhz.uk](mailto:info@1090mhz.uk) for more help.

## Advanced

If you want to look at more options and examples for the `radar1090` container, you can find the repository [here](https://github.com/sdr-enthusiasts/docker-radar1090)

## More information and support

* Please contact [info@1090mhz.uk](mailto:info@1090mhz.uk) for more help.
* You can always find help on the #adsb-containers channel on the [SDR Enthusiasts Discord server](https://discord.gg/m42azbZydy). This channel is meant for Noobs (beginners) and Experts alike.
