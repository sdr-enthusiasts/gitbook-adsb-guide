---
description: 'If you wish to feed FlightAware, follow the steps below.'
---

# Feeding FlightAware \(piaware\)

[FlightAware](https://flightaware.com/) is a digital aviation company and operates the world's largest flight tracking and data platform.

`piaware` is a client program to securely transmit ADS-B and Mode S data to FlightAware.

In exchange for your data, FlightAware will give you an Enterprise Membership. If this is something of interest, you may wish to feed your data to them.

The docker image [`ghcr.io/sdr-enthusiasts/docker-piaware`](https://github.com/sdr-enthusiasts/docker-piaware) contains `piaware` and all of its required prerequisites and libraries. This can run standalone \(without the `readsb` container\), however for flexibility it is recommended to run with `readsb`, and this is the deployment method that will be used in this guide.

## Getting a Feeder ID

### Already running PiAware?

You'll need your _feeder-id_ from your existing feeder.

To get your _feeder-id_, log onto your feeder via SSH and issue the command:

```bash
piaware-config -show feeder-id
```

### New to PiAware?

If you're already running PiAware and you've followed the steps in the previous command, you can skip this section.

You'll need a _feeder-id_. To get one, you can temporarily run the container, to allow it to communicate with the FlightAware servers and get a new feeder ID.

Inside your application directory \(`/opt/adsb`\), run the following commands:

```text
docker pull ghcr.io/sdr-enthusiasts/docker-piaware:latest
source ./.env
timeout 60 docker run --rm -e LAT="$FEEDER_LAT" -e LONG="$FEEDER_LONG" ghcr.io/sdr-enthusiasts/docker-piaware:latest | grep "my feeder ID"
```

The command will run the container for 60 seconds, which should be ample time for the container to receive a feeder-id.

For example:

```text
$ timeout 60 docker run --rm LAT="$FEEDER_LAT" -e LONG="$FEEDER_LONG" ghcr.io/sdr-enthusiasts/docker-piaware:latest | grep "my feeder ID"
Set allow-mlat to yes in /etc/piaware.conf:1
Set allow-modeac to yes in /etc/piaware.conf:2
Set allow-auto-updates to no in /etc/piaware.conf:3
Set allow-manual-updates to no in /etc/piaware.conf:4
2020-03-06 06:16:11.860212500  [piaware] my feeder ID is acbf1f88-09a4-3a47-a4a0-10ae138d0c1g
write /dev/stdout: broken pipe
Terminated
```

As you can see from the output above, the feeder-id given to us from FlightAware is `acbf1f88-09a4-3a47-a4a0-10ae138d0c1g`.

You'll now want to "claim" this feeder.

To do this, go to: [https://flightaware.com/adsb/piaware/claim](https://flightaware.com/adsb/piaware/claim) and follow the instructions there.

## Update `.env` file with feeder-id

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```bash
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our `piaware` feeder-id to this file. Add the following line to the file:

```text
PIAWARE_FEEDER_ID=YOURFEEDERID
```

* Replace `YOURFEEDERID` with the feeder-id that was generated in the previous step.

For example:

```text
PIAWARE_FEEDER_ID=acbf1f88-09a4-3a47-a4a0-10ae138d0c1g
```

## Deploying piaware feeder

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  piaware:
    image: ghcr.io/sdr-enthusiasts/docker-piaware:latest
    tty: true
    container_name: piaware
    restart: always
    depends_on:
      - readsb
    ports:
      - 8081:8080
    environment:
      - BEASTHOST=readsb
      - LAT=${FEEDER_LAT}
      - LONG=${FEEDER_LONG}
      - TZ=${FEEDER_TZ}
      - FEEDER_ID=${PIAWARE_FEEDER_ID}
    tmpfs:
      - /run:exec,size=64M
      - /var/log
```

If you are in the USA and are also running the `dump978` container with a second SDR, add the following additional lines to the `environment:` section:

```yaml
      - UAT_RECEIVER_TYPE=relay
      - UAT_RECEIVER_HOST=dump978
```

To explain what's going on in this addition:

* We're creating a container called `piaware`, from the image `ghcr.io/sdr-enthusiasts/docker-piaware:latest`.
* We're passing several environment variables to the container:
  * `BEASTHOST=readsb` to inform the feeder to get its ADSB data from the container `readsb` over our private `adsbnet` network.
  * `LAT` will use the `FEEDER_LAT` variable from your `.env` file.
  * `LONG` will use the `FEEDER_LONG` variable from your `.env` file.
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
  * `FEEDER_ID` will use the `PIAWARE_FEEDER_ID` variable from your `.env` file.
* For people running `dump978`:
  * `UAT_RECEIVER_TYPE=relay` tells the container to pull UAT data from another host over the network.
  * `UAT_RECEIVER_HOST=dump978` specifies the host to pull UAT data from; in this instance our `dump978` container.
* We're using `tmpfs` for volumes that have regular I/O. Any files stored in a `tmpfs` mount are temporarily stored outside the container's writable layer. This helps to reduce:
  * The size of the container, by not writing changes to the underlying container; and
  * SD Card or SSD wear

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `piaware` container. You should see the following output:

```text
readsb is up-to-date
adsbx is up-to-date
Creating piaware
```

We can view the logs for the environment with the command `docker compose logs`, or continually "tail" them with `docker compose logs -f`. At this stage, the logs will be fairly unexciting and look like this:

```text
piaware           | [s6-init] making user provided files available at /var/run/s6/etc...exited 0.
piaware           | [s6-init] ensuring user provided files have correct perms...exited 0.
piaware           | [fix-attrs.d] applying ownership & permissions fixes...
piaware           | [fix-attrs.d] done.
piaware           | [cont-init.d] executing container initialization scripts...
piaware           | [cont-init.d] 01-piaware: executing...
piaware           | Set feeder-id to acbf1f88-09a4-3a47-a4a0-10ae138d0c1g in /etc/piaware.conf:1
piaware           | Set allow-auto-updates to no in /etc/piaware.conf:2
piaware           | Set allow-manual-updates to no in /etc/piaware.conf:3
piaware           | Set allow-mlat to yes in /etc/piaware.conf:4
piaware           | Set mlat-results to yes in /etc/piaware.conf:5
piaware           | Set receiver-type to relay in /etc/piaware.conf:6
piaware           | Set receiver-host to readsb in /etc/piaware.conf:7
piaware           | Set receiver-port to 30005 in /etc/piaware.conf:8
piaware           | [cont-init.d] 01-piaware: exited 0.
piaware           | [cont-init.d] done.
piaware           | [services.d] starting services
piaware           | [services.d] done.
piaware           | [skyaware] 2020/11/20 14:51:15 2020-11-20 14:51:15: (plugin.c.190) Cannot load plugin mod_setenv more than once, please fix your config (lighttpd may not accept such configs in future releases)
piaware           | [skyaware] 2020/11/20 14:51:15 2020-11-20 14:51:15: (server.c.1464) server started (lighttpd/1.4.53)
piaware           | [dump1090] 2020/11/20 14:51:15 Fri Nov 20 14:51:15 2020 AWST  dump1090-fa unknown starting up.
piaware           | [dump1090] 2020/11/20 14:51:15 Net-only mode, no SDR device or file open.
piaware           | [beast-splitter] 2020/11/20 14:51:15 127.0.0.1:30004: connected to 127.0.0.1:30004 with settings
piaware           | [beast-splitter] 2020/11/20 14:51:15 net(readsb:30005): connected to 172.30.0.12:30005
piaware           | [beast-splitter] 2020/11/20 14:51:15 net(readsb:30005): configured with settings: BCdfGijk
piaware           | [piaware] 2020/11/20 14:51:15 ****************************************************
piaware           | [piaware] 2020/11/20 14:51:15 piaware version 4.0 is running, process ID 329
piaware           | [piaware] 2020/11/20 14:51:15 your system info is: Linux 4e041cdad755 4.4.0-179-generic #209-Ubuntu SMP Fri Apr 24 17:48:44 UTC 2020 x86_64 GNU/Linux
piaware           | [piaware] 2020/11/20 14:51:17 Connecting to FlightAware adept server at piaware.flightaware.com/1200
piaware           | [piaware] 2020/11/20 14:51:17 Connection with adept server at piaware.flightaware.com/1200 established
piaware           | [piaware] 2020/11/20 14:51:18 TLS handshake with adept server at piaware.flightaware.com/1200 completed
piaware           | [piaware] 2020/11/20 14:51:18 FlightAware server certificate validated
piaware           | [piaware] 2020/11/20 14:51:18 encrypted session established with FlightAware
piaware           | [beast-splitter] 2020/11/20 14:51:18 net(readsb:30005): connected to a Beast-style receiver
piaware           | [piaware] 2020/11/20 14:51:18 ADS-B data program 'dump1090' is listening on port 30005, so far so good
piaware           | [piaware] 2020/11/20 14:51:18 Starting faup1090: /usr/lib/piaware/helpers/faup1090 --net-bo-ipaddr localhost --net-bo-port 30005 --stdout --lat -33.333 --lon 111.111
piaware           | [piaware] 2020/11/20 14:51:18 Started faup1090 (pid 354) to connect to dump1090
piaware           | [piaware] 2020/11/20 14:51:18 UAT support disabled by local configuration setting: uat-receiver-type
piaware           | [piaware] 2020/11/20 14:51:18 piaware received a message from dump1090!
piaware           | [piaware] 2020/11/20 14:51:18 adept reported location: -33.33333, 111.11111, 100ft AMSL
piaware           | [piaware] 2020/11/20 14:51:18 logged in to FlightAware as user mikenye
piaware           | [piaware] 2020/11/20 14:51:18 my feeder ID is acbf1f88-09a4-3a47-a4a0-10ae138d0c1g
piaware           | [piaware] 2020/11/20 14:51:18 site statistics URL: https://flightaware.com/adsb/stats/user/planetracker#stats-12345
piaware           | [piaware] 2020/11/20 14:51:18 multilateration data requested
piaware           | [piaware] 2020/11/20 14:51:19 Starting multilateration client: /usr/lib/piaware/helpers/fa-mlat-client --input-connect localhost:30005 --input-type auto --results beast,connect,localhost:30104 --results beast,listen,30105 --results ext_basestation,listen,30106 --udp-transport 70.42.6.232:12262:477609216
piaware           | [piaware] 2020/11/20 14:51:19 mlat-client(356): fa-mlat-client 0.2.11 starting up
piaware           | [piaware] 2020/11/20 14:51:19 mlat-client(356): Using UDP transport to 70.42.6.232 port 12262
piaware           | [piaware] 2020/11/20 14:51:19 mlat-client(356): Listening for Beast-format results connection on port 30105
piaware           | [piaware] 2020/11/20 14:51:19 mlat-client(356): Listening for Extended Basestation-format results connection on port 30106
piaware           | [piaware] 2020/11/20 14:51:19 mlat-client(356): Route MTU changed to 1500
piaware           | [piaware] 2020/11/20 14:51:19 mlat-client(356): Input connected to localhost:30005
piaware           | [piaware] 2020/11/20 14:51:19 mlat-client(356): Detected BEAST format input
piaware           | [piaware] 2020/11/20 14:51:19 mlat-client(356): Input format changed to BEAST, 12MHz clock
piaware           | [piaware] 2020/11/20 14:51:19 mlat-client(356): Beast-format results connection with 127.0.0.1:30104: connection established
piaware           | [piaware] 2020/11/20 14:51:19 piaware has successfully sent several msgs to FlightAware!
```

We can see our container running with the command `docker ps`.

Once running, you can visit `http://docker.host.ip.addr:8081/` to access PiAware's "SkyAware". From there you need to configure your location and altitude on the FlightAware's website. To do this, click on the blue button marked `Go to my ADS-B Statistics Page` on your "SkyAware". When the FA website loads, click on the gear icon near your feeder name and configure your location and height *using the same values you set in your .env file*. If you do not configure these values via the FA website MLAT will not work for your PiAware feeder.  You can also log onto FlightAware's website and click on the `My ADSB` link at the top of the page, and see your statistics, configure your location and altitude and other settings.

Remember, if you change your location and altitude on FlightAware's website, you'll need to update your `.env` file locally \(and re-run `docker compose up -d` from your application directory\)!

