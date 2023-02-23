---
description: >-
  In this step, we prepare the folder structure for our "adsb" application, and
  create a ".env" file containing our basic details.
---

# Prepare the Application Environment

## Create a directory to host our project

We need a directory to host our application. The name of this directory will be the name of our application. Accordingly, we prefer to use `/opt/adsb`, so our application is called "adsb":

```bash
sudo mkdir -p /opt/adsb
```

```bash
cd /opt/adsb
```

You will likely also want to change the ownership of this directory to your regular user account, so you don't have to use `sudo` to edit the files within. To do this:

```bash
sudo chown $(id -u) /opt/adsb
```

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
```

...where:

* `FEEDER_ALT_FT` is set your your antenna's height in feet above [mean sea level](https://www.freemaptools.com/elevation-finder.htm)
* `FEEDER_ALT_M` is set to your antenna's height in metres above [mean sea level](https://www.freemaptools.com/elevation-finder.htm)
* `FEEDER_LAT` is set to your antenna's latitude (also available at link above)
* `FEEDER_LONG` is set to your antenna's longitude (also available at link above)
* `FEEDER_TZ` is set to your timezone, in ["TZ database name" format](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

For example:

```text
FEEDER_ALT_FT=103.3465
FEEDER_ALT_M=31.5
FEEDER_LAT=-31.9505
FEEDER_LONG=115.8605
FEEDER_TZ=Australia/Perth
```

**Note for beginners:** If you run an `ls` command in that directory, you won't see your `.env` file. Files beginning with a period are treated as hidden files. To see the file, you can run `ls -a` \(`-a` for all files\).
