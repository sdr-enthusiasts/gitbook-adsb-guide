# Deploy "readsb" Container

In your favourite text editor, create a file named `docker-compose.yml` in your project directory \(`/opt/adsb`\) if following along verbatim.

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
      - READSB_RTLSDR_DEVICE=00001090
      - READSB_GAIN=autogain
      - READSB_LAT=${FEEDER_LAT}
      - READSB_LON=${FEEDER_LONG}
      - READSB_RX_LOCATION_ACCURACY=2
      - READSB_STATS_RANGE=true
      - READSB_NET_ENABLE=true
    volumes:
      - readsbpb_rrd:/run/collectd
      - readsbpb_autogain:/run/autogain
```

The above will:

* Create two docker volumes - `readsbpb_rrd` and `readsb_autogain`, which are used to store the RRD files and autogain state files respectively.
* Create a service named `readsb` that will run the `mikenye/readsb-protobuf` container.
  * We're presenting the USB bus through to this container \(so `readsb` can talk to the USB-attached SDR\).
  * We're mapping TCP port `8080` through to the container so we can access the web interface.
  * We're passing several environment variables through, including our timezone, latitude and longitude from the `.env` file \(denoted by `${VARIABLE}`\).

Once this file is created, issue the command `docker-compose up -d` to bring up the environment. You should see the following output:

```text
Creating network "adsb_default" with the default driver
Creating readsb         ... done
```

We can view the logs for the environment with the command `docker-compose logs`, or continually "tail" them with `docker-compose logs -f`. At this stage, the logs will be fairly unexciting and look like this:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-timezone: executing...
[cont-init.d] 01-timezone: exited 0.
[cont-init.d] 02-sanity-check: executing...
[cont-init.d] 02-sanity-check: exited 0.
[cont-init.d] 03-initialise-gain: executing...
[cont-init.d] 03-initialise-gain: exited 0.
[cont-init.d] 04-telegraf: executing...
[cont-init.d] 04-telegraf: exited 0.
[cont-init.d] 05-rtlsdr-biastee: executing...
[cont-init.d] 05-rtlsdr-biastee: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
[readsb] 2020/11/09 13:23:47 Mon Nov  9 13:23:47 2020 AWST  Mictronics v4.0.1 starting up.
[autogain] 2020/11/09 13:23:47 Entering auto-gain stage: init
[collectd] 2020/11/09 13:23:47 [2020-11-09 13:23:47] plugin_load: plugin "logfile" successfully loaded.
[readsb] 2020/11/09 13:23:47 rtlsdr: using device #0: Generic RTL2832U (Realtek, RTL2832U, SN 00001090)
[lighttpd] 2020/11/09 13:23:47 2020-11-09 13:23:47: (server.c.1464) server started (lighttpd/1.4.53)
[readsb] 2020/11/09 13:23:47 Found Rafael Micro R820T tuner
[readsb] 2020/11/09 13:23:47 rtlsdr: tuner gain set to 49.6 dB
```

We can see our container running with the command `docker ps`:

```text
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS                    PORTS                    NAMES
f936a37dd488        mikenye/readsb-protobuf:latest      "/init"             4 hours ago         Up 2 hours (healthy)      0.0.0.0:8080->8080/tcp   readsb
```

We can see the `adsb_default` network with the command `docker network ls`:

```text
NETWORK ID          NAME                      DRIVER              SCOPE
2a4ef415c4d3        adsb_adsbnet              bridge              local
```

We can see the `adsb_readsbpb_*` volumes with the command `docker volume ls`:

```text
DRIVER                VOLUME NAME
local                 adsb_readsbpb_autogain
local                 adsb_readsbpb_rrd
```

### Viewing Live Data

To see the data being received and decoded by our new container, run the command `docker exec -it readsb viewadsb`. This should display a real-time departure-lounge-style screen showing all the aircraft being tracked, for example:

