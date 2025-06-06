---
description: 'If you wish to feed FlightRadar24, follow the steps below.'
---

# Feeding FlightRadar24

[FlightRadar24](https://www.flightradar24.com/) is a global flight tracking service that provides real-time information about thousands of aircraft around the world. Their service is currently available online and their mobile app is quite good. In order to access the features of the mobile app, you'll need to feed your ADS-B data to them for a free business plan.

`fr24` is a FlightRadar24 client program to securely transmit ADS-B and Mode S data to the commercial entity FlightRadar24.

We've created a docker image [`ghcr.io/sdr-enthusiasts/docker-flightradar24`](https://github.com/sdr-enthusiasts/docker-flightradar24) that contains `fr24` and all of its required prerequisites and libraries.

## Getting a Sharing Key

### Already running `fr24`?

You'll need your _fr24key_ from your existing feeder.

To get your _fr24key_, log onto your feeder and issue the command:

```shell
cat /etc/fr24feed.ini | grep fr24key
```

You can also find it in the data sharing section on the fr24 website if you have an account with the email address that was used when creating the key.

### New to `fr24`?

If you're already feeding FlightRadar24 and you've followed the steps in the previous command, you can skip this section.

First-time users should obtain a FlightRadar24 sharing key \(a _fr24key_\). To get one, you can run through the sign-up process. This will ask a series of questions allowing you to sign up with FlightRadar24 and get a _fr24key_.
Use the same email address as for your fr24 account if you already have one or plan on creating one.


#### Obtaining a Sharing Key for ADSB

Run the command:

```shell
docker run -it --rm ghcr.io/sdr-enthusiasts/docker-baseimage:qemu bash -c "$(curl -sSL https://raw.githubusercontent.com/sdr-enthusiasts/docker-flightradar24/main/get_adsb_key.sh)"
```

This will start up a container. After installing a bunch of software (which may take a while depending on the speed of your machine and internet connection), it will take you through the sign-up process. Most of the answers don't matter as during normal operation the configuration will be set with environment variables. I would suggest answering as follows:

- `Step 1.1 - Enter your email address (username@domain.tld)`: Enter your FlightRadar24 account email address
- `Step 1.2 - If you used to feed FR24 with ADS-B data before, enter your sharing key.`: Leave blank and press enter
- `Step 1.3 - Would you like to participate in MLAT calculations?`: Answer `no`
- `Would you like to continue using these settings?`: Answer `yes`
- `Step 4.1 - Receiver selection (in order to run MLAT please use DVB-T stick with dump1090 utility bundled with fr24feed)... Enter your receiver type (1-7)`: Answer `4`.
- `Enter your connection type`: Answer `1`.
- `host`: Answer: 127.0.0.1
- `port`: Answer: 30005
- `Step 5`: Answer: `no` twice.

Note that there is a limit of 3 feeders per FR24 account. ADSB and UAT (see below) each count as 1 feeder. If you have more than 3 feeders, you will need to contact <support@fr24.com> to request an additional Feeder Key. Make sure to send them your account email-address, latitude, longitude, altitude, and if the key is for an ADSB or UAT feeder.

At the end of the sign-up process, you'll be presented with:

```text
Congratulations! You are now registered and ready to share ADS-B data with Flightradar24.
+ Your sharing key (xxxxxxxxxxxx) has been configured and emailed to you for backup purposes.
+ Your radar id is X-XXXXXXX, please include it in all email communication with us.
```

Copy the sharing key you are given, and add the following line to your `.env` file:

```shell
FR24_SHARING_KEY=YOURSHARINGKEY
```

- Replace `YOURSHARINGKEY` with the sharing key from the output of the manual sign-up process.

For example:

```shell
FR24_SHARING_KEY=10ae138d0c1g
```

#### UAT key and configuration (USA only, requires 2nd SDR and dump978 container already configured)

Get a separate sharing key for UAT as described [here](https://github.com/sdr-enthusiasts/docker-flightradar24?tab=readme-ov-file#uat-configuration-usa-only):

Copy the UAT sharing key you are given, and add the following line to your `.env` file:

```shell
FR24_SHARING_KEY_UAT=YOURSHARINGKEYUAT
```

- Replace `YOURSHARINGKEYUAT` with the sharing key from the output of the sign-up process.

## Deploying `fr24` container

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  fr24:
    image: ghcr.io/sdr-enthusiasts/docker-flightradar24:latest
    container_name: fr24
    restart: unless-stopped
    ports:
      - 8754:8754
    environment:
      - BEASTHOST=ultrafeeder
      - FR24KEY=${FR24_SHARING_KEY}
      - FR24KEY_UAT=${FR24_SHARING_KEY_UAT}
    tmpfs:
      - /var/log
```

To explain what's going on in this addition:

- We're creating a container called `fr24`, from the image `ghcr.io/sdr-enthusiasts/docker-flightradar24:latest`.
- We're passing several environment variables to the container:
  - `BEASTHOST=ultrafeeder` to inform the feeder to get its ADSB data from the container `ultrafeeder` network.
  - `FR24KEY` will use the `FR24_SHARING_KEY` variable from your `.env` file.
- We're using `tmpfs` for volumes that have regular I/O. Any files stored in a `tmpfs` mount are temporarily stored outside the container's writable layer. This helps to reduce:
  - The size of the container, by not writing changes to the underlying container; and
  - SD Card or SSD wear

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `fr24` container. You should see the following output:

```text
 ✔ Container ultrafeeder  Running
 ✔ Container piaware      Running
 ✔ Container fr24         Started
```

We can view the logs for the environment with the command `docker compose logs`, or continually "tail" them with `docker compose logs -f`. At this stage, the logs will be fairly unexciting and look like this:

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

Once running, you can visit `http://docker.host.ip.addr:8754` to access the `fr24` web interface. You can also log onto FlightRadar24's website and click on the your profile button, and then "My data sharing" link to see your statistics.

## Advanced

If you want to look at more options and examples for the `fr24` container, you can find the repository [here](https://github.com/sdr-enthusiasts/docker-flightradar24)
