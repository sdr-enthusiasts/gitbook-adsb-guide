---
description: 'If you wish to feed the new ADS-B aggregators, follow the steps below.'
---

# New Aggregators

After ADSBExchange was acquired by a data aggregator, several of the collaborators of this group split off and started their own ADS-B aggregation services based on @wiedehopf's open source software. Rather than developing a separate container for each of them, we created a `multifeeder` container that can be configured to feed them all.

`multifeeder` supports:

* feeding ADS-B data to these aggregators
* feeding MLAT data to the aggregators
* receiving MLAT results from each of the aggregators

The docker image [`ghcr.io/sdr-enthusiasts/docker-multifeeder`](https://github.com/sdr-enthusiasts/docker-multifeeder) contains the required feeder software and all required prerequisites and libraries. This needs to run in conjunction with `readsb`, `tar1090`, or another Beast format data provider.

## Setting up Your Station

### Deploying feeder container

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file \(inside the `services:` section\). Please edit the following parameters:

* `READSB_NET_CONNECTOR`
  * remove `dump978,37981,raw_in;` if you don't have a UAT (978MHz) dongle
  * add any additional "new aggregators" to the end of the line separated by `;`. The format is `feeder_hostname,feeder_beast_port,beast_out`
* `UUID` - enter your UUID here -- see below on how to generate this
* `MLAT_CONFIG` - add any additional "new aggregators" to the end of the line separated by `;`. The format is `feeder_mlat_hostname,feeder_mlat_port,local_mlat_results_port`
* The `FEEDER_LAT`, `FEEDER_LONG`, and `FEEDER_ALT_M` variables should already exist in your `.env` file, but if they don't feel free to replace these values with your station's Latitude, Longitude (in decimal degrees), and Altitude above the ground (in (whole) meters)

```yaml
  multifeeder:
    image: ghcr.io/sdr-enthusiasts/docker-multifeeder
    tty: true
    container_name: multifeeder
    hostname: multifeeder
    restart: always
    environment:
      - TZ=${FEEDER_TZ}
      - READSB_NET_CONNECTOR=readsb,30005,beast_in;dump978,37981,raw_in;feed.adsb.fi,30004,beast_reduce_plus_out;feed.adsb.one,64004,beast_reduce_plus_out;in.adsb.lol,30004,beast_reduce_plus_out;feed.theairtraffic.com,30004,beast_out;feed.planespotters.net,30004,beast_reduce_plus_out
      - UUID=00000000-0000-0000-0000-000000000000
      - MLAT_CONFIG=feed.adsb.fi,31090,39000;feed.adsb.one,64006,39001;in.adsb.lol,31090,39002;feed.theairtraffic.com,31090,39003;mlat.planespotters.net,31090,39004
      - READSB_LAT=${FEEDER_LAT}
      - READSB_LON=${FEEDER_LONG}
      - READSB_ALT=${FEEDER_ALT_M}m
    tmpfs:
      - /run/readsb
      - /var/log
```

### How to generate a UUID

If you already have a `UUID` that was generated for the ADSBExchange service, feel free to reuse that one. If you don't have one, you can generate one by logging onto you Linux machine (Raspberry Pi, etc.) and giving this command:

```bash
cat  /proc/sys/kernel/random/uuid
```

You can use the output string of this command (in format of `00000000-0000-0000-0000-000000000000`) as your UUID. Please use the same UUID consistently for all feeders of your station.

### Using the MLAT results

See [https://github.com/sdr-enthusiasts/docker-multifeeder#receiving-mlat-results](https://github.com/sdr-enthusiasts/docker-multifeeder#receiving-mlat-results) for more details on how to configure this.

### What's going on?

To explain what's going on in this addition:

* We're creating a container called `multifeeder`, from the image `ghcr.io/sdr-enthusiasts/docker-multifeeder`.
* We're passing several environment variables to the container (see above)one as your host system

Once the file has been updated, issue the command `docker compose pull && docker compose up -d` in the application directory to apply the changes and bring up the `multifeeder` container. You should see the following output:

```text
readsb is up-to-date
adsbx is up-to-date
piaware is up-to-date
fr24 is up-to-date
pfclient is up-to-date
Creating multifeeder...
```

We can view the logs for the environment with the command `docker logs multifeeder`, or continually "tail" them with `docker logs -f multifeeder`. The logs will be fairly unexciting and look like this:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
...
[cont-init.d] 01-print-container-version: executing...
[Wed Feb  1 16:38:50 EST 2023][INIT] Container Version: 20230201-184212_489b3d0_main, build date 2023-02-01 13:42:12 -0500
[cont-init.d] 01-print-container-version: exited 0.
[cont-init.d] done.
[services.d] starting services
[Wed Feb  1 16:38:50 EST 2023][multifeeder/mlat-client] Started as an s6 service
[2023/02/01 16:38:50][readsb] invoked by: /usr/local/bin/readsb --net --quiet --lat 42.40487 --lon -71.16615 --uuid-file=/run/uuid --write-json=/run/readsb --json-trace-interval 15 --json-reliable 1 --net-ri-port=30001 --net-ro-port=30002 --net-sbs-port=30003 --net-bi-port=30004,30104 --net-bo-port=30005 --net-beast-reduce-out-port=30006 --net-json-port=30047 --net-api-port=30152 --net-sbs-in-port=32006 --net-connector=readsb,30105,beast_in --net-connector=feed.adsb.fi,30004,beast_out --net-connector=feed.adsb.one,64004,beast_out --net-connector=in.adsb.lol,30004,beast_out
[2023/02/01 16:38:50][readsb] Wed Feb  1 16:38:50 2023 EST  readsb starting up.
...
[Wed Feb  1 16:38:50 EST 2023][./run] starting: /usr/bin/mlat-client --input-type auto --input-connect localhost:30005 --server feed.adsb.fi:31090 --lat 42.40487 --lon -71.16615 --alt 18m --user kx1t-test --results beast,listen,39000
[2023/02/01 16:38:50][readsb] Beast TCP input: Connection established: readsb port 30105
[2023/02/01 16:38:50][readsb] Raw TCP input: Connection established: dump978 port 30978
[2023/02/01 16:38:50][./run][feed.adsb.fi] mlat-client 0.4.2 starting up
[2023/02/01 16:38:50][./run][feed.adsb.fi] Listening for Beast-format results connection on port 39000
[2023/02/01 16:38:50][readsb] Beast TCP output: Connection established: feed.adsb.one (198.50.158.1) port 64004
[2023/02/01 16:38:50][readsb] Beast TCP output: Connection established: in.adsb.lol (142.132.241.63) port 30004
[2023/02/01 16:38:50][readsb] Beast TCP output: Connection established: feed.adsb.fi (65.109.2.208) port 30004
[2023/02/01 16:38:50][./run][feed.adsb.fi] Connected to multilateration server at feed.adsb.fi:31090, handshaking
...
```

## List of Aggregators

| **Site**          | **readsb_url**         | **readsb_port** | **mlat_url**           | **mlat_port** |
|-------------------|------------------------|-----------------|------------------------|---------------|
| [adsb.fi](https://adsb.fi/)           | feed.adsb.fi           |           30004 | feed.adsb.fi           |         31090 |
| [ADSB.lol](https://adsb.lol/)          | feed.adsb.lol          |            1337 | feed.adsb.lol          |          1338 |
| [ADSB One](https://adsb.one/)          | feed.adsb.one          |           64004 | feed.adsb.one          |         64006 |
| [Planespotters.net](https://www.planespotters.net/)          | feed.planespotters.net          |           30004 | mlat.planespotters.net         |         31090 |
| [The Air Traffic](https://theairtraffic.com/) | feed.theairtraffic.com |           30004 | feed.theairtraffic.com |         31090 |

## More information and support

* There is extensive documentation available on the container's [GitHub](https://github.com/sdr-enthusiasts/docker-multifeeder) page.
* You can always find help on the #adsb-containers channel on the [SDR Enthusiasts Discord server](https://discord.gg/m42azbZydy). This channel is meant for Noobs (beginners) and Experts alike.
