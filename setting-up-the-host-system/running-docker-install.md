# Installing Docker

## docker-install

The [docker-install.sh](https://github.com/sdr-enthusiasts/docker-install) script helps users get ready to use the SDR-Enthusiasts' Docker containers.
The script is written to be used on a Debian (Ubuntu, Raspberry Pi OS, or DietPi OS) system that is "barebones", i.e., where Docker has not yet been installed. Debian OS versions Stretch, Buster, and Bullseye are supported.

It will **check**, and if necessary **install** the following components and settings:

- `docker`
  - install Docker
  - (optional) add the current user to the `sudoers` group and enable password-free use of `sudo`
  - configure log limits for Docker
  - configure $PATH environment for Docker
  - add current user to `docker` group
- `docker compose`
  - Install latest stable `docker compose` plugin
- Make sure that `libseccomp2` is of a new enough version to support Bullseye-based Docker containers
- Update `udev` rules for use with RTL-SDR dongles
- Blacklist SDR drivers so the `SDR-Enthusiasts`' ADSB and ACARS containers can access the RTL-SDR dongles. Unload any preloaded drivers.
- on `dhcpd` based systems, exclude Docker Container-based virtual ethernet interfaces from using DHCP

After running this script, your system should be ready to use `docker` and `docker compose`. A sample `docker-compose.yml` has been included in the `docker-install` repository.

### How to run it?

- Feel free to inspect the script [here](https://raw.githubusercontent.com/sdr-enthusiasts/docker-install/main/docker-install.sh). You should really not blindly run other people's scripts - make sure you feel comfortable with what it does before executing it.
- To use it, you can enter the following command:

```bash
bash <(curl -s https://raw.githubusercontent.com/sdr-enthusiasts/docker-install/main/docker-install.sh)
```

### Troubleshooting

This script is a work of love, and we don't currently provide support for alternative platforms or configurations.
Feel free to reuse those parts of the script that fit your purpose, subject to the License grant provided with the script.
If you need help or find a bug, please raise an issue on [Github](https://github.com/sdr-enthusiasts/docker-install/issues).
If you have improvements that you'd like to contribute, please submit a PR.

### Errors and how to deal with them

- ISSUE: The script fails with the message below:

```text
E: Repository 'http://raspbian.raspberrypi.org/raspbian buster InRelease' changed its 'Suite' value from 'stable' to 'oldstable'
E: Repository 'http://archive.raspberrypi.org/debian buster InRelease' changed its 'Suite' value from 'testing' to 'oldstable'
```

- SOLUTION: First run `sudo apt-get update --allow-releaseinfo-change && sudo apt-get upgrade -y` and then run the install script again.
