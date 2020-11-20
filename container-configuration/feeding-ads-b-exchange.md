---
description: 'If you wish to feed ADS-B Exchange, follow the steps below.'
---

# Feeding ADS-B Exchange

[ADSBExchange.com](https://adsbexchange.com/) is a co-op of ADS-B/Mode S/MLAT feeders from around the world, and the world’s largest source of unfiltered flight data.

## Generating a UUID for your receiver

If you are new to feeding ADS-B Exchange, you will need to generate a UUID for your feeder. If you already have a UUID from an existing feeder that you wish to re-use, you can skip this step.

In order to generate a feeder UUID, run the following command:

```bash
docker run --rm -it --entrypoint uuidgen mikenye/adsbexchange -t
```

Take note of the UUID returned.

## Update `.env` file with ADS-B Exchange Details

Inside your application directory (`/opt/adsb`), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```bash
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our ADS-B Exchange variables to this file. Add the following lines to the file:

```text
ADSBX_UUID=YOURUUID
ADSBX_SITENAME=YOURSITENAME
```

* Replace `YOURSITENAME` with a unique name for your receiver, using only the characters "A-Z", "a-z", (`-`) and (`_`), and ending in a random number.
* Replace `YOURUUID` with the UUID that was generated in the previous step.

For example:

```text
ADSBX_UUID=4e8413e6-52eb-11ea-8681-1c1b0d925d3g
ADSBX_SITENAME=johnsmith_123
```

## Deploying ADS-B Exchange feeder container

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file (inside the `services:` section):

```yaml
  adsbx:
    image: mikenye/adsbexchange:latest
    tty: true
    container_name: adsbx
    restart: always
    depends_on:
      - readsb
    environment:
      - BEASTHOST=readsb
      - LAT=${FEEDER_LAT}
      - LONG=${FEEDER_LONG}
      - ALT=${FEEDER_ALT_M}m
      - SITENAME=${ADSBX_SITENAME}
      - UUID=${ADSBX_UUID}
      - TZ=${FEEDER_TZ}
```

To explain what's going on in this addition:

* We're creating a container called `adsbx`, from the image `mikenye/adsbexchange:latest`.
* We're passing several environment variables to the container:
  * `BEASTHOST=readsb` to inform the feeder to get its ADSB data from the container `readsb` over our private `adsbnet` network.
  * `LAT` will use the `FEEDER_LAT` variable from your `.env` file.
  * `LONG` will use the `FEEDER_LONG` variable from your `.env` file.
  * `ALT` will use the `FEEDER_ALT_M` variable from your `.env` file.
  * `SITENAME` will use the `ADSBX_SITENAME` variable from your `.env` file.
  * `UUID` will use the `ADSBX_UUID` variable from your `.env` file.
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.

Once the file has been updated, issue the command `docker-compose up -d` in the application directory to apply the changes and bring up the `adsbx` container. You should see the following output:

```text
readsb is up-to-date
Creating adsbx
```

You can see from the output above that the `readsb` container was left alone (as the configuration for this container did not change), and a new container `adsbx` was created.

We can view the logs for the environment with the command `docker-compose logs`, or continually "tail" them with `docker-compose logs -f`. We should now see logs from our newly created `adsbx` container:

```text
adsbx   | [s6-init] making user provided files available at /var/run/s6/etc...exited 0.
adsbx   | [s6-init] ensuring user provided files have correct perms...exited 0.
adsbx   | [fix-attrs.d] applying ownership & permissions fixes...
adsbx   | [fix-attrs.d] done.
adsbx   | [cont-init.d] executing container initialization scripts...
adsbx   | [cont-init.d] 01-adsbexchange: executing...
adsbx   | Statistics will be available at: https://www.adsbexchange.com/api/feeders/?feed=4e8413e6-52eb-11ea-8681-1c1b0d925d3g
adsbx   | [cont-init.d] 01-adsbexchange: exited 0.
adsbx   | [cont-init.d] done.
adsbx   | [services.d] starting services
adsbx   | [services.d] done.
adsbx   | [adsbexchange-stats] Using UUID 4e8413e6-52eb-11ea-8681-1c1b0d925d3g for stats uploads...
adsbx   | [adsbexchange-stats] Using JSON directory /run/readsb for source data...
adsbx   | [adsbexchange-feed] Wed Feb 19 15:47:22 2020 AWST Mictronics v3.8.1 starting up.
adsbx   | [adsbexchange-feed] Net-only mode, no SDR device or file open.
adsbx   | [mlat-client] Wed Feb 19 15:47:22 2020 mlat-client 0.2.10 starting up
adsbx   | [adsbexchange-feed] Beast TCP input: Connection established: readsb (192.168.213.98) port 30005
adsbx   | [mlat-client] Wed Feb 19 15:47:23 2020 Connected to multilateration server at feed.adsbexchange.com:31090, handshaking
adsbx   | [adsbexchange-feed] BeastReduce TCP output: Connection established: feed.adsbexchange.com (167.114.60.74) port 30005
adsbx   | [mlat-client] Wed Feb 19 15:47:23 2020 Beast-format results connection with 192.168.213.98:30005: connection established
adsbx   | [mlat-client] Wed Feb 19 15:47:38 2020 Server says:
adsbx   | [mlat-client]
adsbx   | [mlat-client]     In-development v2 server. Expect odd behaviour.
adsbx   | [mlat-client]
adsbx   | [mlat-client]     The multilateration server source code is available under
adsbx   | [mlat-client]     the terms of the Affero GPL (v3 or later). You may obtain
adsbx   | [mlat-client]     a copy of this server’s source code at the following
adsbx   | [mlat-client]     location: https://github.com/adsbexchange/mlat-server
adsbx   | [mlat-client]
adsbx   | [mlat-client] Wed Feb 19 15:47:38 2020 Handshake complete.
adsbx   | [mlat-client] Wed Feb 19 15:47:38 2020  Compression:    zlib2
adsbx   | [mlat-client] Wed Feb 19 15:47:38 2020  UDP transport:   disabled
adsbx   | [mlat-client] Wed Feb 19 15:47:38 2020  Split sync:    disabled
adsbx   | [mlat-client] Wed Feb 19 15:47:38 2020 Input connected to readsb:30005
adsbx   | [mlat-client] Wed Feb 19 15:47:38 2020 Input format changed to BEAST, 12MHz clock
```

After a few minutes, point your browser at <https://adsbexchange.com/myip/>. You should see green smiley faces indicating that you are successfully sending data.
