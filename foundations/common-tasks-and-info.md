---
description: >-
  Now that we have our first container up and running, common management and
  monitoring tasks and information are outlined below and will apply to the
  remainder of this guide.
---

The following tasks and information will be useful as you continue through this guide. Familiarise yourself with the commands and information on this page.

## Controlling our `adsb` application

### Stopping the `adsb` application

If you need to bring the environment down \(for example, if you need to unplug the RTL-SDR USB dongle for maintenance\), you can issue the command `docker compose down` from the application directory \(the directory containing your `docker-compose.yml` file\).

### Starting the `adsb` application

To start the environment, or apply any changes made to your `docker-compose.yml` file, you can issue the command `docker compose up -d` from the application directory.

### Viewing container logs

To "tail" the consolidated logs of all containers that make up the `adsb` application, you can issue the command `docker compose logs -f` from the application directory.

Individual container logs can be "tailed" with `docker logs -f <container name>`.

If you want to limit the output to, for example, the last 100 lines, you can add `--tail=100` to either of the logs commands above.  If you want to search for a specific word or phrase in the output, you can add a `| grep <keyword>` to the end of the commands.

### Updating containers to latest version

If you would like to manually update your containers to the latest versions of their images, you can run the following commands from the application directory:

```bash
docker compose pull
docker compose up -d
```

### Updating container configuration

If you need to update a container's configuration in `docker-compose.yml` or in the `.env` file, once complete, issue the command `docker compose up -d` \(in the `docker-compose.yml` file's directory\) and the affected containers will be recreated by `docker compose` to reflect the updated configuration.

### Start containers on system boot

All of the containers defined within this document will be configured with the directive `restart: unless-stopped`. This will ensure the containers are automatically started if the host is rebooted unless you have manually stopped the container(s) previously.

## Information on Healthchecks

Images can implement [healthchecks](https://docs.docker.com/engine/reference/builder/). A healthcheck is a script that docker runs within the container periodically that tells docker whether the container is operating as expected.

For example, in the `ghcr.io/sdr-enthusiasts/docker-tar1090` container, the [healthcheck script](https://github.com/sdr-enthusiasts/docker-tar1090/blob/main/rootfs/healthcheck.sh) does the following:

* For each expected network connection, make sure the connection exists
* Make sure that messages are being received from the SDR
* Make sure that the services running within the container aren't dying over and over for some reason

If all of the checks above pass, the container is considered healthy. If any fail, the container is considered unhealthy.

Earlier, we ran the command `docker ps`, to see our newly created `ultrafeeder` container up and running:

```text
CONTAINER ID   IMAGE                                                   COMMAND   CREATED        STATUS                  PORTS                                       NAMES
7b9c4be5a410   ghcr.io/sdr-enthusiasts/docker-adsb-ultrafeeder:latest   "/init"   17 hours ago   Up 17 hours (healthy)   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   ultrafeeder
```

Notice that next to the container status, there is some information about the container's health. This may be one of the following:

* No health information: Not all images have healthchecks implemented. If an image doesn't report health, this is why.
* `(health: starting)`: The container will wait up to a predefined start-period \(defined in the [Dockerfile](https://github.com/sdr-enthusiasts/docker-tar1090/blob/main/Dockerfile#L179)\) or until the healthcheck script returns a healthy result.
* `(healthy)`: The container is operating as expected.
* `(unhealthy)`: The container is not operating as expected.

Later in this guide we will implement a method to automatically restart any unhealthy containers. This way, if your SDR "wedges", the container will eventually go unhealthy and be restarted.

Where practical, we try to include healthchecks in all our images, so as you go through this guide and deploy more containers, you should see a health status whenever you issue the `docker ps` command.

### Reason for healthy/unhealthy?

You can inspect the container for some \(hopefully\) meaningful output from the healthcheck script.

If you issue the command `docker inspect <container name>` \(replacing `<container name>` with the name of the container you're interested in\), you'll see lots of information about the container, including the output of the most recent run of the healthcheck script. This output is in JSON format, so with the help of the `jq` utility we can easily find our `ultrafeeder` container's most recent health information. First make sure `jq` is installed with `sudo apt install -y jq`, then we can check the `ultrafeeder` container's health with the following command:

```bash
docker inspect ultrafeeder | jq .[0].State.Health.Log | jq .[-1].Output | awk '{gsub(/\\n/,"\n")}1'
```

Which will return something like this:

```text
"readsb last updated: 1683575756.737, now: 1683575757.348255120, delta: .611255120. HEALTHY
nginx deaths: 0. HEALTHY
readsb deaths: 0. HEALTHY
tar1090 deaths: 0. HEALTHY
"
```

An `ExitCode` of `0` represents a healthy result. An `ExitCode` of `1` represents an unhealthy result.
The first line is showing that we've received messages from the SDR in the past 15 minutes. The remaining lines show the number of times a service has had an "abnormal death" \(crashed for some reason\).

### Disabling Healthchecks

On systems with with low spec CPU/memory, you may wish to disable healthchecks to gain back some precious CPU cycles. There are ways to disable the docker healthchecks in the `docker-compose.yml` file. However, versions of these images have been published that have the healthcheck removed, so it may be easier to simply change the image's `latest` tag to `latest_nohealthcheck`.
