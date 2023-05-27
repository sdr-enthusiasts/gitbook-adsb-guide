---
description: 'If you wish to feed Plane.watch, follow the steps below.'
---

# Feeding plane.watch

Plane.watch is the end product of a ***Wouldn't it be cool if?*** statement. It is an ADS-B aggregation project by members of SDR-Enthusiasts. It is currently completely non-commercial, run by members of the community as a passion project, and backed by an Australian-based Not-for-Profit association.

The docker image [`ghcr.io/plane-watch/docker-plane-watch`](https://github.com/plane-watch/docker-plane-watch) contains the plane.watch feeder software and all of its required prerequisites and libraries. This needs to run in conjunction with `ultrafeeder` \(or another Beast provider\).

## plane.watch Feeder Registration

### Register for a plane.watch feeder account

Head over to <https://atc.plane.watch> and sign up for an account.

### Create your feeder

Login to <https://atc.plane.watch>, click on **Resources**, **Feeders**, **+ New Feeder**. Fill out your details.

* Your **feed direction** should be set to **push**.
* Your **feed protocol** should be set to **beast**.

When you save your feeder, an **API Key** will be generated. Take note of this, as it will be required below.

## Update `.env` file

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```bash
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our plane.watch variables to this file. Add the following lines to the file:

```text
PW_API_KEY=YOURAPIKEY
```

* Replace `YOURAPIKEY` with the API KEY that was provided the previous step.

For example:

```text
PW_API_KEY=4e8413e6-52eb-11ea-8681-1c1b0d925d3g
```

## Deploying plane.watch feeder

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  planewatch:
    image: ghcr.io/plane-watch/docker-plane-watch:latest
    tty: true
    container_name: planewatch
    restart: always
    environment:
      - BEASTHOST=ultrafeeder
      - LAT=${FEEDER_LAT}
      - LONG=${FEEDER_LONG}
      - ALT=${FEEDER_ALT_M}m
      - TZ=${FEEDER_TZ}
      - API_KEY=${PW_API_KEY}
    tmpfs:
      - /run:exec,size=64M
      - /var/log
```

To explain what's going on in this addition:

* We're creating a container called `planewatch`, from the image `ghcr.io/plane-watch/docker-plane-watch:latest`.
* We're passing several environment variables to the container:
  * `BEASTHOST=ultrafeeder` to inform the feeder to get its ADSB data from the container `ultrafeeder` over our private `adsbnet` network.
  * `LAT` will use the `FEEDER_LAT` variable from your `.env` file.
  * `LONG` will use the `FEEDER_LONG` variable from your `.env` file.
  * `ALT` will use the `FEEDER_ALT_M` variable from your `.env` file.
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
  * `API_KEY` will use the `PW_API_KEY` variable from your `.env` file.
* We're using `tmpfs` for volumes that have regular I/O. Any files stored in a `tmpfs` mount are temporarily stored outside the container's writable layer. This helps to reduce:
  * The size of the container, by not writing changes to the underlying container; and
  * SD Card or SSD wear

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `planewatch` container. You should see the following output:

```text
ultrafeeder is up-to-date
Creating planewatch
```

You can see from the output above that the `ultrafeeder` container was left alone \(as the configuration for this container did not change\), and a new container `planewatch` was created.

We can view the logs for the environment with the command `docker compose logs`, or continually "tail" them with `docker compose logs -f`. We should now see logs from our newly created `planewatch` container:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-timezone: executing...
[cont-init.d] 01-timezone: exited 0.
[cont-init.d] 02-sanity_check: executing...
[cont-init.d] 02-sanity_check: exited 0.
[cont-init.d] done.
[services.d] starting services
2023-05-27T09:09:49+08:00 INF plane.watch feeder started version=20230526
2023-05-27T09:09:49+08:00 INF listening for incoming connections dst=feed.push.plane.watch:12346 listen=127.0.0.1:12346 proto=MLAT
2023-05-27T09:09:49+08:00 INF starting tunnel dst=feed.push.plane.watch:12345 proto=BEAST src=readsb:30005
[services.d] done.
2023-05-27T09:09:49+08:00 INF connection established dst=feed.push.plane.watch:12345 proto=BEAST src=readsb:30005
[mlat-client] Sat May 27 09:09:54 2023 mlat-client 0.2.11 starting up
[mlat-client] Sat May 27 09:09:54 2023 Listening for Beast-format results connection on port 30105
2023-05-27T09:09:54+08:00 INF connection established dst=feed.push.plane.watch:12346 listen=127.0.0.1:12346 proto=MLAT src=127.0.0.1:35492
[mlat-client] Sat May 27 09:09:54 2023 Connected to multilateration server at 127.0.0.1:12346, handshaking
[mlat-client] Sat May 27 09:09:59 2023 Server says:
[mlat-client] Sat May 27 09:09:59 2023 Handshake complete.
[mlat-client] Sat May 27 09:09:59 2023   Compression:       zlib2
[mlat-client] Sat May 27 09:09:59 2023   UDP transport:     disabled
[mlat-client] Sat May 27 09:09:59 2023   Split sync:        disabled
[mlat-client] Sat May 27 09:09:59 2023 Input connected to readsb:30005
[mlat-client] Sat May 27 09:09:59 2023 Input format changed to BEAST, 12MHz clock
2023-05-27T09:14:49+08:00 INF statistics bytesRxLocal=247548 bytesRxRemote=0 bytesTxLocal=0 bytesTxRemote=247548 proto=BEAST
2023-05-27T09:14:49+08:00 INF statistics bytesRxLocal=38354 bytesRxRemote=785 bytesTxLocal=785 bytesTxRemote=38354 proto=MLAT
```

After a few minutes, browse to [https://atc.plane.watch/](https://atc.plane.watch/). Your feeder should be listed as "online". It can take up to 10 minutes for the status to update.
