---
description: 'If you previously did not use the old guide, feel free to skip this page.'
---

# Changes From the Old Guide

If you previously used [the old guide](https://github.com/mikenye/docker-readsb/wiki/Guide-to-ADS-B-Data-Receiving,-Decoding-and-Sharing,-Leveraging-RTLSDR-and-Docker), there are some differences you should be aware of.

## Docker Compose

In the old guide, Python’s `pip` is used to install `docker-compose`. In this guide, `docker-compose` itself is run as a container. This greatly simplifies the installation process as well as upgrading.

## `readsb` Deprecation

The author of `readsb` \([Mictronics](https://github.com/Mictronics)\), has deprecated `readsb` in favour of the newer and better [readsb-protobuf](https://github.com/Mictronics/readsb-protobuf). Some notable differences are:

* No JSON output. This means that tools such as `graphs1090` which relied on the old JSON files will no longer work. Fortunately, the `readsb-protobuf` web interface includes ”performance graphs” which offers the same functionality.
* Different container deployment. The `mikenye/readsb-protobuf` is not a drop-in replacement for `mikenye/readsb`. With `mikenye/readsb`, the command line arguments to the container were passed through to the `readsb` binary. With `miknye/readsb-protobuf`, environment variables are used instead. This makes it easier to implement container healthchecks.
* The `mikenye/readsb-protobuf` container includes ”autogain”, which will \(over a period of several weeks\) determine the best gain kevel for your environment. You can still specify a manual gain setting if preferred.

## Changes to how USB devices are passed through

Previously, `lsusb` was used to determine the USB bus and device number, and the exact device was passed through.

This resulted in issues when people would reboot their host or re-plug their USB device, and the bus & device numbers would change.

To address this, the entire USB bus is passed through. The old method still works if preferred.

## Removal of `adsbnet`

By default, `docker-compose` will place all containers defined in an application into a default network. Accordingly, in an effort to simplify the configuration, I’ve removed this explicit network declaration.



























