---
description: >-
  If you wish to deploy Grafana for alternative graphs, follow the steps below.
---

# Alternative graphs with Grafana using Prometheus

[`Grafana`](https://grafana.com/) is an analytics platform that can provide alternative graphs for `ultrafeeder`.

In this guide we will be using [`Prometheus`](https://prometheus.io/) as the data repository.

Using Grafana and Prometheus in this configuration does not require a plan, account, or credentials for their respective cloud offerings.

## Create docker volumes

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Add the following lines to the  `volumes:` section at the top of the file \(below the `version:` section, and before the `services:` section\):

```yaml
  prometheus_data:
  grafana_data:
```

This creates the volumes that will contain `prometheus` and `grafana`’s application data.

## Deploying `prometheus` and `grafana` containers

Open the `docker-compose.yml` file that was created when deploying `ultrafeeder`.

Add the following lines to the `environment` section of the `ultrafeeder` container definition \(in the `ultrafeeder:` section, below `environment:` and before the `volumes:` section\):

```yaml
      - ENABLE_PROMETHEUS=true
```

Append the following lines to the end of the file:

```yaml
  prometheus:
    image: prom/prometheus:latest
    tty: true
    container_name: prometheus
    hostname: prometheus
    ports:
      - 9090:9090
    volumes:
      - "prometheus_data:/prometheus"

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

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes and bring up the `prometheus` and `grafana` containers. This will also restart the `ultrafeeder` container, which will now use `telegraf` to feed data to `prometheus`.

At this point we will need to add a collector definition to `prometheus` and restart with the new configuration.

1. Issue the command `docker exec -it prometheus sh -c "echo -e \"  - job_name: 'ultrafeeder'\n    static_configs:\n      - targets: ['ultrafeeder:9273']\" >> /etc/prometheus/prometheus.yml"`
2. Issue the command `docker stop prometheus`
3. Issue the command `docker compose up -d`

You should also be able to point your web browser at:

* `http://docker.host.ip.addr:9090/` to access the `prometheus` console.
* `http://docker.host.ip.addr:3000/` to access the `grafana` console, use admin/admin as initial credentials, you should be prompted to change the password on first login.

Remember to change `docker.host.ip.addr` to the IP address of your docker host.

## Configuring data source and dashboard in Grafana

After you have logged into the `grafana` console the following manual steps are required to connect to `prometheus` as the data source

1. Click `Add your first data source` in the main panel
2. Click `Prometheus` from the list of options provided
3. Input or select the following options, if the option is not listed, do not input anything for that option:

Option | Input
------------- | -------------
Name | ultrafeeder
URL | http://prometheus:9090/

Clicking `Save & Test` should return a green message indicating success. The dashboard can now be imported with the following steps

1. Hover over the `four squares` icon in the sidebar, click `+ Import`
2. Enter `18148` into the `Import via grafana.com` section and click `Load`
3. Select `ultrafeeder` from the bottom drop down list
4. Click `Import` on the subsequent dialogue box

At this point you should see a very nice dashboard, you can find it under `General` in the `Dashboards` section.
