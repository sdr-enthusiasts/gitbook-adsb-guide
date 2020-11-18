---
description: 'If you wish to feed ADS-B Exchange, follow the steps below.'
---

# Feeding ADS-B Exchange

[ADSBexchange.com](https://adsbexchange.com/) is a co-op of ADS-B/Mode S/MLAT feeders from around the world, and the world’s largest source of unfiltered flight data.

## Generating a UUID for your receiver

If you are new to feeding ADS-B Exchange, you will need to generate a UUID for your feeder. If you already have a UUID from an existing feeder that you wish to re-use, you can skip this step.

In order to generate a feeder UUID, run the following command:

```shell
docker run --rm -it --entrypoint uuidgen mikenye/adsbexchange -t
```

Take note of the UUID returned.


## Deploying ADS-B Exchange feeder container

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file:

```yaml
  adsbx:
    image: mikenye/adsbexchange:latest
    tty: true
    container_name: adsbx
    restart: always
    environment:
      - BEASTHOST=readsb
      - LAT=YOURLATITUDE
      - LONG=YOURLONGITUDE
      - ALT=YOURALTITUDE
      - SITENAME=YOURSITENAME
      - UUID=YOURUUID
      - TZ=YOURTIMEZONE
    networks:
      - adsbnet
```

Be sure to change the following:

* Replace `YOURLATITUDE` with the latitude of your antenna (xx.xxxxx)
* Replace `YOURLONGITUDE` with the longitude of your antenna (xx.xxxxx)
* Replace `YOURALTITUDE` with the altitude above sea level of your antenna. If specified in feet, add the suffix `ft`. If specified in metres, add the suffix `m`. Note: negative altitudes MUST be in meters, with no suffix.
* Replace `YOURSITENAME` with a unique name for your receiver, using only the characters "A-Z", "a-z", (`-`) and (`_`)
* Replace `YOURUUID` with the UUID that was generated in the previous step
* Replace `YOURTIMEZONE` with your local timezone in "TZ database name" format (<https://en.wikipedia.org/wiki/List_of_tz_database_time_zones>).

So, assuming:

* Our latitude is -33.33333 and longitude is 111.11111
* Our altitude is 95m
* Our site name is `My_Cool_ADSB_Receiver`
* Our generated UUID is `4e8413e6-52eb-11ea-8681-1c1b0d925d3g`
* Our timezone is `Australia/Perth`

...then our `docker-compose.yml` file would be appended with the following:

```yaml
  adsbx:
    image: mikenye/adsbexchange:latest
    tty: true
    container_name: adsbx
    restart: always
    environment:
      - BEASTHOST=readsb
      - LAT=-33.33333
      - LONG=111.11111
      - ALT=95m
      - SITENAME=My_Cool_ADSB_Receiver
      - UUID=4e8413e6-52eb-11ea-8681-1c1b0d925d3g
      - TZ=Australia/Perth
    networks:
      - adsbnet
```

To explain what's going on in this addition:

* We're creating a container called `adsbx`, from the image `mikenye/adsbexchange:latest`.
* We're connecting the container to the docker network `adsbnet`.
* We're passing several environment variables to the container:
  * `BEASTHOST=readsb` to inform the feeder to get its ADSB data from the container `readsb` over our private `adsbnet` network.
  * `LAT=-33.33333` to inform the feeder of the antenna's latitude
  * `LONG=111.11111` to inform the feeder of the antenna's longitude
  * `ALT=95m` to inform the feeder of the antenna's altitude
  * `SITENAME=My_Cool_ADSB_Receiver` to inform the feeder of our site name
  * `UUID=4e8413e6-52eb-11ea-8681-1c1b0d925d3g` to inform the feeder of our UUID
  * `TZ=Australia/Perth` to inform the feeder of our local timezone.

Once the file is created, issue the command `docker-compose up -d` to bring up the ADSB environment. You should see the following output:

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

After a few minutes, point your browser at <https://adsbexchange.com/myip/>. You should see two green smiley faces indicating that you are successfully sending data.