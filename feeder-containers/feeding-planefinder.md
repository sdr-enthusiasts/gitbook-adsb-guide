---
description: 'If you wish to feed PlaneFinder, follow the steps below.'
---

# Feeding PlaneFinder

[PlaneFinder](https://planefinder.net/) provides live flight tracking data to industries around the world with customised products for aviation, business intelligence and emerging markets alongside world class apps.

The docker image [`mikenye/planefinder`](https://github.com/mikenye/docker-planefinder) contains PlaneFinder's `pfclient` feeder software and all of its required prerequisites and libraries. This needs to run in conjunction with `readsb` \(or another Beast provider\).

## Getting a Share Code

### Already running `pfclient`

You'll need your _share code_ from your existing feeder.

To get your _share code_, log into your [planefinder.net](https://planefinder.net) account, and go to "Your Receivers". Your share code will be listed next to your existing receiver.

You will need to make sure your existing receiver is shutdown prior to continuing.

### New to `pfclient`

If you're already running `pfclient` and you've followed the steps in the previous command, you can skip this section.

You'll need a _share code_. In order to obtain a PlaneFinder Share Code, we will start a temporary container running `pfclient`, which will run through a configuration wizard and generate a share code.

Run the command:

```text
docker run \
    --rm \
    -it \
    --name pfclient_temp \
    --entrypoint pfclient \
    -p 30053:30053 \
    mikenye/planefinder
```

Once the container has started, you should see a message such as:

```text
2020-04-11 06:45:25.823307 [-] We were unable to locate a configuration file and have entered configuration mode by default. Please visit: http://172.22.7.12:30053 to complete configuration.
```

At this point, open a web browser and go to `http://docker.host.ip.addr:30053` You won't be able to use the URL given in the log output, as the IP address in the log output will show the private, internal IP of the docker container.

In your browser, follow the steps in the configuration wizard. When finished, you'll be given a PlaneFinder Share Code. Save this in safe place.

You can now kill the temporary container by pressing `CTRL-C`.

You should now claim your receiver:

1. Go to [https://www.planefinder.net/](https://www.planefinder.net/)
2. Create an account and/or sign in
3. Go to "Account" &gt; "Manage Receivers"
4. Click "Add receiver" and enter your share code when prompted

## Update `.env` file with sharing key

Inside your application directory \(`/opt/adsb`\), edit the `.env` file using your favourite text editor. Beginners may find the editor `nano` easy to use:

```text
nano /opt/adsb/.env
```

This file holds all of the commonly used variables \(such as our latitude, longitude and altitude\). We're going to add our `pfclient` share code to this file. Add the following line to the file:

```text
PLANEFINDER_SHARECODE=YOURSHARECODE
```

* Replace `YOURSHARECODE` with the share code that was generated in the previous step.

For example:

```text
PLANEFINDER_SHARECODE=zg84632abhf231
```

## Deploying `pfclient` container

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  pfclient:
    image: mikenye/planefinder:latest
    tty: true
    container_name: pfclient
    restart: always
    ports:
      - 30053:30053
    environment:
      - TZ=${FEEDER_TZ}
      - BEASTHOST=readsb
      - LAT=${FEEDER_LAT}
      - LONG=${FEEDER_LONG}
      - SHARECODE=${PLANEFINDER_SHARECODE}
    tmpfs:
      - /run:exec,size=64M
      - /var/log/pfclient
```

To explain what's going on in this addition:

* We're creating a container called `pfclient`, from the image `mikenye/planefinder:latest`.
* We're passing several environment variables to the container:
  * `BEASTHOST=readsb` to inform the feeder to get its ADSB data from the container `readsb`
  * `TZ` will use the `FEEDER_TZ` variable from your `.env` file.
  * `LAT` will use the `FEEDER_LAT` variable from your `.env` file.
  * `LONG` will use the `FEEDER_LONG` variable from your `.env` file.
  * `SHARECODE` will use the `PLANEFINDER_SHARECODE` variable from your `.env` file.
* We're using `tmpfs` for volumes that have regular I/O. Any files stored in a `tmpfs` mount are temporarily stored outside the container's writable layer. This helps to reduce:
  * The size of the container, by not writing changes to the underlying container; and
  * SD Card or SSD wear

Once the file has been updated, issue the command `docker-compose up -d` in the application directory to apply the changes and bring up the `pfclient` container. You should see the following output:

```text
readsb is up-to-date
adsbx is up-to-date
piaware is up-to-date
fr24 is up-to-date
Creating pfclient
```

We can view the logs for the environment with the command `docker-compose logs`, or continually "tail" them with `docker-compose logs -f`. At this stage, the logs will be fairly unexciting and look like this:

```text
pfclient          | [s6-init] making user provided files available at /var/run/s6/etc...exited 0.
pfclient          | [s6-init] ensuring user provided files have correct perms...exited 0.
pfclient          | [fix-attrs.d] applying ownership & permissions fixes...
pfclient          | [fix-attrs.d] done.
pfclient          | [cont-init.d] executing container initialization scripts...
pfclient          | [cont-init.d] 01-pfclient: executing...
pfclient          | [cont-init.d] 01-pfclient: exited 0.
pfclient          | [cont-init.d] done.
pfclient          | [services.d] starting services
pfclient          | [services.d] done.
pfclient          | 2020-04-11 09:14:33.361261 [-] pfclient (4.1.1 i386) started with the following options:
pfclient          | 2020-04-11 09:14:33.361432 [-]      connection_type = 1
pfclient          | 2020-04-11 09:14:33.361437 [-]      tcp_address = readsb
pfclient          | 2020-04-11 09:14:33.361440 [-]      tcp_port = 30005
pfclient          | 2020-04-11 09:14:33.361442 [-]      data_format = 1
pfclient          | 2020-04-11 09:14:33.361445 [-]      aircraft_timeout = 30
pfclient          | 2020-04-11 09:14:33.361448 [-]      select_timeout = 10
pfclient          | 2020-04-11 09:14:33.361450 [-]      web_server_port = 30053
pfclient          | 2020-04-11 09:14:33.361454 [-]      user_latitude = -33.33333
pfclient          | 2020-04-11 09:14:33.361458 [-]      user_longitude = 111.11111
pfclient          | 2020-04-11 09:14:33.361539 [V] Performing NTP sync (1.planefinder.pool.ntp.org)...
pfclient          | 2020-04-11 09:14:33.361679 [-] Web server is now listening on: http://172.99.7.64:30053
pfclient          | 2020-04-11 09:14:33.361698 [-] Echo port is now listening on: 172.99.7.64:30054
pfclient          | 2020-04-11 09:14:33.362179 [-] TCP connection established: readsb:30005
pfclient          | 2020-04-11 09:14:33.723215 [V] NTP sync succeeded with settings:
pfclient          | 2020-04-11 09:14:33.723269 [V]      Stratum: 3
pfclient          | 2020-04-11 09:14:33.723287 [V]      System clock time: 1586596473.7232
pfclient          | 2020-04-11 09:14:33.723299 [V]      Corrected clock time: 1586596473.7181
pfclient          | 2020-04-11 09:14:33.723310 [V]      NTP offset: -0.0052s
pfclient          | 2020-04-11 09:15:38.239652 [-] User location has been verified.
pfclient          | 2020-04-11 09:16:23.809962 [-] Successfully sent 46 aircraft updates across 10 packets (8.00KB)
pfclient          | 2020-04-11 09:18:14.117198 [-] Successfully sent 57 aircraft updates across 10 packets (9.00KB)
pfclient          | 2020-04-11 09:20:04.389081 [-] Successfully sent 53 aircraft updates across 10 packets (8.00KB)
```

Once running, you can visit `http://docker.host.ip.addr:30053` to access the `pfclient` web interface. You can also visit the PlaneFinder website, and go to "Account" &gt; "Manage Receivers" and click your receiver to see your live data and statistics.

