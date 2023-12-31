---
description: 'If you wish to feed ADS-B Exchange, follow the steps below.'
---

# Feeding ADS-B Exchange

[ADSBExchange.com](https://adsbexchange.com/) collects data from over 10,000 receivers located around the world, aggregating the data onto a real-time web-based display for enthusiasts, and raw-data products for professional and commercial usage. It has recently been sold to a private firm called JETNET, for more information, please see:

* <https://www.rtl-sdr.com/ads-b-exchange-acquired-by-private-firm-jetnet/>
* <https://www.jetnet.com/news/jetnet-acquires-ads-b-exchange.html>


## Update `ultrafeeder` container configuration

Before running `docker compose`, we also want to update the configuration of the `ultrafeeder` container, so that it generates MLAT data for adsbexchange.

Open the `docker-compose.yml` and make the following environment value is part of the `ULTRAFEEDER_CONFIG` variable to the `ultrafeeder` service:

```yaml
      - ULTRAFEEDER_CONFIG=
          adsb,feed1.adsbexchange.com,30004,beast_reduce_plus_out;
          mlat,feed.adsbexchange.com,31090,39005;
```


## Refresh running containers

Once the file has been updated, issue the command `docker compose up -d` in the application directory to apply the changes to the `ultrafeeder` container.

After a few minutes, point your browser at [https://adsbexchange.com/myip/](https://adsbexchange.com/myip/). You should see green smiley faces indicating that you are successfully sending data.
Also check if your MLAT is synchronized: <https://map.adsbexchange.com/mlat-map/>
