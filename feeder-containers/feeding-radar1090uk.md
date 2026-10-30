---
description: "If you wish to feed Radar1090 UK, follow the steps below."
---

# Feeding Radar1090 UK

[Radar1090 UK](https://www.1090mhz.uk/) is an aggregator based in the UK. They are mostly interested in getting data feeds from the UK, the Republic of Ireland, and its direct neighboring countries, so if you are located in their operating area, feel free to start feeding them.

The docker image [`ghcr.io/sdr-enthusiasts/docker-radar1090`](https://github.com/sdr-enthusiasts/docker-radar1090) contains the required feeder software and all required prerequisites and libraries. This needs to run in conjunction with `ultrafeeder`, `tar1090`, or another RAW provider.

This container helps feeding your ADS-B data to [Radar1090 UK](https://www.1090mhz.uk/). Radar1090 UK is an aggregator based in the UK. They are mostly interested in getting data feeds from the UK, the Republic of Ireland, and its direct neighboring countries, so if you are located in their operating area, feel free to start feeding them.

## Getting a sharing key

The following information was copied from the [Radar1090 UK website](https://www.1090mhz.uk/setup.html). Please go there for the most accurate and latest way to get a sharing key.

> **Station name**
> We need to give your receiver a "station name". Station names string of 3-9 characters in length which can be a placename, callsign, nickname, etc.
>
> We have receivers called things like:
>
> FERNHILL (place)
> PENBREY (place)
> BIRTIES (nickname)
> In many cases an approximate location works well as it reminds us where the receiver is located but provides a degree of privacy.
>
> **Antenna location**
> We need the Latitude/Longitude of your antenna...
>
> The latitude and longitude of your antenna should be in degrees and decimal degrees to six decimal places, for example 52.212457 -2.187936. You can get your antenna's location using Google Maps.

Please send the desired station name and latitude/longitude to Mike Tubby via [email](mailto:mike@tubby.org) or [WhatsApp](whatsapp:+441905888020).

If accepted, you will receive a feeder key that looks like this: `0x7A3DF151D95F3E9A`. You may also receive an optional passphrase.

## Installing `docker-radar1090`

### Prerequisites

You need an existing feeder setup supporting the Beast protocol (or AVR, bur we strongly recommend using Beast). This setup can either be containerized or installed directly on your machine. If you don't have this, here are a few resources that can help you get started:

- [ADSB Gitbook](https://sdr-enthusiasts.gitbook.io/ads-b/) with all the background information you need to set up your containerized station in a way that is easy to understand
- [adsb.im](https://adsb.im) which is a GUI based setup mechanism that creates an SD card image for your SBC that is ready to use for receiving ADSB data with a SDR dongle
- [SDR Enthusiasts' Docker Install Repo](https://github.com/sdr-enthusiasts/docker-install) with a script to easily add and setup Docker on any Debian Linux machine, and example `docker-compose.yml` and `.env` files that you can use as a starting point for your own deployment.

### Adding and configuring Radar1090 UK

Once you have your basic feeder installed, you can add the following minimal configuration to your `docker-compose.yml` file to start feeding Radar1090 UK. Additionally supported parameters are listed further down this document.

```yaml
radar1090:
  image: ghcr.io/sdr-enthusiasts/docker-radar1090:latest
  container_name: radar1090
  hostname: radar1090
  restart: always
  environment:
    - TZ=${FEEDER_TZ}
    - RADAR1090_KEY=${RADAR1090_KEY} # replace ${RADAR1090_KEY} with your sharing key
    - HMAC_KEY=${RADAR1090_PASSPHRASE} # replace ${RADAR1090_PASSPHRASE} with the "secret" passphrase you received. Leave blank if you don't have a passphase.
    - VERBOSE=false # set to true to get lots of feeder information in your container logs
    - BEASTHOST=ultrafeeder # replace ultrafeeder with the container name of your ADSB receiver or the IP address of your host machine if your feeder is not in the same container stack
  tmpfs:
    - /run:exec,size=256M
    - /tmp:size=128M
    - /var/log:size=32M
```

Once you have added this to your setup, simply do `docker compose up -d` in the directory where your `docker-compose.yml` file is located, and you will start feeding your ADS-B data to Radar1090 UK!

## Watchdog

The container uses an internal Watchdog to ensure that data is still flowing to the Radar1090 Server. Data flow can stop for any reason, and often the container can self-repair to get the data flow starting again.
The watchdog runs by default every 15 minutes, and when it runs, it will sample the data stream for 15 seconds. If no data flow was detected going from the container to the Radar1090 Server, it will try to restart the internal feeder module in an attempt to get data flowing again.

Additionally, it will increase the _failure counter_ (or reset this counter if data is flowing again).
Once the _failure counter_ is greater or equal to 3, the container's HEALTHCHECK will go _UNHEALTHY_. This will enable external management containers like `autoheal` to automatically restart the entire container.

## Supported parameters

The following parameters are supported for the `docker-radar1090` container. Please note that only the `RADAR1090_KEY` parameter is mandatory, the rest are optional.

| Parameter                  | Description                                                                                                                                                                                                 | Default value if omitted |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| `TZ`                       | Sets the timezone for the container, in the format `Europe/London`                                                                                                                                          | Unset (UTC)              |
| `RADAR1090_KEY`            | Sharing Key (in this format `0x7A3DF151D95F3E9A`) as provided by Radar1090 UK                                                                                                                               | Unset                    |
| `BEASTHOST`                | Hostname or IP address of the Beast-format data source. Use the container name if available, or use the host machine's IP address (and not `localhost` or `127.0.0.1`!) if your feeder is not containerized | `ultrafeeder`            |
| `RADARSERVER`              | Hostname or IP address of the Radar1090 Server. You shouldn't have to set this parameter unless Radar1090 asks you to change it                                                                             | `adsb-in.1090mhz.uk`     |
| `HMAC_KEY`                 | HMAC PSK key to be used when secure data transmissions are required. Do not change unless you are requested by the Radar1090b server owner                                                                  | Unset                    |
| `MEASURE_INTERVAL`         | Watchdog measurement interval (in secs) - interval in which the internal Watchdog verifies that data is still flowing to the Radar1090 Server                                                               | `300`                    |
| `MEASURE_TIME`             | Watchdog measurement time (in secs) - How long the internal Watchdog will monitor that stat is still flowing to the Radar1090 Server                                                                        | `15`                     |
| `FAILURES_TO_GO_UNHEALTHY` | HEALTHCHECK related parameter - the minimum number of consecutive Watchdog failures that will make the container go `UNHEALTHY`                                                                             | `3`                      |

## HealthCheck

The container supports Docker's HEALTHCHECK feature. During the startup period of the first 5 minutes of the container running (or until the first HEALTHY status is returned), it will check the container's health every 30 seconds. After this startup period, it will check every 5 minutes if the container is healthy. The result of this HEALTHCHECK is used by external programs like `autoheal` to manage the container and attempt to restart it in case of failure.

The HealthCheck itself performs two tests:

- It checks if the Watchdog has 3 or more dataflow related failures
- It checks if there were 3 or more unsuccessful attempts to resolve the BEASTHOST name.

Note that the number of failures is configurable with the `FAILURES_TO_GO_UNHEALTHY` parameter.

## Logging

- All processes are logged to the container's stdout, and can be viewed with `docker logs [-f] radar1090`.

## Getting help

Please feel free to [open an issue on the project's GitHub](https://github.com/sdr-enthusiasts/docker-radar1090/issues).

We don't always immediately see issues filed on Github. Please join us on the `#adsb-containers` channel on the [SDR-Enthusiasts Discord Server](https://discord.gg/sTf9uYF) where there are always a number of us able and willing to help!

## Advanced

If you want to look at more options and examples for the `radar1090` container, you can find the [docker-radar1090 repository](https://github.com/sdr-enthusiasts/docker-radar1090)

## More information and support

- Please contact [info@1090mhz.uk](mailto:info@1090mhz.uk) for more help.
- You can always find help on the #adsb-containers channel on the [SDR Enthusiasts Discord server](https://discord.gg/m42azbZydy). This channel is meant for Noobs (beginners) and Experts alike.
