---
description: 'If you wish to feed ADS-B Exchange, follow the steps below.'
---

# Feeding ADS-B Exchange

[ADSBExchange.com](https://adsbexchange.com/) collects data from over 10,000 receivers located around the world, aggregating the data onto a real-time web-based display for enthusiasts, and raw-data products for professional and commercial usage. It has recently been sold to a private firm called JETNET, for more information, please see:

* <https://www.rtl-sdr.com/ads-b-exchange-acquired-by-private-firm-jetnet/>
* <https://www.jetnet.com/news/jetnet-acquires-ads-b-exchange.html>

The docker image [`ghcr.io/sdr-enthusiasts/docker-adsbexchange`](https://github.com/sdr-enthusiasts/docker-adsbexchange) contains the ADS-B Exchange feeder software and all of its required prerequisites and libraries. This needs to run in conjunction with `ultrafeeder` \(or another Beast provider\).

## Generating a UUID

If you are new to feeding ADS-B Exchange, you will need to generate a UUID for your feeder. If you already have a UUID from an existing feeder that you wish to re-use, you can skip this step.

In order to generate a feeder UUID, run the following command:

```bash
cat /proc/sys/kernel/random/uuid
```

Take note of the UUID returned.

## Update `.env` file

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```bash
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our ADS-B Exchange variables to this file. Add the following lines to the file:

```text
ADSBX_UUID=YOURUUID
ADSBX_SITENAME=YOURSITENAME
```

* Replace `YOURSITENAME` with a unique name for your receiver, using only the characters "A-Z", "a-z", \(`-`\) and \(`_`\), and ending in a random number.
* Replace `YOURUUID` with the UUID that was generated in the previous step.

For example:

```text
ADSBX_UUID=4e8413e6-52eb-11ea-8681-1c1b0d925d3g
ADSBX_SITENAME=johnsmith_123
```

## Deploying ADS-B Exchange feeder

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  adsbx:
    image: ghcr.io/sdr-enthusiasts/docker-adsbexchange:latest
    tty: true
    container_name: adsbx
    restart: always
    environment:
      - BEASTHOST=ultrafeeder
      - LAT=${FEEDER_LAT}
      - LONG=${FEEDER_LONG}
      - ALT=${FEEDER_ALT_M}m
      - SITENAME=${ADSBX_SITENAME}
      - UUID=${ADSBX_UUID}
      - TZ=${FEEDER_TZ}
    tmpfs:
      - /run:exec,size=64M,uid=1000,gid=1000
      - /var/log
```

To explain what's going on in this addition:

* We're creating a container called `adsbx`, from the image `ghcr.io/sdr-enthusiasts/docker-adsbexchange:latest`.
* We're passing several environment variables to the container:
  * `BEASTHOST=ultrafeeder` to inform the feeder to get its ADSB data from the container `ultrafeeder` over our private `adsbnet` network.
  * `LAT` will use the `FEEDER_LAT` variable from your `.env` file.
  * `LONG` will use the `FEEDER_LONG` variable from your `.env` file.
  * `ALT` will use the `FEEDER_ALT_M` variable from your `.env` file.
  * `SITENAME` will use the `ADSBX_SITENAME` variable from your `.env` file.
  * `UUID` will use the `ADSBX_UUID` variable from your `.env` file.
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
* We're using `tmpfs` for volumes that have regular I/O. Any files stored in a `tmpfs` mount are temporarily stored outside the container's writable layer. This helps to reduce:
  * The size of the container, by not writing changes to the underlying container; and
  * SD Card or SSD wear

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `adsbx` container. You should see the following output:

```text
ultrafeeder is up-to-date
Creating adsbx
```

You can see from the output above that the `ultrafeeder` container was left alone \(as the configuration for this container did not change\), and a new container `adsbx` was created.

We can view the logs for the environment with the command `docker compose logs`, or continually "tail" them with `docker compose logs -f`. We should now see logs from our newly created `adsbx` container:

```text
adsbx             | [s6-init] making user provided files available at /var/run/s6/etc...exited 0.
adsbx             | [s6-init] ensuring user provided files have correct perms...exited 0.
adsbx             | [fix-attrs.d] applying ownership & permissions fixes...
adsbx             | [fix-attrs.d] done.
adsbx             | [cont-init.d] executing container initialization scripts...
adsbx             | [cont-init.d] 01-adsbexchange: executing...
adsbx             | Statistics available at: https://www.adsbexchange.com/api/feeders/?feed=4e8413e6-52eb-11ea-8681-1c1b0d925d3g
adsbx             | [cont-init.d] 01-adsbexchange: exited 0.
adsbx             | [cont-init.d] done.
adsbx             | [services.d] starting services
adsbx             | [services.d] done.
adsbx             | [adsbexchange-feed] Fri Nov 20 14:48:27 2020 AWST  readsb starting up.
adsbx             | [adsbexchange-feed] readsb version: wiedehopf git: 9f533868 (cpr focus json_reliable fast track, Sun Oct 18 18:26:34 2020 0200)
adsbx             | [adsbexchange-feed] struct sizes: 1552, 24, 64, 100
adsbx             | [adsbexchange-stats] Found local resolver in resolv.conf, disabling DNS Cache
adsbx             | [adsbexchange-stats] Using UUID [4e8413e6-52eb-11ea-8681-1c1b0d925d3g] for stats uploads
adsbx             | [adsbexchange-stats] Using JSON directory [/run/adsbexchange-feed] for source data
adsbx             | [adsbexchange-stats] NOT using script's DNS cache
adsbx             | [mlat-client] mlat-client 0.3.1 starting up
adsbx             | [mlat-client] Listening for Beast-format results connection on port 30105
adsbx             | [mlat-client] Connecting to feed.adsbexchange.com:31090
adsbx             | [adsbexchange-feed] Beast TCP input: Connection established: readsb (172.30.0.12) port 30005
adsbx             | [mlat-client] Connected to multilateration server at feed.adsbexchange.com:31090, handshaking
adsbx             | [adsbexchange-feed] BeastReduce TCP output: Connection established: feed.adsbexchange.com (216.48.109.64) port 30004
adsbx             | [adsbexchange-feed] UUID: 4e8413e6-52eb-11ea-8681-1c1b0d925d3g
adsbx             | [mlat-client] Beast-format results connection with 172.30.0.12:30005: connection established
adsbx             | [mlat-client] Server says:
adsbx             | [mlat-client]
adsbx             | [mlat-client]         In-development v2 server. Expect odd behaviour.
adsbx             | [mlat-client]
adsbx             | [mlat-client]         The multilateration server source code is available under
adsbx             | [mlat-client]         the terms of the Affero GPL (v3 or later). You may obtain
adsbx             | [mlat-client]         a copy of this server's source code at the following
adsbx             | [mlat-client]         location: https://github.com/adsbexchange/mlat-server
adsbx             | [mlat-client]
adsbx             | [mlat-client] Handshake complete.
adsbx             | [mlat-client]   Compression:       zlib2
adsbx             | [mlat-client]   UDP transport:     disabled
adsbx             | [mlat-client]   Split sync:        disabled
adsbx             | [mlat-client] Connecting to readsb:30005
adsbx             | [mlat-client] Input connected to readsb:30005
adsbx             | [mlat-client] Input format changed to BEAST, 12MHz clock
adsbx             | [mlat-client] Accepted Beast-format results connection from ::ffff:172.30.0.11:53198
```

After a few minutes, point your browser at [https://adsbexchange.com/myip/](https://adsbexchange.com/myip/). You should see green smiley faces indicating that you are successfully sending data.
Also check if your MLAT is synchronized: <https://map.adsbexchange.com/mlat-map/>
