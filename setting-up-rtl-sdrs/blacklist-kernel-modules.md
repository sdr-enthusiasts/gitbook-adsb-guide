---
description: >-
  In this step we blacklist the RTL-SDR kernel modules, to ensure the devices
  are available to be used by our containers. You can skip this step if you're
  not using an RTLSDR radio (eg: bladeRF).
---

# Kernel Module Configuration

**NOTE: If you used the [docker-install.sh](https://github.com/sdr-enthusiasts/docker-install) script, you can skip this section.**

Before we can plug in our RTL-SDR dongle, we need to blacklist the kernel modules for the RTL-SDR USB device from being loaded into the host's kernel and taking ownership of the device.

### **There are three parts to this.**
1. Blacklist modules from being directly loaded AND blacklist modules from being loaded as a dependency of other modules
2. Unload any of our blacklisted modules from memory
3. Updating the initramfs boot image to remove any references to our now blacklisted modules

#### 1. Blacklist Modules
To do this, we will create a blacklist file at `/etc/modprobe.d/blacklist-rtlsdr.conf` with the following command. While logged in as root, please copy and paste all lines at once, and press enter after to ensure the final line is given allowing it to run.

```bash

sudo tee /etc/modprobe.d/blacklist-rtlsdr.conf <<TEXT1
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
blacklist rtl8192cu
blacklist rtl8xxxu

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
install rtl8192cu /bin/false
install rtl8xxxu /bin/false

TEXT1

```

#### 2. Unload Modules
Next, ensure the modules are unloaded by running the following commands:

```bash
sudo rmmod dvb_core
sudo rmmod dvb_usb_rtl2832u
sudo rmmod dvb_usb_rtl28xxu
sudo rmmod dvb_usb_v2
sudo rmmod r820t
sudo rmmod rtl2830
sudo rmmod rtl2832
sudo rmmod rtl2832_sdr
sudo rmmod rtl2838
sudo rmmod rtl8192cu
sudo rmmod rtl8xxxu

```
#### 3. Update the Boot Image

Now we need to update our boot image to ensure any references to the modules we've blacklisted are removed
```bash
update-initramfs -u
```
This will take a minute or more depending on the speed of your system, and output lots of status message lines as it goes until it is finished.

-----

Failure to do the steps above will result in the error below being spammed to the `readsb` container log.

```text
usb_claim_interface error -6
rtlsdr: error opening the RTLSDR device: Device or resource busy
```

