---
description: >-
  If you wish to deploy tar1090 for improved visualisation, follow the steps
  below.
---

# Improved Visualisation with tar1090

[`tar1090`](https://github.com/wiedehopf/tar1090) is a visualisation tool by [wiedehopf](https://github.com/wiedehopf) that provides some additional functionality over and above the `readsb` web interface.

This is my personal preference for displaying real-time ADS-B information.

## Create docker volumes

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Add the following lines to the  `volumes:` section at the top of the file \(below the `version:` section, and before the `services:` section\):

```yaml
  tar1090_heatmap:
```

This creates the volumes that will contain `tar1090`’s application data.

## Deploying `tar1090` container

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file:

```yaml
  tar1090:
    image: mikenye/tar1090:latest
    tty: true
    container_name: tar1090
    restart: always
    environment:
      - UPDATE_TAR1090=false
      - TZ=${FEEDER_TZ}
      - BEASTHOST=readsb
      - LAT=${FEEDER_LAT}
      - LONG=${FEEDER_LONG}
      - TAR1090_DEFAULTCENTERLAT=${FEEDER_LAT}
      - TAR1090_DEFAULTCENTERLON=${FEEDER_LONG}
    ports:
      - 8082:80
    volumes:
      - "tar1090_heatmap:/var/globe_history"
    tmpfs:
      - /run:exec,size=64M
      - /var/log
```

Once the file has been updated, issue the command `docker-compose up -d` in the application directory to apply the changes and bring up the `tar1090` container.

You should also be able to point your web browser at:

* `http://docker.host.ip.addr:8082/` to view the map showing your currently tracked aircraft.
* After a few hours: `http://docker.host.ip.addr:8082/?heatmap` to see the heatmap for the past 24 hours.
* After a few hours: `http://docker.host.ip.addr:8082/?heatmap&realHeat` to see a different heatmap for the past 24 hours.

Remember to change `docker.host.ip.addr` to the IP address of your docker host.

## Displaying MLAT aircraft in tar1090

Add the following line to the environment section of the `tar1090` section of `docker-compose.yml`:
```yaml
      - MLATHOST=adsbx
```
if using the `adsbx` image, or `MLATHOST=piaware` if using the `piaware` image.

