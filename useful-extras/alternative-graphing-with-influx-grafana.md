---
description: >-
  If you wish to deploy Grafana for alternative graphs, follow the steps below.
---

# Alternative graphs with Grafana

[`Grafana`](https://grafana.com/) is an analytics platform that can provide alternative graphs for `ultrafeeder`.

In this guide we will be using [`InfluxDB`](https://www.influxdata.com/) as the data repository.

Using Grafana and InfluxDB in this configuration does not require a plan, account, or credentials for their respective cloud offerings.

## Create docker volumes

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Add the following lines to the  `volumes:` section at the top of the file \(below the `version:` section, and before the `services:` section\):

```yaml
  influxdb_data:
  influxdb_config:
  grafana_data:
```

This creates the volumes that will contain `influxdb` and `grafana`’s application data.

## Deploying `influxdb` and `grafana` containers

Open the `.env` file that was created when deploying `ultrafeeder`.

Append the following lines to the end of the file; avoid using surrounding "" for the variables, which can be set to any value you like and token should be thought of as a very strong password:

```properties
INFLUXDB_USER=<your influxdb username>
INFLUXDB_PASSWORD=<your influxdb password>
INFLUXDB_ADMIN_TOKEN=<your influxdb token>
```

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Add the following lines to the `environment` section of the `ultrafeeder` container definition \(in the `ultrafeeder:` section, below `environment:` and before the `volumes:` section\):

```yaml
      - INFLUXDBV2_URL=http://influxdb:8086
      - INFLUXDBV2_BUCKET=ultrafeeder
      - INFLUXDBV2_ORG=ultrafeeder
      - INFLUXDBV2_TOKEN=${INFLUXDB_ADMIN_TOKEN}
```

Append the following lines to the end of the file:

```yaml
  influxdb:
    image: influxdb:latest
    tty: true
    container_name: influxdb
    hostname: influxdb
    restart: always
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_BUCKET=ultrafeeder
      - DOCKER_INFLUXDB_INIT_ORG=ultrafeeder
      - DOCKER_INFLUXDB_INIT_RETENTION=52w
      - DOCKER_INFLUXDB_INIT_USERNAME=${INFLUXDB_USER}
      - DOCKER_INFLUXDB_INIT_PASSWORD=${INFLUXDB_PASSWORD}
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=${INFLUXDB_ADMIN_TOKEN}
    ports:
      - 8086:8086
    volumes:
      - influxdb_data:/var/lib/influxdb2
      - influxdb_config:/etc/influxdb2

  grafana:
    image: grafana/grafana-oss:latest
    tty: true
    container_name: grafana
    hostname: grafana
    restart: always
    ports:
      - 3000:3000
    volumes:
      - grafana_data:/var/lib/grafana
```

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `influxdb` and `grafana` containers. This will also restart the `ultrafeeder` container, which will now use `telegraf` to feed data to `influxdb`.

You should also be able to point your web browser at:

* `http://docker.host.ip.addr:8086/` to access the `influxdb` console, use the credentials from your `.env` file.
* `http://docker.host.ip.addr:3000/` to access the `grafana` console, use admin/admin as initial credentials, you should be prompted to change the password on first login.

Remember to change `docker.host.ip.addr` to the IP address of your docker host.

## Configuring data source and dashboard in Grafana

After you have logged into the `grafana` console the following manual steps are required to connect to `influxdb` as the data source

1. Click `Add your first data source` in the main panel
2. Click `InfluxDB` from the list of options provided
3. Input or select the following options, if the option is not listed, do not input anything for that option (for `Value` the word `Token` must be included in the input:

Option | Input
------------- | -------------
Name | ultrafeeder
Query Language | InfluxQL
URL | `http://influxdb:8086`
Custom HTTP Headers | Click `+ Add header`
Header | Authorization
Value | Token \<your influxdb token\>
Database | ultrafeeder
User | \<your influxdb username\>
Password | \<your influxdb password\>
HTTP Method | GET

Clicking `Save & Test` should return a green message indicating success. The dashboard can now be imported with the following steps

1. Hover over the `four squares` icon in the sidebar, click `+ Import`
2. Enter `13168` into the `Import via grafana.com` section and click `Load`
3. Select `ultrafeeder` from the bottom drop down list
4. Click `Import` on the subsequent dialogue box

At this point you should see a very nice dashboard that was created by [Mike](https://github.com/mikenye) \(Thanks!\). The final step is to add the radar plugin required by this dashboard:

1. Hover over the `cog` icon in the lower area of the sidebar, click `Plugins`
2. Enter `radar` into the `Search Grafana plugins` box, at this point `Radar Graph` should appear below
3. Click on `Radar Graph` in the main section
4. Click `Install`

Full functionality of the dashboard is now available, you can find it under `General` in the `Dashboards` section.
