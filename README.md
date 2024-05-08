# ADS-B Reception, Decoding & Sharing with Docker

Automatic Dependent Surveillance-Broadcast \(ADS-B\) is a safety and surveillance technology in which an aircraft determines its position via satellite navigation and periodically broadcasts it, enabling it to be tracked.

These ADS-B data can be received by ~~nerds~~ enthusiasts using Software Defined Radio \(SDR\), and shared with aggregators, which collect, combine, and redistribute flight information for fun and profit.

## Fun and Non-Profit Aggregators

* [ADSBHub](https://www.adsbhub.org)
* [adsb.exposed](https://adsb.exposed/)
* [adsb.fi](https://adsb.fi/)
* [ADSB.lol](https://adsb.lol/)
* [Airplanes.live](https://airplanes.live/)
* [OpenSky Network](https://opensky-network.org/)
* [Planespotters.net](https://www.planespotters.net/)
* [Plane Watch](https://plane.watch/)
* [The Air Traffic](https://theairtraffic.com/)
* [RadarPlane.com](https://radarplane.com/)
* [HPRadar](https://skylink.hpradar.com)
* [Fly Italy ADSB](https://flyitalyadsb.com/)

## For-Profit Aggregators

* [FlightAware](https://flightaware.com/adsb/piaware/)
* [FlightRadar24](https://www.flightradar24.com/share-your-data)
* [Plane Finder](https://planefinder.net)
* [RadarBox](https://www.radarbox.com)
* [radarvirtuel.com](https://www.radarvirtuel.com)
* [AV Delphi](https://avdelphi.com)
* [ADS-B Exchange](https://adsbexchange.com)

## Objective

This guide will walk you through the process to deploy and configure Docker containers to allow reception and decoding of ADS-B data, as well as submission to various flight tracking services, both open and commercial, and the visualisation of this data.

This document is best viewed on GitBook. If you're reading it elsewhere, we humbly suggest going here: [https://sdr-enthusiasts.gitbook.io/ads-b/](https://sdr-enthusiasts.gitbook.io/ads-b/)

This document is intended to be "living". Please feel free to fork the [GitHub repository](https://github.com/sdr-enthusiasts/gitbook-adsb-guide), contribute and submit pull requests! We value your input!

This guide is made available under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).
