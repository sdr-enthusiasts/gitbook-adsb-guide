---
description: 'If you wish to feed AirNav RadarBox, follow the steps below.'
---

# Feeding RadarBox

[RadarBox](https://www.radarbox.com/) is a flight tracking company that displays aircraft & flight information in real-time on a map. RadarBox offers flight data such as latitude and longitude positions, origins and destinations, flight numbers, aircraft types, altitudes, headings and speeds. Based in Tampa, Florida, with a R&D center in Europe, RadarBox’s business operations include providing related data to aviation service providers worldwide.

`rbfeeder` is a RadarBox's client program to transmit ADS-B and Mode S data to RadarBox.

In exchange for your data, RadarBox will give you a Business Plan. If this is something of interest, you may wish to feed your data to them.

The docker image [`ghcr.io/sdr-enthusiasts/docker-radarbox`](https://github.com/sdr-enthusiasts/docker-radarbox) contains `rbfeeder` and all of its required prerequisites and libraries. This needs to run in conjunction with `ultrafeeder` \(or another Beast provider\).

## Getting a Sharing Key

### Already running `rbfeeder`?

If you're not a first time user and are migrating from another installation, you can retrieve your sharing key using either of the following methods:

* SSH onto your existing receiver and run the command `rbfeeder --showkey --no-start`
* SSH onto your existing receiver and run the command `grep key= /etc/rbfeeder.ini`

### New to `rbfeeder`?

You'll need a _sharing key_. To get one, you can temporarily run the container, to allow it to communicate with the RadarBox servers generate a new sharing key.

Inside your application directory \(`/opt/adsb`\), run the following commands:

```bash
docker pull ghcr.io/sdr-enthusiasts/docker-radarbox:latest
source ./.env
timeout 60 docker run \
    --rm \
    -it \
    --network adsb_default \
    -e BEASTHOST=ultrafeeder \
    -e LAT=${FEEDER_LAT} \
    -e LONG=${FEEDER_LONG} \
    -e ALT=${FEEDER_ALT_M} \
    ghcr.io/sdr-enthusiasts/docker-radarbox
```

The command will run the container for one minute, which should be ample time for the container to connect to RadarBox receive a sharing key.

For example:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-rbfeeder: executing...
WARNING: TZ environment variable not set

WARNING: SHARING_KEY environment variable was not set!
Please make sure you note down the key generated.
Pass the key as environment var SHARING_KEY on next launch!

[cont-init.d] 01-rbfeeder: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
[mlat-client] Delaying mlat-client startup until rbfeeder receives station sn...
[rbfeeder] [2020-11-20 08:55:03]  Starting RBFeeder Version 0.3.5 (build 20200727132301)
[rbfeeder] [2020-11-20 08:55:03]  Using configuration file: /etc/rbfeeder.ini
[rbfeeder] [2020-11-20 08:55:03]  Network-mode enabled.
[rbfeeder] [2020-11-20 08:55:03]                Remote host to fetch data: 172.30.0.12
[rbfeeder] [2020-11-20 08:55:03]                Remote port: 30005
[rbfeeder] [2020-11-20 08:55:03]                Remote protocol: BEAST
[rbfeeder] [2020-11-20 08:55:03]  System: raspberry
[rbfeeder] [2020-11-20 08:55:03]  Start date/time: 2020-11-20 08:55:03
[rbfeeder] [2020-11-20 08:55:03]  Socket for ANRB created. Waiting for connections on port 32088
[rbfeeder] [2020-11-20 08:55:04]  Connection established.
[rbfeeder] [2020-11-20 08:55:04]  Empty sharing key. We will try to create a new one for you!
[rbfeeder] [2020-11-20 08:55:05]  Your new key is g45643ab345af3c5d5g923a99ffc0de9. Please save this key for future use. You will have to know this key to link this receiver to your account in RadarBox24.com. This key is also saved in configuration file (/etc/rbfeeder.ini)
[mlat-client] Delaying mlat-client startup until rbfeeder receives station sn...
[rbfeeder] [2020-11-20 08:55:35]  Connection established.
[rbfeeder] [2020-11-20 08:55:36]  Connection with RadarBox24 server OK! Key accepted by server.
[cont-finish.d] executing container finish scripts...
[cont-finish.d] done.
[s6-finish] waiting for services.
[s6-finish] sending all processes the TERM signal.
[s6-finish] sending all processes the KILL signal and exiting.
```

In the output above, see the line:

```text
[rbfeeder] [2020-11-20 08:55:05]  Your new key is g45643ab345af3c5d5g923a99ffc0de9.
```

As you can see from the output above, the sharing key given to us from Radarbox is `g45643ab345af3c5d5g923a99ffc0de9`.

If the script doesn't output the sharing key, it can be found by using the following command:

```bash
docker exec -it rbfeeder /bin/sh -c "cat /etc/rbfeeder.ini" | grep key
```

Command output:

```text
key=g45643ab345af3c5d5g923a99ffc0de9
```

## Claiming Your Receiver

1. Go to [https://www.radarbox.com/](https://www.radarbox.com/)
2. Create an account or sign in
3. Claim your receiver by visiting [https://www.radarbox.com/raspberry-pi/claim](https://www.radarbox.com/raspberry-pi/claim) and following the instructions

## Update `.env` file with sharing key

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```bash
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our `rbfeeder` sharing key to this file. Add the following line to the file:

```bash
RADARBOX_SHARING_KEY=YOURSHARINGKEY
```

* Replace `YOURSHARINGKEY` with the sharing key that was generated in the previous step.

For example:

```bash
RADARBOX_SHARING_KEY=g45643ab345af3c5d5g923a99ffc0de9
```

## Deploying `rbfeeder`

### Create `rbfeeder` container

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  rbfeeder:
    image: ghcr.io/sdr-enthusiasts/docker-radarbox:latest
    tty: true
    container_name: rbfeeder
    restart: always
    environment:
      - BEASTHOST=ultrafeeder
      - LAT=${FEEDER_LAT}
      - LONG=${FEEDER_LONG}
      - ALT=${FEEDER_ALT_M}
      - TZ=${FEEDER_TZ}
      - SHARING_KEY=${RADARBOX_SHARING_KEY}
    tmpfs:
      - /run:exec,size=64M
      - /var/log
```

If you are in the USA and are also running the `dump978` container with a second SDR, add the following additional lines to the `environment:` section:

```yaml
      - UAT_RECEIVER_HOST=dump978
```

To explain what's going on in this addition:

* We're creating a container called `rbfeeder`, from the image `ghcr.io/sdr-enthusiasts/docker-radarbox:latest`.
* We're passing several environment variables to the container:
  * `BEASTHOST=ultrafeeder` to inform the feeder to get its ADSB data from the container `ultrafeeder` over our private `adsbnet` network.
  * `LAT` will use the `FEEDER_LAT` variable from your `.env` file.
  * `LONG` will use the `FEEDER_LONG` variable from your `.env` file.
  * `ALT` will use the `FEEDER_ALT_M` variable from your `.env` file.
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
  * `SHARING_KEY` will use the `RADARBOX_SHARING_KEY` variable from your `.env` file.
* For people running `dump978`:
  * `UAT_RECEIVER_HOST=dump978` specifies the host to pull UAT data from; in this instance our `dump978` container.
* We're using `tmpfs` for volumes that have regular I/O. Any files stored in a `tmpfs` mount are temporarily stored outside the container's writable layer. This helps to reduce:
  * The size of the container, by not writing changes to the underlying container; and
  * SD Card or SSD wear

## Update `ultrafeeder` container configuration

Before running `docker compose`, we also want to update the configuration of the `ultrafeeder` container, so that it generates MLAT data for piaware.

Open the `docker-compose.yml` and make the following environment value is part of the `ULTRAFEEDER_CONFIG` variable to the `ultrafeeder` service:

```yaml
      - ULTRAFEEDER_CONFIG=mlathub,rbfeeder,30105,beast_in;
```

To explain this addition, the `ultrafeeder` container will connect to the `rbfeeder` container on port `30105` and receive MLAT data. This data will then be included in any outbound data streams from `ultrafeeder`.

## Refresh running containers

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `rbfeeder` container. You should see the following output:

```text
ultrafeeder is up-to-date
piaware is up-to-date
fr24 is up-to-date
Creating rbfeeder
```

We can view the logs for the environment with the command `docker compose logs`, or continually "tail" them with `docker compose logs -f`. At this stage, the logs will be fairly unexciting and look like this:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-rbfeeder: executing...
[cont-init.d] 01-rbfeeder: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
[mlat-client] Delaying mlat-client startup until rbfeeder receives station sn...
[rbfeeder] [2020-11-20 17:04:26]  Starting RBFeeder Version 0.3.5 (build 20200727132301)
[rbfeeder] [2020-11-20 17:04:26]  Using configuration file: /etc/rbfeeder.ini
[rbfeeder] [2020-11-20 17:04:26]  Network-mode enabled.
[rbfeeder] [2020-11-20 17:04:26]                Remote host to fetch data: 192.168.69.35
[rbfeeder] [2020-11-20 17:04:26]                Remote port: 30005
[rbfeeder] [2020-11-20 17:04:26]                Remote protocol: BEAST
[rbfeeder] [2020-11-20 17:04:26]  System: raspberry
[rbfeeder] [2020-11-20 17:04:26]  Start date/time: 2020-11-20 17:04:26
[rbfeeder] [2020-11-20 17:04:26]  Socket for ANRB created. Waiting for connections on port 32088
[rbfeeder] [2020-11-20 17:04:27]  Connection established.
[rbfeeder] [2020-11-20 17:04:28]  Connection with RadarBox24 server OK! Key accepted by server.[mlat-client] Fri Nov 20 17:04:56 2020 mlat-client 0.2.11 starting up
[mlat-client] Fri Nov 20 17:04:56 2020 Listening for Beast-format results connection on port 30105
[mlat-client] Fri Nov 20 17:04:56 2020 Connected to multilateration server at mlat1.rb24.com:40900, handshaking
[mlat-client] Fri Nov 20 17:04:57 2020 Server says:
[mlat-client]
[mlat-client]         AirNAv Server
[mlat-client]
[mlat-client]         The multilateration server source code is available under
[mlat-client]         the terms of the Affero GPL (v3 or later). You may obtain
[mlat-client]         a copy of this server's source code at the following
[mlat-client]         location: https://github.com/mutability/mlat-server
[mlat-client]
[mlat-client] Fri Nov 20 17:04:57 2020 Handshake complete.
[mlat-client] Fri Nov 20 17:04:57 2020   Compression:       zlib2
[mlat-client] Fri Nov 20 17:04:57 2020   UDP transport:     disabled
[mlat-client] Fri Nov 20 17:04:57 2020   Split sync:        disabled
[mlat-client] Fri Nov 20 17:04:57 2020 Input connected to 192.168.69.35:30005
[mlat-client] Fri Nov 20 17:04:57 2020 Input format changed to BEAST, 12MHz clock
[mlat-client] Fri Nov 20 17:05:25 2020 Accepted Beast-format results connection from ::ffff:172.30.0.11:60196
```

We can see our container running with the command `docker ps`.

Once running, you can visit the RadarBox website, and go to "Account" &gt; "Stations" and click your station to see your live data.
