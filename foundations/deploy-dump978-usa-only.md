---
description: >-
  The "dump978" container receives 978MHz UAT signals from your SDR (a different SDR from the one receiving 1090MHz signals), and
  demodulates ADS-B UAT messages, making them available for all other containers.
---

# Deploy "dump978" \(USA Only\)

## USA Only

The FAA has adopted 1090MHz for all flight levels, and UAT only for operations below 18,000 feet. UAT supports two-way links, and the FAA provides additional services on the uplink including [TIS-B](https://www.faa.gov/air_traffic/technology/equipadsb/capabilities/ins_outs/#tisb), and [ADS-R](https://www.faa.gov/air_traffic/technology/equipadsb/capabilities/ins_outs#adsr), as well as [FIS-B](https://www.faa.gov/air_traffic/technology/equipadsb/capabilities/ins_outs#fisb), for weather and aeronautical information. Dual 1090/UAT systems have not been adopted in any other country.

**If you live outside of the USA \(or only have one SDR\), you can skip this section!**

## Identify your UAT dongle's optimal PPM

Every RTL-SDR dongle will have a small frequency error as it is cheaply mass produced and not tested for accuracy. This frequency error is linear across the spectrum, and can be adjusted in most SDR programs by entering a PPM (parts per million) offset value. This  allows you to adjust the PPM figure using the ADSB_SDR_PPM environment variable.

Unplug all SDRs, leaving only the SDR to be used for 978MHz reception plugged in. Issue the following command:

`docker run --rm -it --entrypoint /scripts/estimate_rtlsdr_ppm.sh --device /dev/bus/usb ghcr.io/sdr-enthusiasts/docker-readsb-protobuf:latest`

This takes about 30 minutes and will print a numerical value for estimated optimum PPM setting.

## Update the .env file for UAT

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```bash
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add more variables associated our UAT dongle.

```text
UAT_SDR_SERIAL=978
UAT_SDR_GAIN=<your desired gain>
UAT_SDR_PPM=<your PPM from the step above>
```

* `UAR_SDR_SERIAL` is set to the serial number for your ADS-B dongle; the previous serialization steps set this to 978 by default but if you have used a different serial number enter it here
* `UAT_SDR_GAIN` is set to your desired dongle gain in dB, or `autogain` if you would like the software to determine the optimal gain
* `UAT_SDR_PPM` is set to your desired dongle PPM setting. Enter the number from the PPM estimation step earlier on this page.

For example:

```text
UAT_SDR_SERIAL=978
UAT_SDR_GAIN=autogain
UAT_SDR_PPM=1
```

## Deploying `dump978` container

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file \(under the `services:` section\):

```yaml
  dump978:
    image: ghcr.io/sdr-enthusiasts/docker-dump978:latest
    tty: true
    container_name: dump978
    restart: unless-stopped
    device_cgroup_rules:
      - 'c 189:* rwm'
    environment:
      - TZ=${FEEDER_TZ}
      - LAT=${FEEDER_LAT}
      - LON=${FEEDER_LONG}
      - DUMP978_RTLSDR_DEVICE=${UAT_SDR_SERIAL}
      - DUMP978_SDR_GAIN=${UAT_SDR_GAIN}
      - DUMP978_SDR_PPM=${UAT_SDR_PPM}
    volumes:
      - /opt/adsb/dump978:/var/globe_history
      - /dev:/dev:ro
    ports:
      - 30980:80
    tmpfs:
      - /run:exec,size=64M
      - /tmp:size=64M
      - /var/log:size=32M
```

To explain what's going on in this addition:

* Create a service named `dump978` that will run the `ghcr.io/sdr-enthusiasts/docker-dump978` container.
  * We're presenting the USB bus through to this container \(so `dump978` can talk to the USB-attached SDR\).
  * We're passing several environment variables to the container:
    * `TZ` will use the `FEEDER_TZ` variable from your `.env` file
    * `DUMP978_RTLSDR_DEVICE=${UAT_SDR_SERIAL}` tells `dump978` to use the RTL-SDR device with the serial from your `.env` file
    * The container will use the SDR gain and PPM values from your `.env` file (`${UAT_SDR_GAIN}` and `${UAT_SDR_PPM}`)

## Update `ultrafeeder` container configuration

Before running `docker compose`, we also want to update the configuration of the `ultrafeeder` container, so that it pulls the demodulated UAT data from the `dump978` container.  If you used the sample `docker-compose.yml` provided this has already been done.

Open the `docker-compose.yml` and add the following environment value to the `ULTRAFEEDER_CONFIG` variable under the `ultrafeeder` service:

```yaml
      - ULTRAFEEDER_CONFIG=adsb,dump978,30978,uat_in;
```

In addition, add these lines in the `GRAPHS1090` section of the `ultrafeeder` service:

```yaml
      # GRAPHS1090 (Decoder and System Status Web Page) parameters:
      - ENABLE_978=yes
      - URL_978=http://dump978/skyaware978
```

To explain this addition, the `ultrafeeder` container will connect to the `dump978` container on port `30978` and receive UAT data. This UAT data will then be included in any outbound data streams sent from `ultrafeeder`.

## Refresh running containers

At this point, you can issue the command `docker compose up -d` to refresh both the `ultrafeeder` and `dump978` containers.

## Viewing Live Data

Firstly, it should be noted that there is generally vastly less UAT traffic than ADS-B 1090MHz traffic, so don't immediately assume the `dump978` container isn't working if you can't immediately see UAT flights. Provided the container is running and healthy, to see the data being received and decoded by our new container, run the command `docker exec -it dump978 viewadsb`. This should display a real-time departure-lounge-style screen showing all the aircraft being tracked.

For example:

```text
Hex    Mode  Sqwk  Flight   Alt    Spd  Hdg    Lat      Long   RSSI  Msgs  Ti |
─-------------------------------------------------------------------------------
 A646B3 S                     3000   83  295   42.106  -71.352 -24.7    22  0
 ... other aircraft removed from output for brevity ...
```

Press `CTRL-C` to escape this screen.

You should also be able to point your web browser at `http://docker.host.ip.addr:30980/skyaware978` to view the web interface \(change `docker.host.ip.addr` to the IP address of your docker host\). You should see a map showing your currently tracked aircraft by the docker-dump978 container; these may include both aircraft received via UAT as well as TIS-B/ADS-R repeated transmissions that you receive at 978 MHz.

## Feeder Configuration

The majority of feeders will happily accept a combined 1090MHz & 978MHz feed coming from `ultrafeeder`, so there should be nothing further to do.

The current exceptions are:

* `piaware` - FlightAware has separate feeder binaries for 1090MHz and 978MHz.
* `radarbox` - Radarbox needs some additional parameters to support both 1090MHz and 978MHz.

The additional configuration directives are discussed on each container's page.

## Advanced

If you want to look at more options and examples for the `dump978` container, you can find the repository [here](https://github.com/sdr-enthusiasts/docker-dump978).
