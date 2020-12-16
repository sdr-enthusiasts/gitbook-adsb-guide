---
description: >-
  Now we have our first container up and running, common management and
  monitoring tasks and information are outlined below, and will apply to the
  remainder of this guide.
---

# Container Monitoring and Management

The following tasks and information will be useful as you continue through this guide. Familiarise yourself with the commands and information on this page.

## Controlling our `adsb` application

### Stopping the `adsb` application

If you need to bring the environment down \(for example, if you need to unplug the RTL-SDR USB dongle for maintenance\), you can issue the command `docker-compose down` from the application directory \(the directory containing your `docker-compose.yml` file\).

### Starting the `adsb` application

To start the environment, or apply any changes made to your `docker-compose.yml` file, you can issue the command `docker-compose up -d` from the application directory.

### Viewing container logs

To "tail" the consolidated logs of all containers that make up the `adsb` application, you can issue the command `docker-compose logs -f` from the application directory.

Individual container logs can be "tailed" with `docker logs -f CONTAINER`.

If you want to limit the output to, for example, the last 100 lines, you can add `--tail=100` to either of the logs commands above.

### Updating containers to latest version

If you would like to manually update your containers to the latest versions of their images, you can run the following commands from the application directory:

```bash
docker-compose pull
docker-compose up -d
```

### Updating container configuration

If you need to update a container's configuration in `docker-compose.yml` or in the `.env` file, once complete, issue the command `docker-compose up -d` \(in the `docker-compose.yml` file's directory\) and the affected containers will be recreated by `docker-compose` to reflect the updated configuration.

### Start containers on system boot

All of the containers defined within this document will be configured with the directive `restart: always`. This will ensure the containers are automatically started if the host is rebooted.

## Information on Healthchecks

Images can implement [healthchecks](https://docs.docker.com/engine/reference/builder/). A healthcheck is a script that docker runs within the container periodically that tells docker whether the container is operating as expected.

For example, in the `mikenye/readsb-protobuf` container, the [healthcheck script](https://github.com/mikenye/docker-readsb-protobuf/blob/main/rootfs/scripts/healthcheck.sh) does the following:

* For each expected network connection, make sure the connection exists
* Make sure that messages are being received from the SDR
* Make sure that the services running within the container aren't dying over and over for some reason

If all of the checks above pass, the container is considered healthy. If any fail, the container is considered unhealthy.

Earlier, we ran the command `docker ps`, to see our newly created `readsb` container up and running:

```text
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS                    PORTS                    NAMES
f936a37dd488        mikenye/readsb-protobuf:latest      "/init"             4 hours ago         Up 2 hours (healthy)      0.0.0.0:8080->8080/tcp   readsb
```

Notice that next to the container status, there is some information about the container's health. This may be one of the following:

* No health information: Not all images have healthchecks implemented. If an image doesn't report health, this is why.
* `(health: starting)`: The container will wait up to a predefined start-period \(defined in the [Dockerfile](https://github.com/mikenye/docker-readsb-protobuf/blob/63a617e00d4c9bcd8a5d6a9d94cc2f2b32ac0489/Dockerfile#L286)\) or until the healthcheck script returns a healthy result.
* `(healthy)`: The container is operating as expected.
* `(unhealthy)`: The container is not operating as expected.

Later in this guide we will implement a method to automatically restart any unhealthy containers. This way, if your SDR "wedges", the container will eventually go unhealthy and be restarted.

Where practical, I try to include healthchecks in all my images, so as you go through this guide and deploy more containers, you should see a health status whenever you issue the `docker ps` command.

### Reason for healthy/unhealthy?

You can inspect the container for some \(hopefully\) meaningful output from the healthcheck script.

If you issue the command `docker inspect CONTAINER` \(replacing `CONTAINER` with the name of the container you're interested in\), you'll see lots of information about the container, including the output of the most recent run of the healthcheck script. This output is in JSON format, so if you have the `jq` utility installed, we can easily find our `readsb` container's most recent health information with the following command:

```bash
docker inspect readsb | jq .[0].State.Health.Log | jq .[-1]
```

Which will return something like this:

```javascript
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

The first line is showing that we've received messages from the SDR in the past 15 minutes. The remaining lines show the number of times a service has had an "abnormal death" \(crashed for some reason\).

### Disabling Healthchecks

On systems with with low spec CPU/memory, you may wish to disable healthchecks to gain back some precious CPU cycles. There are ways to disable the docker healthchecks in the `docker-compose.yml` file. However, I publish versions of my images that have the healthcheck removed, so it may be easier to simply change the image's `latest` tag to `latest_nohealthcheck`.

