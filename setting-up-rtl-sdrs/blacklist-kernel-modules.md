---
description: >-
  In this step we blacklist the RTL-SDR kernel modules, to ensure the devices
  are available to be used by our containers. You can skip this step if you're
  not using an RTLSDR radio (eg: bladeRF).
---

# Blacklist Kernel Modules

Before we can plug in our RTL-SDR dongle, we need to blacklist the kernel modules for the RTL-SDR USB device from being loaded into the host's kernel and taking ownership of the device.

To do this, create a file `/etc/modprobe.d/blacklist-rtl2832.conf` containing the following:

```text
blacklist rtl2832
blacklist dvb_usb_rtl28xxu
blacklist rtl2832_sdr
```

Ensure the modules are unloaded by running the following commands:

```text
sudo rmmod rtl2832_sdr
sudo rmmod dvb_usb_rtl28xxu
sudo rmmod rtl2832
```

Failure to do the steps above will result in the error below being spammed to the `readsb` container log.

```text
usb_claim_interface error -6
rtlsdr: error opening the RTLSDR device: Device or resource busy
```

