---
description: >-
  In this step we blacklist the RTL-SDR kernel modules, to ensure the devices
  are available to be used by our containers. You can skip this step if you're
  not using an RTL-SDR radio (eg: bladeRF).
---

# Kernel Module Configuration

**NOTE: If you used the [docker-install.sh](https://github.com/sdr-enthusiasts/docker-install) script, you can skip this section.**

Before we can plug in our RTL-SDR dongle, we need to blacklist the kernel modules for the RTL-SDR USB device from being loaded into the host's kernel and taking ownership of the device.

## **There are four parts to this.**

1. Blacklist modules from being directly loaded AND blacklist modules from being loaded as a dependency of other modules
1. Unload any of our blacklisted modules from memory
1. Rebuild module dependency database
1. Updating the initramfs boot image to remove any references to our now blacklisted modules

### 1. Blacklist Modules

To do this, we will create a blacklist file at `/etc/modprobe.d/blacklist-rtlsdr.conf` with the following command. While logged in as root, please copy and paste all lines at once, and press enter after to ensure the final line is given allowing it to run.

```shell
sudo tee /etc/modprobe.d/blacklist-rtlsdr.conf <<EOF
# Blacklist host from loading modules for RTL-SDRs to ensure they
# are left available for the Docker guest.

blacklist dvb_core
blacklist dvb_usb_rtl2832u
blacklist dvb_usb_rtl28xxu
blacklist dvb_usb_v2
blacklist r820t
blacklist rtl2830
blacklist rtl2832
blacklist rtl2832_sdr
blacklist rtl2838

# This alone will not prevent a module being loaded if it is a
# required or an optional dependency of another module. Some kernel
# modules will attempt to load optional modules on demand, which we
# mitigate here by causing /bin/false to be run instead of the module.
#
# The next time the loading of the module is attempted, the /bin/false
# will be executed instead. This will prevent the module from being
# loaded on-demand. Source: https://access.redhat.com/solutions/41278

install dvb_core /bin/false
install dvb_usb_rtl2832u /bin/false
install dvb_usb_rtl28xxu /bin/false
install dvb_usb_v2 /bin/false
install r820t /bin/false
install rtl2830 /bin/false
install rtl2832 /bin/false
install rtl2832_sdr /bin/false
install rtl2838 /bin/false

EOF
```

### 2. Unload Modules

Next, ensure the modules are unloaded by running the following commands:

```shell
sudo modprobe -r dvb_core
sudo modprobe -r dvb_usb_rtl2832u
sudo modprobe -r dvb_usb_rtl28xxu
sudo modprobe -r dvb_usb_v2
sudo modprobe -r r820t
sudo modprobe -r rtl2830
sudo modprobe -r rtl2832
sudo modprobe -r rtl2832_sdr
sudo modprobe -r rtl2838
```

### 3. Rebuild module dependency database

Next we rebuild the module dependency database with this command:

```shell
sudo depmod -a
```

This may appear to initially not be doing anything, but after a short wait will begin outputting many lines of status updates as it runs until it finishes.

### 4. Update the Boot Image

Now we need to update our boot image to ensure any references to the modules we've blacklisted are removed. (This is only needed on certain systems; feel free to ignore if this command fails.)

```shell
sudo update-initramfs -u
```

This will take a minute or more depending on the speed of your system, and output lots of status message lines as it goes until it is finished.

---

Failure to do the steps above will result in the error below being spammed to the `ultrafeeder` container log.

```shell
usb_claim_interface error -6
rtlsdr: error opening the RTLSDR device: Device or resource busy
```