```text
 Hex    Mode  Sqwk  Flight   Alt    Spd  Hdg    Lat      Long   RSSI  Msgs  Ti -
────────────────────────────────────────────────────────────────────────────────
 7CF86F S     2061  BFRT22   10025  219  286  -31.871  116.586 -28.6    14  1
 7C79CA S     1200  YCC       2975  126  152  -32.490  115.887 -28.2    68  0
 7C79CB S     3000  YCD       1525  118  352  -32.221  115.948 -25.6   269  0
 7C79D1 S     1200  YCJ       3575  113  185  -32.375  115.837 -29.2   289  1
 7C79DB S     3000  YCT       1375  119  358  -32.176  115.940 -25.1   126  0
 7C79DC S     1200  YCU       3000   96  229  -32.437  115.929 -28.5   260  5
 7CF9E1 S     2055            1250  178  084                   -29.8    18  0
 7C822A S     3730  ZZW       1500                             -23.5   258  0
 7C7A3F S     1273  VOZ1485    grnd   0                        -25.4    11  3
 7C7A6E S     1200  YGW       2575   99  191  -32.296  115.813 -20.3   522  0
 7C1ABD S     4265  UTY6071  33125  398  197  -30.535  116.638 -23.6   363  0
 7C42D2 S     3664  NWK1663    grnd  59  239  -31.936  115.968 -21.6   258 12
 7C1B35 S                      grnd   9  281                   -28.3     4 15
 7C1B3C S     4306  VOZ9224  34000  405  192  -30.804  116.239 -22.5   150  0
 7C1C68 S     3646  FWA       5000  191  253  -31.803  116.299 -25.4   396  0
 7C6CA2 S     3760  NWK1885   7825  239  193  -31.609  116.244 -13.1   509  0
 7C6CA4 S     4035  NWK2873   3075  141  239  -31.846  116.143 -20.8   566  0
 7C4518 S     1464  QJE1928  13225  437  037  -31.840  116.316 -10.7   516  0
 7C0DAB S     3000  CZH        800   71  303  -32.085  115.923 -18.3   273  0
 7C6DB5 S                    36975                             -31.8    11 32
 7C3F19 S     4063  MQZ       1325  105  240  -31.898  116.043 -16.8   601  0
 7C7F72 S                                                      -31.5     7 37
 7C7796 S     4310  UTY734   34000  381  181  -30.327  116.639 -25.8    82  0
 7CF7C4 S           PHRX1A                                     -20.3    22  1
 7CF7C5 S           PHRX1B                                     -21.6    15  9
 7CF7C6 S           PHRX2A                                     -21.3    15  1
 7CF7C7 S           PHRX2B                                     -28.1     3  2
 7C2FD6 S     4223  NWK2878   4025  212  176  -32.006  115.948  -2.1   831  0
```

Press `CTRL-C` to escape this screen.

