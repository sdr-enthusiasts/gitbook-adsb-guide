---
description: >-
  The following steps will guide you through deploying the
  "containrrr/watchtower" container, which will upgrade any containers whenever
  a new version of the image is released.
---

# Auto-Upgrade Containers

The containers used in this guide are regularly updated - most of them daily. This ensures that:

* Any security updates of the underlying containers \(for example: `debian/stable-slim`\) are captured in these containers.
* Receiver \(`readsb`\), feeders, visualisation services \(`tar1090`\) etc are also regularly updated. This ensures that updates are captured in these containers.
* As issues are raised and fixed, it ensures that fixes are present in these containers.

We can configure a container to regularly \(daily\) check DockerHub for new versions of underlying images, automatically pull the new versions and recreate your containers.

This can be a double-edged sword, as container functionality may change between versions \(for example, if a feeder drastically changes how their application behaves\). Rest assured that if behaviour does change, I'll make every effort to ensure backwards compatibility. Accordingly, if you implement auto-upgrade, I'd suggest a regular check of your environment to ensure it is operating as expected.

The container `containrrr/watchtower` has been created to automatically update containers when a new image is released.

There are two ways to implement `containrrr/watchtower`:

1. Monitor all containers and upgrade any that have a new image released
2. Only monitor and upgrade a specific set of containers

This page will discuss both methods.

## A Word About Security

The `containrrr/watchtower` image requires that the container has access to the host's docker socket: `/var/run/docker.sock`.

Mounting `/var/run/docker.sock` inside a container effectively gives the container and anything running within it root privileges on the underlying host, since now you can do anything that a root user and a member of the `docker` group can.

For example, you could create a container, mount the host's `/etc`, modify configurations to open an attack vector to take over the host.

The image has been around since 2015, has several thousand stars, [the source code is available for public scrutiny](https://github.com/containrrr/watchtower) and many people run it \(myself included\), so the risk of nefarious behaviour by the authors is \(in my opinion\) fairly low. Nevertheless, if you decide to give this container access to the host's `/var/run/docker.sock`, you do so at your own risk.

## Monitor and Upgrade All Containers

Open the `docker-compose.yml` file that was created when deploying `readsb`.

Append the following lines to the end of the file \(inside the `services:` section\):

```yaml
  watchtower:
    image: containrrr/watchtower:latest
    tty: true
    container_name: watchtower
    restart: always
    environment:
      - TZ=${FEEDER_TZ}
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_POLL_INTERVAL=86400
      - WATCHTOWER_ROLLING_RESTART=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

To explain what's going on in this addition:

* We're creating a container called `watchtower`, from the image `containrrr/watchtower:latest`.
* We're passing several environment variables to the container:
  * `WATCHTOWER_CLEANUP=true` Removes old images after updating. When this flag is specified, watchtower will remove the old image after restarting a container with a new image. This prevents the accumulation of orphaned images on your system as containers are updated.
  * `WATCHTOWER_POLL_INTERVAL=86400` Poll interval \(in seconds\). This value controls how frequently watchtower will poll for new images. This is set to 24 hours to prevent hitting DockerHub's [new pull limits](https://www.docker.com/increase-rate-limits?utm_source=docker&utm_medium=web%20referral&utm_campaign=pull%20limits%20hub%20home%20page&utm_budget=).
  * `WATCHTOWER_ROLLING_RESTART=true` Restart one image at time instead of stopping and starting all at once. Prevents clobbering the CPU if you run a low power system.
  * `TZ=${FEEDER_TZ}` So that the container's logs are in our local timezone.
* We're passing through the docker socket `/var/run/docker.sock` so that autoheal can control docker \(to restart containers\).

Once the file has been updated, issue the command `docker-compose up -d` in the application directory to apply the changes and bring up the `watchtower` container. You should see the following output:

```text
readsb is up-to-date
adsbx is up-to-date
piaware is up-to-date
fr24 is up-to-date
pfclient is up-to-date
rbfeeder is up-to-date
adsbhub is up-to-date
opensky is up-to-date
autoheal is up-to-date
Creating watchtower
```

We can view the logs for this container with the command `docker logs watchtower`, or continually "tail" them with `docker logs -f watchtower`. The logs will be fairly unexciting initially and look like this:

```text
INFO[0001] Starting Watchtower and scheduling first run: 2020-12-17 18:19:28 +0800 AWST m=+86401.067704768
```

The `watchtower` container logs messages when it upgrades a container, for example:

```text
INFO[864207] Found new ghcr.io/sdr-enthusiasts/docker-readsb-protobuf:latest image (cc3a1b572023)
INFO[864334] Stopping /readsb (38bd04997c8d) with SIGTERM
INFO[864339] Creating /readsb
```

