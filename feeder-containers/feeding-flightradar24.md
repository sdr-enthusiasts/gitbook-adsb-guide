---
description: 'If you wish to feed FlightAware, follow the steps below.'
---

# Feeding FlightRadar24

[FlightRadar24](https://www.flightradar24.com/) is a global flight tracking service that provides real-time information about thousands of aircraft around the world. Their service is currently available online and their mobile app is quite good. In order to access the features of the mobile app, you'll need to feed your ADS-B data to them for a free business plan.

`fr24feed` is a FlightRadar24 client program to securely transmit ADS-B and Mode S data to the commercial entity FlightRadar24.

I've created a docker image [`ghcr.io/sdr-enthusiasts/docker-flightradar24`](https://github.com/sdr-enthusiasts/docker-flightradar24) that contains `fr24feed` and all of its required prerequisites and libraries.

## Getting a Sharing Key

### Already running `fr24feed`?

You'll need your _fr24key_ from your existing feeder.

To get your _fr24key_, log onto your feeder and issue the command:

```bash
cat /etc/fr24feed.ini | grep fr24key
```

### New to `fr24feed`?

If you're already feeding FlightRadar24 and you've followed the steps in the previous command, you can skip this section.

First-time users should obtain a FlightRadar24 sharing key \(a _fr24key_\). To get one, you can run through the sign-up process. This will ask a series of questions allowing you to sign up with FlightRadar24 and get a _fr24key_.

There's an automated script that you can run, however if this breaks \(please let me know so I can fix it\), you can also use the manual sign-up method.

#### Automatic Sign-Up Script Method

Run these commands from within your application directory \(`/opt/adsb`\):

```text
source ./.env
docker run \
  --rm \
  -it \
  -e FEEDER_LAT="$FEEDER_LAT" \
  -e FEEDER_LONG="$FEEDER_LONG" \
  -e FEEDER_ALT_FT="$FEEDER_ALT_FT" \
  -e FR24_EMAIL="YOUR@EMAIL.ADDRESS" \
  --entrypoint /scripts/signup.sh \
  ghcr.io/sdr-enthusiasts/docker-flightradar24
```

Be sure to replace `YOUR@EMAIL.ADDRESS` with your actual email address!

After about 30 seconds or so, if the script method was successful, you should see output similar to this:

```text
FR24_SHARING_KEY=5fa9ca2g9049b615
FR24_RADAR_ID=T-XXXX123
```

Simply copy these lines and paste them into your `.env` file.

If something went wrong, please take a moment to let me know, then try the manual method below.

#### Manual Sign-Up Method

Run the command:

```text
docker run --rm -it --entrypoint fr24feed ghcr.io/sdr-enthusiasts/docker-flightradar24 --signup
```

This will take you through the sign-up process. At the end of the sign-up process, you'll be presented with:

```text
Congratulations! You are now registered and ready to share ADS-B data with Flightradar24.
+ Your sharing key (xxxxxxxxxxxx) has been configured and emailed to you for backup purposes.
+ Your radar id is X-XXXXXXX, please include it in all email communication with us.
```

Copy the sharing key you are given, and add the following line to your `.env` file:

```text
FR24_SHARING_KEY=YOURSHARINGKEY
```

* Replace `YOURSHARINGKEY` with the sharing key from the output of the manual sign-up process.

For example:

```text
FR24_SHARING_KEY=10ae138d0c1g
```

## Deploying `fr24feed` container

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  fr24:
    image: ghcr.io/sdr-enthusiasts/docker-flightradar24:latest
    tty: true
    container_name: fr24
    restart: always
    depends_on:
      - readsb
    ports:
      - 8754:8754
    environment:
      - BEASTHOST=readsb
      - TZ=${FEEDER_TZ}
      - FR24KEY=${FR24_SHARING_KEY}
    tmpfs:
      - /var/log
```

To explain what's going on in this addition:

* We're creating a container called `fr24`, from the image `ghcr.io/sdr-enthusiasts/docker-flightradar24:latest`.
* We're passing several environment variables to the container:
  * `BEASTHOST=readsb` to inform the feeder to get its ADSB data from the container `readsb` network.
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
  * `FR24KEY` will use the `FR24_SHARING_KEY` variable from your `.env` file.
* We're using `tmpfs` for volumes that have regular I/O. Any files stored in a `tmpfs` mount are temporarily stored outside the container's writable layer. This helps to reduce:
  * The size of the container, by not writing changes to the underlying container; and
  * SD Card or SSD wear

Once the file has been updated, issue the command `docker-compose up -d` in the application directory to apply the changes and bring up the `fr24` container. You should see the following output:

```text
readsb is up-to-date
adsbx is up-to-date
piaware is up-to-date
Creating fr24
```

We can view the logs for the environment with the command `docker-compose logs`, or continually "tail" them with `docker-compose logs -f`. At this stage, the logs will be fairly unexciting and look like this:

```text
[s6-init] making user provided files available at /var/run/s6/etc...exited 0.
[s6-init] ensuring user provided files have correct perms...exited 0.
[fix-attrs.d] applying ownership & permissions fixes...
[fix-attrs.d] done.
[cont-init.d] executing container initialization scripts...
[cont-init.d] 01-fr24feed: executing...
[cont-init.d] 01-fr24feed: exited 0.
[cont-init.d] done.
[services.d] starting services
[services.d] done.
2020-11-20 16:35:53 | ______  _  _         _      _                    _              _____    ___
2020-11-20 16:35:53 | |  ___|| |(_)       | |    | |                  | |            / __  \  /   |
2020-11-20 16:35:53 | | |_   | | _   __ _ | |__  | |_  _ __  __ _   __| |  __ _  _ __`' / /' / /| |
2020-11-20 16:35:53 | |  _|  | || | / _` || '_ \ | __|| '__|/ _` | / _` | / _` || '__| / /  / /_| |
2020-11-20 16:35:53 | | |    | || || (_| || | | || |_ | |  | (_| || (_| || (_| || |  ./ /___\___  |
2020-11-20 16:35:53 | \_|    |_||_| \__, ||_| |_| \__||_|   \__,_| \__,_| \__,_||_|  \_____/    |_/
2020-11-20 16:35:53 |                __/ |
2020-11-20 16:35:53 |               |___/
2020-11-20 16:35:53 | info | [httpd]Server started, listening on 0.0.0.0:8754
2020-11-20 16:35:54 | [i]PacketSenderConfiguration::fetch_config(): Yoda configuration for this receiver is disabled
2020-11-20 16:35:54 | [d]TLSConnection::ctor(): Enable verify_peer in production code!
2020-11-20 16:35:55 | [feed][d]fetching configuration
2020-11-20 16:35:56 | [feed][c]Max range AIR: 350.0nm
2020-11-20 16:35:56 | [feed][c]Max range GND: 100.0nm
2020-11-20 16:35:56 | [feed][c]Timestamps: optional
2020-11-20 16:35:56 | info | [stats]Stats thread started
2020-11-20 16:35:56 | info | Stopping ReceiverACSender threads for feed
2020-11-20 16:35:56 | info | Configured ReceiverACSender: 185.218.24.22:8099,185.218.24.23:8099,185.218.24.24:8099, feed: XXXX1234, send_interval: 5s, max age: 15s, send metadata: true, mode: 1, filtering: true
2020-11-20 16:35:56 | info | Network thread connecting to 185.218.24.22:8099 for feed XXXX1234
```

Once running, you can visit `http://docker.host.ip.addr:8754` to access the `fr24feed` web interface. You can also log onto FlightRadar24's website and click on the your profile button, and then "My data sharing" link to see your statistics.