You should also be able to point your web browser at [http://docker.host.ip.addr:8080/](http://dockerhost:8080/) to view the web interface \(change `docker.host.ip.addr` to the IP address of your docker host\). You should see a map showing your currently tracked aircraft, and a link to the "Performance Graphs".

## Controlling our `adsb` application

If you need to bring the environment down \(for example, if you need to unplug the RTL-SDR USB dongle for maintenance\), you can issue the command `docker-compose down` from the application directory (the directory containing your `docker-compose.yml` file).

To start the environment, or apply any changes made to your `docker-compose.yml` file, you can issue the command `docker-compose up -d` from the application directory.

To "tail" the consolidated logs of all containers that make up the `adsb` application, you can issue the command `docker-compose logs -f` from the application directory. Individual container logs can be "tailed" with `docker logs -f CONTAINER`.

If you would like to manually update your containers to the latest versions of their images, you can run the following commands from the application directory:

```bash
docker-compose pull
docker-compose up -d
```

As we have defined our `readsb` container with `restart: always`, the containers will be automatically started if the host is rebooted.

## Healthcheck

Images can implement [healthchecks](https://docs.docker.com/engine/reference/builder/). A healthcheck is a script that docker runs within the container periodically thats tells docker whether the container is operating as expected.

In the `mikenye/readsb-protobuf` container, the [healthcheck script](https://github.com/mikenye/docker-readsb-protobuf/blob/main/rootfs/scripts/healthcheck.sh) does the following:

* For each expected network connection, make sure the connection exists
* Make sure that messages are being recieved from the SDR
* Make sure that the services running within the container aren't dying over and over for some reason

If all of the checks above pass, the container is considered healthy. If any fail, the container is considered unhealthy.

Earlier, we ran the command `docker ps`, to see our newly created `readsb` container up and running:

```text
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS                    PORTS                    NAMES
f936a37dd488        mikenye/readsb-protobuf:latest      "/init"             4 hours ago         Up 2 hours (healthy)      0.0.0.0:8080->8080/tcp   readsb
```

Notice that next to the container status, there is some information about the container's health. This may be one of the following:

* No health information: Not all images have healthchecks implemented. If an image doesn't report health, this is why.
* `(health: starting)`: The container will wait up to a predefined start-period (defined in the [Dockerfile](https://github.com/mikenye/docker-readsb-protobuf/blob/63a617e00d4c9bcd8a5d6a9d94cc2f2b32ac0489/Dockerfile#L286)) or until the healthcheck script returns a healthy result.
* `(healthy)`: The container is operating as expected.
* `(unhealthy)`: The container is not operating as expected.

Later in this guide we will implement a method to automatically restart any unhealthy containers. This way, if your SDR "wedges", the container will eventually go unhealthy and be restarted.

Where practical, I try to include healthchecks in all my images, so as you go through this guide and deploy more containers, you should see a health status whenever you issue the `docker ps` command.

### Why is my container healthy/unhealthy?

You can inspect the container for some (hopefully) meaningful output from the healthcheck script.

If you issue the command `docker inspect CONTAINER` (replacing `CONTAINER` with the name of the container you're interested in), you'll see lots of information about the container, including the output of the most recent run of the healthcheck script. This output is in JSON format, so if you have the `jq` utility installed, we can easily find our `readsb` container's most recent health information with the following command:

```bash
docker inspect readsb | jq .[0].State.Health.Log | jq .[-1]
```

Which will return something like this:

```json
{
  "Start": "2020-11-18T20:50:56.939989864+08:00",
  "End": "2020-11-18T20:50:57.210787023+08:00",
  "ExitCode": 0,
  "Output": "last_15min:local_accepted is 15891: HEALTHY\nabnormal death count for service autogain is 0: HEALTHY\nabnormal death count for service collectd is 0: HEALTHY\nabnormal death count for service graphs_1h-24h is 0: HEALTHY\nabnormal death count for service graphs_7d-1y is 0: HEALTHY\nabnormal death count for service lighttpd is 0: HEALTHY\nabnormal death count for service readsb is 0: HEALTHY\nabnormal death count for service readsbrrd is 0: HEALTHY\nabnormal death count for service telegraf_socat_vrs_json is 0: HEALTHY\nabnormal death count for service telegraf is 0: HEALTHY\n"
}
```

The `Start` and `End` sections show when the healthcheck script was started and finished.

An `ExitCode` of `0` represents a healthy result. An `ExitCode` of `1` represents an unhealthy result.

The `Output` section shows the output from the healthcheck script. For my images, I try to explain why the script came to its healthy/unhealthy conclusion in the output of the script. Note that newlines are printed as `\n`, so the output above is as follows:

```text
last_15min:local_accepted is 15891: HEALTHY
abnormal death count for service autogain is 0: HEALTHY
abnormal death count for service collectd is 0: HEALTHY
abnormal death count for service graphs_1h-24h is 0: HEALTHY
abnormal death count for service graphs_7d-1y is 0: HEALTHY
abnormal death count for service lighttpd is 0: HEALTHY
abnormal death count for service readsb is 0: HEALTHY
abnormal death count for service readsbrrd is 0: HEALTHY
abnormal death count for service telegraf_socat_vrs_json is 0: HEALTHY
abnormal death count for service telegraf is 0: HEALTHY
```

The first line is showing that we've received messages from the SDR in the past 15 minutes. The remaining lines show the number of times a service has had an "abnormal death" (crashed for some reason).

### Disabling Healthchecks

On systems with with low spec CPU/memory, you may wish to disable healthchecks to gain back some precious CPU cycles. There are ways to disable the docker healthchecks in the `docker-compose.yml` file. However, I publish versions of my images that have the healthcheck removed, so it may be easier to simply change the image's `latest` tag to `latest_nohealthcheck`.
