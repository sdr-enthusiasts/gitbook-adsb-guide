---
description: 'If you wish to feed OpenSky Network, follow the steps below.'
---

# Feeding OpenSky Network

The [OpenSky Network](https://opensky-network.org/) is a non-profit association based in Switzerland. It aims at improving the security, reliability and efficiency of the air space usage by providing open access of real-world air traffic control data to the public.

The docker image [`ghcr.io/sdr-enthusiasts/docker-opensky-network`](https://github.com/sdr-enthusiasts/docker-opensky-network) contains the required feeder software and all required prerequisites and libraries. This needs to run in conjunction with `ultrafeeder` \(or another Beast provider\).

## Obtaining an OpenSky Network Feeder Serial Number

First-time users should obtain a feeder serial number.

Firstly, make sure you have registered for an account on the [OpenSky Network website](https://opensky-network.org/), and have your username on-hand.

In order to obtain a feeder serial number, we will start a temporary container running `opensky-feeder`, which will connect to OpenSky Network and be issued a serial number. The temporary container will automatically be stopped and deleted after 60 seconds.

To do this, run the command:

```text
timeout 60s docker run \
    --rm \
    -it \
    -e LAT=YOURLATITUDE \
    -e LONG=YOURLONGITUDE \
    -e ALT=YOURALTITUDE \
    -e BEASTHOST=ultrafeeder\
    -e OPENSKY_USERNAME=YOUROPENSKYUSERNAME \
    ghcr.io/sdr-enthusiasts/docker-opensky-network
```

Be sure to change the following:

* Replace `YOURLATITUDE` with the latitude of your antenna \(xx.xxxxx\)
* Replace `YOURLONGITUDE` with the longitude of your antenna \(xx.xxxxx\)
* Replace `YOURALTITUDE` with the altitude above sea level of your antenna _**in metres**_
* Replace `YOUROPENSKYUSERNAME` with your OpenSky Network username

Once the container has started, you should see output similar to the following:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-opensky-network: executing...

WARNING: OPENSKY_SERIAL environment variable was not set!
Please make sure you note down the serial generated.
Pass the key as environment var OPENSKY_SERIAL on next launch!

[cont-init.d] 01-opensky-network: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
[opensky-feeder] [INFO] [COMP] Initialize STAT
[opensky-feeder] [INFO] [COMP] Initialize POS
[opensky-feeder] [INFO] [COMP] Initialize DEVTYPE
[opensky-feeder] [INFO] [COMP] Initialize NET
[opensky-feeder] [INFO] [COMP] Initialize TB
[opensky-feeder] [INFO] [COMP] Initialize SERIAL
[opensky-feeder] [INFO] [COMP] Initialize BUF
[opensky-feeder] [INFO] [COMP] Initialize RELAY
[opensky-feeder] [INFO] [COMP] Initialize RC
[opensky-feeder] [INFO] [COMP] Initialize FILTER
[opensky-feeder] [INFO] [COMP] Initialize RECV
[opensky-feeder] [INFO] [COMP] Start STAT
[opensky-feeder] [INFO] [COMP] Start POS
[opensky-feeder] [INFO] [COMP] Start DEVTYPE
[opensky-feeder] [INFO] [COMP] Start NET
[opensky-feeder] [INFO] [COMP] Start TB
[opensky-feeder] [INFO] [COMP] Start SERIAL
[opensky-feeder] [INFO] [COMP] Start RELAY
[opensky-feeder] [INFO] [COMP] Start RC
[opensky-feeder] [INFO] [COMP] Start FILTER
[opensky-feeder] [INFO] [COMP] Start RECV
[opensky-feeder] [INFO] [INPUT] Trying to connect to '10.0.0.1': [10.0.0.1]:30005
[opensky-feeder] [INFO] [INPUT] connected to '10.0.0.1'
[opensky-feeder] [INFO] [NET] Trying to connect to 'collector.opensky-network.org': [194.209.200.6]:10004
[opensky-feeder] [INFO] [NET] connected to 'collector.opensky-network.org'
[opensky-feeder] [INFO] [LOGIN] Sending Device ID 5, Version 2.1.7
[opensky-feeder] [INFO] [SERIAL] Requesting new serial number
[opensky-feeder] [INFO] [SERIAL] Got a new serial number: -1408234269
[opensky-feeder] [INFO] [LOGIN] Sending Serial Number -1408234269
[opensky-feeder] [INFO] [GPS] Sending position -33.3333°, +111.1111°, +100.8m
[opensky-feeder] [INFO] [LOGIN] Sending Username 'johnnytightlips'
[cont-finish.d] executing container finish scripts...
[cont-finish.d] done.
[s6-finish] waiting for services.
[s6-finish] sending all processes the TERM signal.
[s6-finish] sending all processes the KILL signal and exiting.
```

As you can see from the output above, we've been allocated a serial number of `-1408234269`.

```text
[opensky-feeder] [INFO] [SERIAL] Got a new serial number: -1408234269
```

## Update `.env` file with OpenSky-Network details

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```text
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our OpenSky-Network username and serial to this file. Add the following lines to the file:

```text
OPENSKY_USERNAME=YOUROPENSKYUSERNAME
OPENSKY_SERIAL=YOUROPENSKYSERIAL
```

* Replace `YOUROPENSKYUSERNAME` with your OpenSky Network username
* Replace `YOUROPENSKYSERIAL` with your OpenSky Network serial

For example:

```text
OPENSKY_USERNAME=johnnytightlips
OPENSKY_SERIAL=-1408234269
```

Failure to specify the `OPENSKY_SERIAL` environment variable will cause a new feeder serial to be created every time the container is started. Please do the right thing and set `OPENSKY_SERIAL`!

## Deploying feeder container

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  opensky:
    image: ghcr.io/sdr-enthusiasts/docker-opensky-network:latest
    tty: true
    container_name: opensky
    restart: unless-stopped
    environment:
      - TZ=${FEEDER_TZ}
      - BEASTHOST=ultrafeeder
      - LAT=${FEEDER_LAT}
      - LONG=${FEEDER_LONG}
      - ALT=${FEEDER_ALT_M}
      - OPENSKY_USERNAME=${OPENSKY_USERNAME}
      - OPENSKY_SERIAL=${OPENSKY_SERIAL}
    tmpfs:
      - /run:exec,size=64M
      - /var/log
```

To explain what's going on in this addition:

* We're creating a container called `opensky`, from the image `ghcr.io/sdr-enthusiasts/docker-opensky-network:latest`.
* We're passing several environment variables to the container:
  * `BEASTHOST=ultrafeeder` to inform the feeder to get its ADSB data from the container `ultrafeeder`
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
  * `LAT` will use the `FEEDER_LAT` variable from your `.env` file.
  * `LONG` will use the `FEEDER_LONG` variable from your `.env` file.
  * `ALT` will use the `FEEDER_ALT_M` variable from your `.env` file \(as metres are required for this feeder\).
  * `OPENSKY_USERNAME` will use the `OPENSKY_USERNAME` variable from your `.env` file.
  * `OPENSKY_SERIAL` will use the `OPENSKY_SERIAL` variable from your `.env` file.
* We're using `tmpfs` for volumes that have regular I/O. Any files stored in a `tmpfs` mount are temporarily stored outside the container's writable layer. This helps to reduce:
  * The size of the container, by not writing changes to the underlying container; and
  * SD Card or SSD wear

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `adsbhub` container. You should see the following output:

```text
 ✔ Container ultrafeeder  Running
 ✔ Container piaware      Running
 ✔ Container fr24         Running
 ✔ Container adsbhub      Running
 ✔ Container opensky      Started
```

We can view the logs for the environment with the command `docker logs opensky`, or continually "tail" them with `docker logs -f opensky`. The logs will be fairly unexciting and look like this:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-opensky-network: executing...
[cont-init.d] 01-opensky-network: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
[opensky-feeder] [INFO] [COMP] Initialize STAT
[opensky-feeder] [INFO] [COMP] Initialize POS
[opensky-feeder] [INFO] [COMP] Initialize DEVTYPE
[opensky-feeder] [INFO] [COMP] Initialize NET
[opensky-feeder] [INFO] [COMP] Initialize TB
[opensky-feeder] [INFO] [COMP] Initialize SERIAL
[opensky-feeder] [INFO] [COMP] Initialize BUF
[opensky-feeder] [INFO] [COMP] Initialize RELAY
[opensky-feeder] [INFO] [COMP] Initialize RC
[opensky-feeder] [INFO] [COMP] Initialize FILTER
[opensky-feeder] [INFO] [COMP] Initialize RECV
[opensky-feeder] [INFO] [COMP] Start STAT
[opensky-feeder] [INFO] [COMP] Start POS
[opensky-feeder] [INFO] [COMP] Start DEVTYPE
[opensky-feeder] [INFO] [COMP] Start NET
[opensky-feeder] [INFO] [COMP] Start TB
[opensky-feeder] [INFO] [COMP] Start SERIAL
[opensky-feeder] [INFO] [COMP] Start RELAY
[opensky-feeder] [INFO] [COMP] Start RC
[opensky-feeder] [INFO] [COMP] Start FILTER
[opensky-feeder] [INFO] [COMP] Start RECV
[opensky-feeder] [INFO] [INPUT] Trying to connect to 'ultrafeeder': [172.30.0.6]:30005
[opensky-feeder] [INFO] [INPUT] connected to 'ultrafeeder'
[opensky-feeder] [INFO] [NET] Trying to connect to 'collector.opensky-network.org': [194.209.200.4]:10004
[opensky-feeder] [INFO] [NET] connected to 'collector.opensky-network.org'
[opensky-feeder] [INFO] [LOGIN] Sending Device ID 5, Version 2.1.7
[opensky-feeder] [INFO] [LOGIN] Sending Serial Number -1408234269
[opensky-feeder] [INFO] [GPS] Sending position -33.3333°, +111.1111°, +100.8m
[opensky-feeder] [INFO] [LOGIN] Sending Username 'johnnytightlips'
[opensky-feeder] [INFO] [TB] Setting sync filter: 0
[opensky-feeder] [INFO] [TB] Setting ext squitter only filter: 1
```

Once running, you can visit [https://opensky-network.org/receiver-profile](https://opensky-network.org/receiver-profile) to view the data you are feeding to OpenSky-Network.

## Advanced

If you want to look at more options and examples for the `opensky` container, you can find the repository [here](https://github.com/sdr-enthusiasts/docker-opensky-network)
