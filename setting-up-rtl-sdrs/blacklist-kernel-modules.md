---
description: >-
  In this step we blacklist the RTL-SDR kernel modules, to ensure the devices
  are available to be used by our containers. You can skip this step if you're
  not using an RTLSDR radio (eg: bladeRF).
---

# Blacklist Kernel Modules

Before we can plug in our RTL-SDR dongle, we need to blacklist the kernel modules for the RTL-SDR USB device from being loaded into the host's kernel and taking ownership of the device.

To do this, we will create a blacklist file at `/etc/modprobe.d/blacklist-rtlsdr.conf` with the following command. Please copy and paste all lines at once, and press enter after to ensure the final line is given allowing it to run.

```bash

sudo tee /etc/modprobe.d/blacklist-rtlsdr.conf > /dev/null <<TEXT1
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
TEXT1

```


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

Failure to do the steps above will result in the error below being spammed to the `readsb` container log.

```text
usb_claim_interface error -6
rtlsdr: error opening the RTLSDR device: Device or resource busy
```

