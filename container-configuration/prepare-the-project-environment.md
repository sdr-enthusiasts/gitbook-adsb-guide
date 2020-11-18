# Prepare the Project Environment

### Create a directory to host our project

We need a directory to host our project. The name of this directory will be the name of our project. Accordingly, I prefer to use `/opt/adsb`, so our project is called "adsb":

```bash
mkdir -p /opt/adsb
cd /opt/adsb
```

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

* `FEEDER_ALT_FT` is set your your antenna's height in feet
* `FEEDER_ALT_M` is set to your antenna's height in metres
* `FEEDER_LAT` is set to your antenna's latitude
* `FEEDER_LONG` is set to your antenna's longitude
* `FEEDER_TZ` is set to your timezone, in ["TZ database name" format](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones>).

For example:

```text
FEEDER_ALT_FT=103.3465
FEEDER_ALT_M=31.5
FEEDER_LAT=-31.9505
FEEDER_LONG=115.8605
FEEDER_TZ=Australia/Perth
```

**Note for beginners:** If you run an `ls` command in that directory, you won't see your `.env` file. Files beginning with a period are treated as hidden files. To see the file, you can run `ls -a` \(`-a` for all files\).

