---
description: 'If you wish to feed RadarPlane.com, follow the steps below.'
---
# RadarPlane/docker-radarplane

Docker container to feed ADS-B data into [RadarPlane](https://radarplane.com). Designed to work in tandem
with [sdr-enthusiasts/docker-readsb-protobuf][docker-readsb-protobuf] or another BEAST provider. Builds and runs on x86,
x86_64, arm32v7 & arm64v8.

The container pulls ADS-B information from a BEAST provider and sends data to [RadarPlane](https://radarplane.com).

For more information on [radarPlane](https://radarplane.com), see
here: [RadarPlane Feeding Instructions](https://radarplane.com/feed). This container uses a modified version of the "
script method" outlined on that page.

## Configuring `RadarPlane/docker-radarplane` Container

* If you're using this container with the [sdr-enthusiasts/docker-readsb-protobuf][docker-readsb-protobuf] container to
  provide ModeS/BEAST data, and the containers are on different docker networks or on different hosts:
    * You'll need to ensure you've opened port `30005` (or whatever port is serving BEAST data) into
      the [sdr-enthusiasts/docker-readsb-protobuf][docker-readsb-protobuf] container.
    * The IP address or hostname of the docker host running
      the [sdr-enthusiasts/docker-readsb-protobuf][docker-readsb-protobuf] container should be passed to
      the `RadarPlane/docker-radarplane` container via the `BEASTHOST` environment variable shown below. The port can be
      changed from the default of `30005` with the optional `BEASTPORT` environment variable if required.
* The latitude and longitude of your antenna must be passed via the `LAT` and `LONG` environment variables respectively.
* The altitude of your antenna must be passed via the `ALT` environment variable respectively. Defaults to metres, but
  units may specified with a 'ft' or 'm' suffix.
* A UUID for this feeder must be passed via the `UUID` environment variable (see below).
* Lastly, you should specify a site name via the `SITENAME` environment variable. This field supports letters,
  numbers, `-` & `_` only. Any other characters will be stripped upon container initialization.

## Generating a site UUID Number

First-time users should generate a static UUID using this command:

```shell
cat /proc/sys/kernel/random/uuid
```

Take note of the UUID returned. You should pass it as the `UUID` environment variable when running the container.

## Up-and-Running with `docker run`

```shell
docker run \
 -d \
 --rm \
 --name radarplane \
 -e TZ=YOUR_TIMEZONE \
 -e BEASTHOST=readsb \
 -e LAT=-33.33333 \
 -e LONG=111.11111 \
 -e ALT=50m \
 -e SITENAME=My_Cool_ADSB_Receiver \
 -e UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
 --tmpfs=/run:rw,nosuid,nodev,exec,relatime,size=64M,uid=1000,gid=1000 \
 ghcr.io/RadarPlane/docker-radarplane:main
```

## Up-and-Running with Docker Compose

First timers are encouraged to
read [ADS-B Reception, Decoding & Sharing with Docker](https://mikenye.gitbook.io/ads-b/).

An example docker compose service definition is below:

```yaml
  radarplane:
    image: ghcr.io/radarplane/docker-radarplane:main
    tty: true
    container_name: radarplane
    restart: always
    environment:
      - BEASTHOST=readsb
      - TZ=UTC
      - LAT=-37.7468
      - LONG=-25.5716
      - ALT=50m
      - SITENAME=My_Cool_RadarPlane_Receiver
      - UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    tmpfs:
      - /run:rw,nosuid,nodev,exec,relatime,size=64M,uid=1000,gid=1000
```

## Runtime Environment Variables

There are a series of available environment variables:

| Environment Variable             | Purpose                                                                                                                                                                                                                                              | Default                 |
|----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------|
| `BEASTHOST`                      | Required. IP/Hostname of a Mode-S/BEAST provider (dump1090)                                                                                                                                                                                          |                         |
| `BEASTPORT`                      | Optional. TCP port number of Mode-S/BEAST provider (dump1090)                                                                                                                                                                                        | `30005`                 |
| `UUID`                           | Required. Your static UUID                                                                                                                                                                                                                           |                         |
| `LAT`                            | Required. The latitude of the antenna                                                                                                                                                                                                                |                         |
| `LONG`                           | Required. The longitude of the antenna                                                                                                                                                                                                               |                         |
| `ALT`                            | Required. The altitude of the antenna above sea level. If positive (above sea level), must include either 'm' or 'ft' suffix to indicate metres or feet. If negative (below sea level), must have no suffix, and the value is interpreted in metres. |                         |
| `SITENAME`                       | Required. The name of your site (A-Z, a-z, `-`, `_`)                                                                                                                                                                                                 |                         |
| `TZ`                             | Optional. Your local timezone                                                                                                                                                                                                                        | `GMT`                   |
| `REDUCE_INTERVAL`                | Optional. How often beastreduce data is transmitted to ADSBExchange. For low bandwidth feeds, this can be increased to `5` or even `10`                                                                                                              | `0.5`                   |
| `PRIVATE_MLAT`                   | Optional. Setting this to true will prevent feeder being shown on the feeder maps                                                                                                                                                                    | `false`                 |
| `MLAT_INPUT_TYPE`                | Optional. Sets the input receiver type. Run `docker run --rm -it --entrypoint mlat-client ghcr.io/RadarPlane/docker-radarplane:latest --help` and see `--input-type` for valid values.                                                               | `dump1090`              |
| `STATS_DISABLE`                  | Optional. Set to any value to disable stats module / anywhere map (if you don't like lots of DNS lookups, set this to 1)                                                                                                                             | unset                   |
| `ADSB_FEED_DESTINATION_HOSTNAME` | Optional. Allows changing the hostname that ADS-B data is fed to.                                                                                                                                                                                    | `feed.radarplane.com`   |
| `ADSB_FEED_DESTINATION_PORT`     | Optional. Allows changing the TCP port that ADS-B data is fed to.                                                                                                                                                                                    | `30001`                 |
| `ADSB_FEED_DESTINATION_TYPE`     | Optional. Allows changing the `readsb` output data type.                                                                                                                                                                                             | `beast_reduce_plus_out` |
| `MLAT_FEED_DESTINATION_HOSTNAME` | Optional. Allows changing the MLAT server hostname.                                                                                                                                                                                                  | `feed.radarplane.com`   |
| `MLAT_FEED_DESTINATION_PORT`     | Optional. Allows changing the MLAT server TCP port.                                                                                                                                                                                                  | `31090`                 |

## Ports

| Port    | Purpose                                                                                                                                     |
|---------|---------------------------------------------------------------------------------------------------------------------------------------------|
| `30105` | MLAT data in Beast format for tools such as [`graphs1090`](https://github.com/mikenye/docker-graphs1090) and/or [`tar1090`][docker-tar1090]

## Logging

* All processes are logged to the container's stdout, and can be viewed with `docker logs [-f] container`.

[docker-readsb-protobuf]: https://github.com/sdr-enthusiasts/docker-readsb-protobuf

[docker-tar1090]: https://github.com/sdr-enthusiasts/docker-tar1090
