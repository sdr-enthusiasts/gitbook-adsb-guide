---
description: >-
  The "ultrafeeder" container is the heart of our "adsb" application. It receives
  1090MHz ADS-B ES signals from your SDR, and demodulates ADS-B messages,
  making them available for all other containers. It also provides a website with a map based on tar1090, station statistics (graphs1090), mlat-client, and an mlat-hub to aggregate mlat results.
---

# Deploy "ultrafeeder"

In your favorite text editor, create a file named `docker-compose.yml` in your application directory (`/opt/adsb`) if following along verbatim.

```bash
nano docker-compose.yml
```

```yaml
version: '3.8'

services:
  ultrafeeder:
  # ultrafeeder combines a number of functions:
  # - it retrieves and decodes 1090MHz Mode A/C/S data from the SDR(s) using Wiedehopf's branch of readsb
  # - it implements a `tar1090` based map on port 80 (mapped to port 8080 on the host)
  # - it includes graph1090 (system statistics website) on http://xxxxx/graphs1090
  # - it sends ADSB data directly (without the need of additional containers) to the
  #   "new" aggregators, and, if desired, also to AdsbExchange
  # - it includes mlat-client to send MLAT data to these aggregators
  # - it includes an MLAT Hub to consolidate MLAT results and make them available to the built-in map and other services

    image: ghcr.io/sdr-enthusiasts/docker-adsb-ultrafeeder
    tty: true
    container_name: ultrafeeder
    hostname: ultrafeeder
    restart: unless-stopped
    device_cgroup_rules:
      - 'c 189:* rwm'
    ports:
      - 8080:80               # to expose the web interface
    environment:
      # --------------------------------------------------
      # general parameters:
      - LOGLEVEL=error
      - TZ=${FEEDER_TZ}
      # --------------------------------------------------
      # SDR related parameters:
      - READSB_DEVICE_TYPE=rtlsdr
      - READSB_RTLSDR_DEVICE=${ADSB_SDR_SERIAL}
      - READSB_RTLSDR_PPM=${ADSB_SDR_PPM}
      #
      # --------------------------------------------------
      # readsb/decoder parameters:
      - READSB_LAT=${FEEDER_LAT}
      - READSB_LON=${FEEDER_LONG}
      - READSB_ALT=${FEEDER_ALT_M}m
      - READSB_GAIN=${ADSB_SDR_GAIN}
      - READSB_MODEAC=true
      - READSB_RX_LOCATION_ACCURACY=2
      - READSB_STATS_RANGE=true
      #
      # --------------------------------------------------
      # Sources and Aggregator connections:
      # Note - remove the ones you are not using / feeding
      #  Make sure that each line ends with a semicolon ";",  with the
      #  exception of the last line which shouldn't have a ";"
      - ULTRAFEEDER_CONFIG=
          adsb,feed.adsb.fi,30004,beast_reduce_plus_out;
          adsb,in.adsb.lol,30004,beast_reduce_plus_out;
          adsb,feed.adsb.one,64004,beast_reduce_plus_out;
          adsb,feed.planespotters.net,30004,beast_reduce_plus_out;
          adsb,feed.theairtraffic.com,30004,beast_reduce_plus_out;
          mlat,feed.adsb.fi,31090,39000;
          mlat,in.adsb.lol,31090,39001;
          mlat,feed.adsb.one,64006,39002;
          mlat,mlat.planespotters.net,31090,39003;
          mlat,feed.theairtraffic.com,31090,39004
      # --------------------------------------------------
      - UUID=${ULTRAFEEDER_UUID}
      - READSB_FORWARD_MLAT_SBS=true
      #
      # --------------------------------------------------
      # TAR1090 (Map Web Page) parameters:
      - UPDATE_TAR1090=true
      - TAR1090_DEFAULTCENTERLAT=${FEEDER_LAT}
      - TAR1090_DEFAULTCENTERLON=${FEEDER_LONG}
      - TAR1090_MESSAGERATEINTITLE=true
      - TAR1090_PAGETITLE=${FEEDER_NAME}
      - TAR1090_PLANECOUNTINTITLE=true
      - TAR1090_ENABLE_AC_DB=true
      - TAR1090_FLIGHTAWARELINKS=true
      - HEYWHATSTHAT_PANORAMA_ID=${FEEDER_HEYWHATSTHAT_ID}
      - HEYWHATSTHAT_ALTS=${FEEDER_HEYWHATSTHAT_ALTS}
      - TAR1090_SITESHOW=true
      - TAR1090_RANGE_OUTLINE_COLORED_BY_ALTITUDE=true
      - TAR1090_RANGE_OUTLINE_WIDTH=2.0
      - TAR1090_RANGERINGSDISTANCES=50,100,150,200
      - TAR1090_RANGERINGSCOLORS='#1A237E','#0D47A1','#42A5F5','#64B5F6'
      - TAR1090_USEROUTEAPI=true
      #
      # --------------------------------------------------
      # GRAPHS1090 (Decoder and System Status Web Page) parameters:
      - GRAPHS1090_DARKMODE=true
      #
      # --------------------------------------------------
    volumes:
      - /opt/adsb/ultrafeeder/globe_history:/var/globe_history
      - /opt/adsb/ultrafeeder/graphs1090:/var/lib/collectd
      - /proc/diskstats:/proc/diskstats:ro
      - /dev:/dev:ro
    tmpfs:
      - /run:exec,size=256M
      - /tmp:size=128M
      - /var/log:size=32M
```

