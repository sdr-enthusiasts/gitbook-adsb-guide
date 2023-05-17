---
description: >-
  In this step, we prepare the folder structure for our "adsb" application, and
  create a ".env" file containing our basic details.
---

# Prepare the Application Environment

## Create a directory to host our project

We need a directory to host our application. The name of this directory will be the name of our application. Accordingly, we prefer to use `/opt/adsb`, so our application is called "adsb":

```bash
sudo mkdir -p -m 777 /opt/adsb
cd /opt/adsb
```

## Generate a UUID

A UUID is a unique identifier that will identify you to various feeding servers. If you already have a `UUID` that was generated for the ADSBExchange service, feel free to reuse that one. If you don't have one, you can generate one by logging onto you Linux machine (Raspberry Pi, etc.) and giving this command:

```bash
cat  /proc/sys/kernel/random/uuid
```

You can use the output string of this command (in format of `00000000-0000-0000-0000-000000000000`) as your UUID. Please use the same UUID consistently for each feeder of your station.

## Identify your ADS-B dongle's optimal PPM

Every RTL-SDR dongle will have a small frequency error as it is cheaply mass produced and not tested for accuracy. This frequency error is linear across the spectrum, and can be adjusted in most SDR programs by entering a PPM (parts per million) offset value. This  allows you to adjust the PPM figure using the ADSB_SDR_PPM environment variable.

Unplug all SDRs, leaving only the SDR to be used for 1090MHz reception plugged in. Issue the following command:

`docker run --rm -it --entrypoint /scripts/estimate_rtlsdr_ppm.sh --device /dev/bus/usb ghcr.io/sdr-enthusiasts/docker-readsb-protobuf:latest`

This takes about 30 minutes and will print a numerical value for Estimated optimum PPM setting.

## Create a heywhatsthat Panorama ID

Heywhatsthat is a website that can generate an overlay on your map that will show the theoretical range of your location based on obstacles and the curvature of the earth. Follow step 1 at the instructions [here](https://github.com/wiedehopf/tar1090#heywhatsthatcom-range-outline) to generate a panorama for your feeder's location and altitude. In the upper left of the panorama page there will be a URL that will look like this: `https://www.heywhatsthat.com/?view=NN3NNNN1`. That code will be used later in the setup instructions.

## Create a `.env` file to hold our environment's variables

Inside this directory, create a file named `.env` using your favourite text editor. Beginners may find the editor `nano` easy to use:

```bash
nano /opt/adsb/.env
```

This file will hold all of the commonly used variables \(such as our latitude, longitude and altitude\). Initially, add the contents of the file as follows \(replacing the values enclosed in `<>` with values for your environment:

```text
FEEDER_ALT_FT=<your antenna's altitude in feet>
FEEDER_ALT_M=<your antenna's altitude in metres>
FEEDER_LAT=<your latitude>
FEEDER_LONG=<your longitude>
FEEDER_TZ=<your timezone>
FEEDER_NAME=<your location name>
ADSB_SDR_SERIAL=1090
ADSB_SDR_GAIN=<your desired gain>
ADSB_SDR_PPM=<your PPM from the step above>
ULTRAFEEDER_UUID=<your UUID from the step above>
FEEDER_HEYWHATSTHAT_ID=<your heywhatsthat ID from the step above>
FEEDER_HEYWHATSTHAT_ALTS=<desired theoretical range altitudes>

```

...where:

* `FEEDER_ALT_FT` is set your your antenna's height in feet above [mean sea level](https://www.freemaptools.com/elevation-finder.htm)
* `FEEDER_ALT_M` is set to your antenna's height in metres above [mean sea level](https://www.freemaptools.com/elevation-finder.htm)
* `FEEDER_LAT` is set to your antenna's latitude (also available at link above)
* `FEEDER_LONG` is set to your antenna's longitude (also available at link above)
* `FEEDER_TZ` is set to your timezone, in ["TZ database name" format](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones). You can also see your Pi's timezone by giving this command: `cat /etc/timezone`
* `FEEDER_NAME` is set to a location name. This is only used in the title of the map's web page.
* `ADSB_SDR_SERIAL` is set to the serial number for your ADS-B dongle; the previous steps set this to 1090 by default but if you have used a different serial number enter it here
* `ADSB_SDR_GAIN` is set to your desired dongle gain in dB, or `autogain` if you would like the software to determine the optimal gain
* `ADSB_SDR_PPM` is set to your desired dongle PPM setting. Enter the number from the PPM estimation step earlier on this page. 
* `ULTRAFEEDER_UUID` is set to the UUID you generated above
* `FEEDER_HEYWHATSTHAT_ID` is set to the code in the URL generated above
* `FEEDER_HEYWHATSTHAT_ALTS` is a comma delimited list of altitudes in meters for which the map will display a theoretical maximum range; a common starting position is 3000 meters and 12000 meters

For example:

```text
FEEDER_ALT_FT=103.3465
FEEDER_ALT_M=31.5
FEEDER_LAT=-31.9505
FEEDER_LONG=115.8605
FEEDER_TZ=Australia/Perth
ADSB_SDR_SERIAL=1090
ADSB_SDR_GAIN=autogain
ADSB_SDR_PPM=1
ULTRAFEEDER_UUID=00000000-0000-0000-0000-000000000000
FEEDER_HEYWHATSTHAT_ID=NN3NNNN1
FEEDER_HEYWHATSTHAT_ALTS=3000,12000
```

**Note for beginners:** If you run an `ls` command in that directory, you won't see your `.env` file. Files beginning with a period are treated as hidden files. To see the file, you can run `ls -a` \(`-a` for all files\).
