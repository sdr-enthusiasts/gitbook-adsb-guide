---
description: 'If you wish to feed AirNav RadarBox, follow the steps below.'
---

# Feeding RadarBox

[RadarBox](https://www.radarbox.com/) is a flight tracking company that displays aircraft & flight information in real-time on a map. RadarBox offers flight data such as latitude and longitude positions, origins and destinations, flight numbers, aircraft types, altitudes, headings and speeds. Based in Tampa, Florida, with a R&D center in Europe, RadarBox’s business operations include providing related data to aviation service providers worldwide.

`rbfeeder` is a RadarBox's client program to transmit ADS-B and Mode S data to RadarBox.

In exchange for your data, RadarBox will give you a Business Plan. If this is something of interest, you may wish to feed your data to them.

Personally, I really like their visualisation. Overlaying the flight data with precipitation and cloud cover looks fantastic.

The docker image [`mikenye/radarbox`](https://github.com/mikenye/docker-radarbox) contains `rbfeeder` and all of its required prerequisites and libraries. This needs to run in conjunction with `readsb` \(or another Beast provider\).

## Getting a Sharing Key

### Already running `rbfeeder`?

If you're not a first time user and are migrating from another installation, you can retrieve your sharing key using either of the following methods:

* SSH onto your existing receiver and run the command `rbfeeder --showkey --no-start`
* SSH onto your existing receiver and run the command `grep key= /etc/rbfeeder.ini`

### New to `rbfeeder`?

You'll need a _sharing key_. To get one, you can temporarily run the container, to allow it to communicate with the RadarBox servers generate a new sharing key.

Inside your application directory \(`/opt/adsb`\), run the following commands:

```text
docker pull mikenye/piaware:latest
source ./.env
timeout 60 docker run \
    --rm \
    -it \
    --network adsb_default \
    -e BEASTHOST=readsb \
    -e LAT=${FEEDER_LAT} \
    -e LONG=${FEEDER_LONG} \
    -e ALT=${FEEDER_ALT_M} \
    mikenye/radarbox
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

As you can see from the output above, the sharing key given to us from Radarbox is `g45643ab345af3c5d5g923a99ffc0de9`.  
  
You should now claim your receiver:

1. Go to [https://www.radarbox.com/](https://www.radarbox.com/)
2. Create an account or sign in
3. Claim your receiver by visiting [https://www.radarbox.com/raspberry-pi/claim](https://www.radarbox.com/raspberry-pi/claim) and following the instructions

## Update `.env` file with sharing key

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```bash
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our `rbfeeder` sharing key to this file. Add the following line to the file:

```text
RADARBOX_SHARING_KEY=YOURSHARINGKEY
```

* Replace `YOURSHARINGKEY` with the sharing key that was generated in the previous step.

For example:

```text
RADARBOX_SHARING_KEY=g45643ab345af3c5d5g923a99ffc0de9
```

## Deploying `rbfeeder`

### SegFault Fix

As the `rbfeeder` binary is designed to run on a Rasbperry Pi, the `rbfeeder` binary expects a file `/sys/class/thermal/thermal_zone0/temp` to be present, and contain the CPU temperature. If this file doesn't exist, the `rbfeeder` binary will crash and restart every few minutes. For more information, see [here](https://github.com/mikenye/docker-radarbox/issues/16#issuecomment-699627387).

The `mikenye/radarbox` container is multi-architecture, and accordingly you might not be running on a Raspberry Pi.

As a workaround, we can "fake" this file by performing the following additional steps:

#### Create "fake" directory structure

Inside your application directory \(assumed to be`/opt/adsb`\), run the following commands:

```bash
mkdir -p ./data/radarbox_segfault_fix/thermal_zone0
echo 24000 > ./data/radarbox_segfault_fix/thermal_zone0/temp
```

This creates a fake directory structure that contains a fake CPU temperature file that reads 24 degrees celcius.

#### Create docker volume

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Add the following lines below the `version:` section, and before the `services:` section:

```yaml

volumes:
  radarbox_segfault_fix:
    driver: local
    driver_opts:
      type: none
      device: /opt/adsb/data/radarbox_segfault_fix
      o: bind
      
```

This creates a volume containing our fake directory structure. We will map this through to the feeder container to prevent the SegFault from occurring.

### Create `rbfeeder` container

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  rbfeeder:
    image: mikenye/radarbox:latest
    tty: true
    container_name: radarbox
    restart: always
    depends_on:
      - readsb
    volumes:
      - "radarbox_segfault_fix:/sys/class/thermal:ro"
    environment:
      - BEASTHOST=readsb
      - LAT=${FEEDER_LAT}
      - LONG=${FEEDER_LONG}
      - ALT=${FEEDER_ALT_M}
      - TZ=${FEEDER_TZ}
      - SHARING_KEY=${RADARBOX_SHARING_KEY}
```

To explain what's going on in this addition:

* We're creating a container called `rbfeeder`, from the image `mikenye/rbfeeder:latest`.
* We're passing several environment variables to the container:
  * `BEASTHOST=readsb` to inform the feeder to get its ADSB data from the container `readsb` over our private `adsbnet` network.
  * `LAT` will use the `FEEDER_LAT` variable from your `.env` file.
  * `LONG` will use the `FEEDER_LONG` variable from your `.env` file.
  * `ALT` will use the `FEEDER_ALT_M` variable from your `.env` file.
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
  * `SHARING_KEY` will use the `RADARBOX_SHARING_KEY` variable from your `.env` file.

Once the file has been updated, issue the command `docker-compose up -d` in the application directory to apply the changes and bring up the `rbfeeder` container. You should see the following output:

```text
readsb is up-to-date
adsbx is up-to-date
piaware is up-to-date
fr24 is up-to-date
Creating rbfeeder
```

We can view the logs for the environment with the command `docker-compose logs`, or continually "tail" them with `docker-compose logs -f`. At this stage, the logs will be fairly unexciting and look like this:

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

