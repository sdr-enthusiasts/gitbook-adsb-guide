---
description: 'If you wish to feed ADSBHub, follow the steps below.'
---

# Feeding ADSBHub

The main goal of [ADSBHub](https://adsbhub.org/) is to become a ADS-B data sharing centre and valuable data source for all enthusiasts and professionals interested in development of ADS-B related software.

The docker image [`ghcr.io/sdr-enthusiasts/docker-adsbhub`](https://github.com/sdr-enthusiasts/docker-adsbhub) contains the required feeder software and all required prerequisites and libraries. This needs to run in conjunction with `ultrafeeder` \(or another Beast provider\).

## Getting a Station Key

### Obtaining an ADSBHub Station Key

First-time users should obtain a ADSBHub Station dynamic IP key. Follow the directions for steps 1 and 2 at [ADSBHub how to feed](https://www.adsbhub.org/howtofeed.php), ensuring your station is set up as a client and the data protocol set as "SBS" (see below.)

Existing users should sign in to their ADSBHub account, go to their "Settings" page, click on their station \(in the bar at the top of the settings table\) and retrieve their station key.

### Setting up Your Station

In your station preferences, you should set the following:

* Feeder type: `Linux`
* Data Protocol: `SBS`
* Station mode: `Client`

## Update `.env` file with ADSBHub Station Key

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```text
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our ADSBHub Station Key to this file. Add the following line to the file:

```text
ADSBHUB_STATION_KEY='YOURSTATIONKEY'
```

* Replace `YOURSTATIONKEY` with the station key you retrieved earlier.
* The single quotes \(`'`\) are important, as the station key from ADSBHub contains special characters that would confuse `docker compose` if the single quotes were missing.

For example:

```text
ADSBHUB_STATION_KEY='vrMr@AZn660X0H^0Usn~rcj$UJA7VlR.vEu4c;uh7mfU-J9ZUBXpJiUuWj37DTa5BtL'
```

## Deploying feeder container

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  adsbhub:
    image: ghcr.io/sdr-enthusiasts/docker-adsbhub:latest
    tty: true
    container_name: adsbhub
    restart: unless-stopped
    environment:
      - TZ=${FEEDER_TZ}
      - SBSHOST=ultrafeeder
      - CLIENTKEY=${ADSBHUB_STATION_KEY}
```

To explain what's going on in this addition:

* We're creating a container called `adsbhub`, from the image `ghcr.io/sdr-enthusiasts/docker-adsbhub/adsbhub:latest`.
* We're passing several environment variables to the container:
  * `SBSHOST=ultrafeeder` to inform the feeder to get its ADSB data from the container `ultrafeeder`
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
  * `CLIENTKEY` will use the `ADSBHUB_STATION_KEY` variable from your `.env` file.

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `adsbhub` container. You should see the following output:

```text
 ✔ Container ultrafeeder  Running
 ✔ Container piaware      Running
 ✔ Container fr24         Running
 ✔ Container adsbhub      Started
```

We can view the logs for the environment with the command `docker logs adsbhub`, or continually "tail" them with `docker logs -f adsbhub`. The logs will be fairly unexciting and look like this:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] 01-adsbhubclient: applying...
[fix-attrs.d] 01-adsbhubclient: exited 0.
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-adsbhubclient: executing...
[cont-init.d] 01-adsbhubclient: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
not connected
d3aae607bf68183e4a39be14fa4117144
connected
connected
connected
```

Once running, you can visit [https://www.adsbhub.org/statistic.php](https://www.adsbhub.org/statistic.php) to view the data you are feeding to ADSBHub.

## Advanced

If you want to look at more options and examples for the `adsbhub` container, you can find the repository [here](https://github.com/sdr-enthusiasts/docker-adsbhub)