In the file above, you will find several parameters that have values denoted as `${xxxx}`. These values are read from a file in the same directory named `.env`. You can find an example of such file that you can download and edit [here](https://github.com/sdr-enthusiasts/docker-install/blob/main/sample-docker-compose.yml). Alternatively, you can simply replace `${xxxx}` with the value you want to use, for example `READSB_RTLSDR_DEVICE=${ADSB_SDR_SERIAL}` --> `READSB_RTLSDR_DEVICE=0000001090`.

The `docker-compose.yml` file above will:

* Create a few mapped docker volumes to store historic message values and autogain values (`/var/globe_history`), statistics for the graphs (`/var/lib/collectd`), and make the disk statistics (`/proc/diskstats`) and USB devices (`/dev') available to the container.
* Create a service named `ultrafeeder` that will run the `ghcr.io/sdr-enthusiasts/docker-adsb-ultrafeeder` container.
  * We're mapping TCP port `8080` through to the container so we can access the web interface.
  * The variable `READSB_RTLSDR_DEVICE` tells `readsb` to look for an RTLSDR device with the serial of `1090` (that we re-serialized in an earlier step).
  * We're passing several environment variables through, including our timezone, latitude and longitude from the `.env` file \(denoted by `${VARIABLE}`\).
* We're using `tmpfs` for volumes that have regular I/O. Any files stored in a `tmpfs` mount are temporarily stored outside the container's writable layer. This helps to reduce:
  * The size of the container, by not writing changes to the underlying container; and
  * SD Card or SSD wear

## Using the MLAT results

See [https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder/blob/main/README.md#configuring-the-built-in-mlat-hub](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder/blob/main/README.md#configuring-the-built-in-mlat-hub) for more details on how to configure this.

## Deploying `ultrafeeder`

Once the `docker-compose.yml` file is created, issue the command `docker compose up -d` to bring up the environment.

```bash
docker compose up -d
```

You should see the following output:

```text
Creating network "adsb_default" with the default driver
Creating ultrafeeder         ... done
```

We can view the logs for the environment with the command `docker compose logs`, or continually "tail" them with `docker compose logs -f`. At this stage, the logs will be fairly extensive and unexciting and look like this:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 00-libsecomp2: executing...
[cont-init.d] 00-libsecomp2: exited 0.
[cont-init.d] 01-print-container-version: executing...
[2023-05-08 13:15:51.203][01-print-container-version] Container Version: 20230505-190743_9e4ed76_main, build date 2023-05-05 15:07:43 -0400
[cont-init.d] 01-print-container-version: exited 0.
... (more logs here)
[cont-init.d] done.
[services.d] starting services
[2023-05-08 13:15:53.482][mlat-client] Started as an s6 service
[services.d] done.
[2023-05-08 13:15:53.542][graphs1090] 646 (process ID) old priority 0, new priority 10
[2023-05-08 13:15:53.568][graphs1090-readback] copying DB from disk to /run/collectd
[2023-05-08 13:15:53.612][readsb] WARNING -- READSB_FORWARD_MLAT_SBS has been set! Do not feed the SBS (BaseStation) output of this container to any aggregators!
[2023-05-08 13:15:53.631][readsb] invoked by: /usr/local/bin/readsb --net --quiet --lat ...etc
```

We can see our container running with the command `docker ps`:

```text
CONTAINER ID   IMAGE                                                   COMMAND   CREATED        STATUS                  PORTS                                       NAMES
548becf06f0f   ghcr.io/sdr-enthusiasts/docker-adsb-ultrafeeder:latest            "/init"                  2 days ago     Up 2 days (healthy)                0.0.0.0:8080->80/tcp   ultrafeeder
```

We can see the `adsb_default` network with the command `docker network ls`:

```text
NETWORK ID     NAME           DRIVER    SCOPE
9950236691cc   adsb_default   bridge    local
2facb5a2ac76   bridge         bridge    local
0c73e1072dfc   host           host      local
74247d059bbb   none           null      local
```

## Ultrafeeder Web Pages

If configured and started using the example above, the container will make a website available at port 8080 of your host machine. Here are a few web pages that are generated (replace `my_host_ip` with the name or IP address of your host machine):

* `http://my_host_ip:8080/` : `tar1090` map and table of all aircraft received
* `http://my_host_ip:8080/graphs1090` : page with graphs and operations statistics of your station
* `http://my_host_ip:8080?pTracks` : showing all aircraft tracks received in the last 24 hours
* `http://my_host_ip:8080?heatmap&realheat` : showing a heatmap of all aircrafts of the last 24 hours
* `http://my_host_ip:8080?replay` : showing a timelapse replay of the past few days

## Viewing Live Data in Text Format

To see the data being received and decoded by our new container, run the command `docker exec -it ultrafeeder viewadsb`. This should display a real-time departure-lounge-style screen showing all the aircraft being tracked, for example:

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

You should also be able to point your web browser at `http://docker.host.ip.addr:8080/` to view the web interface \(change `docker.host.ip.addr` to the IP address of your docker host\). You should see a map showing your currently tracked aircraft, and a link to the "Performance Graphs".

## UUID security

The example files above use the same UUID for all feeders. Doing so makes it possible that your information from one site could be tracked on another. An alternative approach is to generate a unique UUID for each website, load those into a variable in .env, and append the UUID to the row in the `ULTRAFEEDER_CONFIG` section. For example:

```yaml
      - ULTRAFEEDER_CONFIG=
          adsb,feed.adsb.fi,30004,beast_reduce_plus_out,uuid=${ADSBFI_UUID};
          adsb,in.adsb.lol,30004,beast_reduce_plus_out,uuid=${ADSBLOL_UUID};
          adsb,feed.adsb.one,64004,beast_reduce_plus_out,uuid=${ADSBONE_UUID};
```


## Preparing and setting up `ultrafeeder` with Prometheus and Grafana

See [`readme-grafana.MD`](https://github.com/sdr-enthusiasts/docker-adsb-ultrafeeder/blob/main/README-grafana.md) at the container's Github repository web page.

## Minimalist setup

If you want to use `ultrafeeder` *only* as a SDR decoder but without any mapping or stats/graph websites, without MLAT connections or MLAT-hub, etc., for example to minimize CPU and RAM needs on a low CPU/memory single board computer, then do the following:

* in the `ULTRAFEEDER_CONFIG` parameter, remove any entry that starts with `mlat` or `mlathub`. This will prevent any `mlat-client`s or `mlathub` instances to be launched. If you want to connect the `mlat-client`(s) to external MLAT servers but you don't want to run the overhead of a MLATHUB, you can leave any entries starting with `mlat` in the `ULTRAFEEDER_CONFIG` parameter, and set `MLATHUB_DISABLE=true`
* Set the parameter `TAR1090_DISABLE=true`. This will prevent the `nginx` webserver and any websites to be launched
* Make sure not to use the `dhcr.io/sdr-enthusiasts/docker-adsb-ultrafeeder:telegraf` label as Telegraf adds a LOT of resource use to the container

## Troubleshooting

It is possible that you won't see any planes, either with the docker command above or when pointing your web browser at the readsb container. This can have a number of root causes - a common one being that active radio transmissions in other frequency bands that are reasonably "close" to the ADS-B band are completely overwhelming your SDR at the default starting gain of 49.6. It may be necessary to lower the starting point for the autogain script to at least allow the detection of some planes in order for the script to work. So if even after a few minutes you don't see any planes at all (and no ADS-B messages in the "Performance Graphs"), you may want to try to force a lower starting gain value into the autogain algorithm. To do this, please do the following. You may have to try different values instead of the value of `34` suggested here:

* Set the `READSB_AUTOGAIN_INITIAL_GAIN` variable in the `ultrafeeder` section of your docker-compose file:

```yaml
    environment:
      - READSB_AUTOGAIN_INITIAL_GAIN=34
```

* Then reset the autogain process with this command and recreate the container to apply the new `READSB_AUTOGAIN_INITIAL_GAIN` value :

```bash
cd /opt/adsb
docker exec -it ultrafeeder autogain reset
docker compose up -d
```
