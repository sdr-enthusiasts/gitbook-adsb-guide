---
description: >-
  The "dump978" container is receives 978MHz UAT signals from your SDR, and
  demodulates ADS-B UAT messages, making them available for all other
  containers.
---

# Deploy "dump978" \(USA Only\)

## USA Only!

The FAA has adopted 1090MHz for all flight levels, and UAT only for operations below 18,000 feet. UAT supports two-way links, and the FAA provides additional services on the uplink including [TIS-B](https://www.faa.gov/nextgen/equipadsb/capabilities/ins_outs/#tisb), and [ADS-R](https://www.faa.gov/nextgen/equipadsb/capabilities/ins_outs/#adsr), as well as [FIS-B](https://www.faa.gov/nextgen/equipadsb/capabilities/ins_outs/#fisb), for weather and aeronautical information. Dual 1090/UAT systems have not been adopted in any other country.

**If you live outside of the USA \(or only have one SDR\), you can skip this section!**

## Deploying `dump978` container

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  dump978:
    image: mikenye/dump978:latest
    tty: true
    container_name: dump978
    restart: always
    devices:
      - /dev/bus/usb
    environment:
      - TZ=${FEEDER_TZ}
      - DUMP978_RTLSDR_DEVICE=978
    tmpfs:
      - /run/readsb
```

To explain what's going on in this addition:

* Create a service named `dump978` that will run the `mikenye/dump978` container.
  * We're presenting the USB bus through to this container \(so `dump978` can talk to the USB-attached SDR\).
  * We're passing several environment variables to the container:
    * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
    * `DUMP978_RTLSDR_DEVICE=978` tells `dump978` to use the RTL-SDR device with the serial `978`.

## Update `readsb` container configuration

Before running `docker-compose`, we also want to update the configuration of the `readsb` container, so that it pulls the demodulated UAT data from the `dump978` container.

Open the `docker-compose.yml` and add the following environment variable to the `readsb` service:

```yaml
      - READSB_NET_CONNECTOR=dump978,37981,raw_in
```

So, if your `readsb` service has not been modified from the previous step, it should look now look like this:

```yaml
version: '3.8'

volumes:
  readsbpb_rrd:
  readsbpb_autogain:

services:
  readsb:
    image: mikenye/readsb-protobuf:latest
    tty: true
    container_name: readsb
    hostname: readsb
    restart: always
    devices:
      - /dev/bus/usb
    ports:
      - 8080:8080
    environment:
      - TZ=${FEEDER_TZ}
      - READSB_DEVICE_TYPE=rtlsdr
      - READSB_RTLSDR_DEVICE=1090
      - READSB_GAIN=autogain
      - READSB_LAT=${FEEDER_LAT}
      - READSB_LON=${FEEDER_LONG}
      - READSB_RX_LOCATION_ACCURACY=2
      - READSB_STATS_RANGE=true
      - READSB_NET_ENABLE=true
      - READSB_NET_CONNECTOR=dump978,37981,raw_in
    volumes:
      - readsbpb_rrd:/run/collectd
      - readsbpb_autogain:/run/autogain
    tmpfs:
      - /run/readsb
```

To explain this addition, the `readsb` container will connect to the `dump978` container on port `37981` and receive UAT data.

The UAT data will be sent out over BEAST connections from the readsb container to the feeder containers.

## Refresh running containers

At this point, you can issue the command `docker-compose up -d` to refresh both the `readsb` and `dump978` containers.

## Viewing Live Data

Firstly, it should be noted that there is generally vastly less UAT traffic than ADS-B 1090MHz traffic, so don't immediately assume the `dump978` container isn't working if you can't immediately see UAT flights. Provided the container is running and healthy

To see the data being received and decoded by our new container, run the command `docker exec -it readsb viewadsb`. This should display a real-time departure-lounge-style screen showing all the aircraft being tracked. Look for entries in the `Mode` column listed as `blort`.

For example:

```text
 Hex    Mode  Sqwk  Flight   Alt    Spd  Hdg    Lat      Long   RSSI  Msgs  Ti -
────────────────────────────────────────────────────────────────────────────────
 ... other aircraft removed from output for brevity ...

 ... other aircraft removed from output for brevity ...
```

Press `CTRL-C` to escape this screen.

You should also be able to point your web browser at `http://docker.host.ip.addr:8080/` to view the web interface \(change `docker.host.ip.addr` to the IP address of your docker host\). You should see a map showing your currently tracked aircraft, and the UAT aircraft will be denoted by a different colour in the list.

## Feeder Configuration

The majority of feeders will happily accept a combined 1090MHz & 978MHz feed coming from `readsb`, so there should be nothing further to do.

The current exceptions are:

*  `piaware` - FlightAware has separate feeder binaries for 1090MHz and 978MHz.
*  `radarbox` - Radarbox needs some additional parameters to support both 1090MHz and 978MHz.

The additional configuration directives are discussed on each container's page.

