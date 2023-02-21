# ADS-B Reception, Decoding & Sharing with Docker

Automatic Dependent Surveillance-Broadcast \(ADS-B\) is a surveillance technology in which an aircraft determines its position via satellite navigation and periodically broadcasts it, enabling it to be tracked.

This ADS-B data can be received by ~~nerds~~ enthusiasts using Software Defined Radio \(SDR\), and used for fun and profit. For example:

**Fun:**

* [ADSBHub](https://www.adsbhub.org)
* [OpenSky Network](https://opensky-network.org)
* [Plane Watch](https://plane.watch/)
* [The Air Traffic](https://theairtraffic.com/)
* [https://twinfan.gitbook.io/livetraffic/](https://twinfan.gitbook.io/livetraffic/)
* "New" aggregator services such as [adsb.fi](https://globe.adsb.fi/), [adsb.one](https://adsb.one/), and [adsb.lol](https://adsb.lol/)

**Profit:**

* [https://flightaware.com/adsb/piaware/](https://flightaware.com/adsb/piaware/)
* [https://www.flightradar24.com/share-your-data](https://www.flightradar24.com/share-your-data)
* [https://planefinder.net](https://planefinder.net)
* [https://www.radarbox.com](https://www.radarbox.com)
* [https://radarvirtuel.com](https://radarvirtuel.com)
* [ADS-B Exchange](https://adsbexchange.com/)

This guide will walk you through the process to deploy and configure Docker containers to allow reception and decoding of ADS-B data, as well as submission to various flight tracking services, both open and commercial, and the visualisation of this data.

This document is best viewed on GitBook. If you're reading it elsewhere, we humbly suggest going here: [https://sdr-enthusiasts.gitbook.io/ads-b/](https://sdr-enthusiasts.gitbook.io/ads-b/)

This document is intended to be "living". Please feel free to fork the [GitHub repository](https://github.com/sdr-enthusiasts/gitbook-adsb-guide), contribute and submit pull requests! We value your input!

This guide is made available under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).
