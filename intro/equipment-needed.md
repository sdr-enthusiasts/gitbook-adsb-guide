# Equipment Needed

To get started, you'll need:

## **A SDR that can receive 1090MHz**

I started with a [FlightAware Pro Stick Plus](https://www.amazon.com/dp/B07J2RJQ9Z/ref=cm_sw_em_r_mt_dp_U_uwltEbJ8ER2KN). However, eventually you may wish to expand into other areas of SDR. I have moved onto a [KerberosSDR](https://othernet.is/products/kerberossdr-4x-coherent-rtlsdr), which is four RTL-SDRs in one. This lets me dedicate one SDR to ADS-B reception, and three others for things like [AirBand](https://en.wikipedia.org/wiki/Airband), [ACARS](https://app.airframes.io), etc. Note that KerberosSDR has reached EOL and it has been replaced by the [KrakenSDR](https://www.crowdsupply.com/krakenrf/krakensdr) outfitted with 5 SDRs.

If you're just getting started and don't want to spend a lot of cash, a [cheap DVB-T RTL2832U WITH R820T2 dongle](https://www.amazon.com/dp/B07K47P7XD) will do the job.

## An antenna optimised for 1090MHz

I use an eBay version of [this](https://www.amazon.com/dp/B00WZL6WPO/ref=cm_sw_em_r_mt_dp_U_CxltEb9JS155W). You could also [make your own](https://discussions.flightaware.com/t/three-easy-diy-antennas-for-beginners/16348).

## A computer running Linux

Capable of running Docker, with at least one free USB port. This can be a Raspberry Pi or an x86.

## Cables

USB cable to connect the SDR to the computer. I use a 20m active USB cable \(similar to [this](https://www.amazon.com/BlueRigger-Female-Active-Extension-Repeater/dp/B005LJKEXS/ref=sr_1_4?keywords=active+usb+cable&qid=1582085965&sr=8-4)\) which runs from the Linux computer in my study up into my roof void, where the SDR and antenna are located.

Coaxial cable to connect the antenna to the SDR. As my SDR is located just below my antenna, I use a "pigtail" similar to [this](https://www.amazon.com/DZS-Elec-Connecting-Coaxial-Extender/dp/B072C6CJBC/ref=sr_1_13?keywords=SMA+to+N+male&qid=1582086024&sr=8-13).

## Other Stuff

There's a whole bunch of additional equipment that you could purchase and use, such as 1090MHz bandpass filters \(recommended\), amplifiers, etc etc. This is somewhat outside the scope of this document. If you want more information, we'd recommend you read this guide: [https://flightaware.com/adsb/piaware/build](https://flightaware.com/adsb/piaware/build)

## **Optional: A SDR that can receive 978MHz (and antenna)**

**Note:** USA-only

In the USA, the FAA operates a supplemental data service on 978MHz aimed at the General Aviation community, called Universal Access Transmitter/Transceiver (UAT). This service is similar to ADS-B on 1090MHz, but provides additional services beyond position reporting. GA aircraft are able to broadcast their position data just like aircraft utilizing ADS-B on 1090ES. However, UAT ground stations that receive position reports from airborne aircraft respond with data packages containing all other traffic located within 15 nm and 3,500 vertical feet of the reporting aircraft. All UAT receivers within range are able to process the position reports directly from reporting aircraft as well as the custom data packages (called "pucks" due to the shape of the included airspace) that are transmitted by ADS-B/UAT ground stations, regardless of the intended recipient.

This is particularly helpful to the hobbyist community as there are still thousands of airborne aircraft whose positions are only known to the National Airspace System (NAS). Multilateration (MLAT) is an attempt at recreating the positions of that hidden traffic. These traffic packages being broadcast by the ground stations expose the precise locations of the aircraft that would normally be estimated via MLAT or completely hidden in areas not covered by enough hobbyist receivers. It only takes one receiver near a tower to cover hundreds of square miles of airspace.

UAT ground stations also broadcast NEXRAD weather radar and METARs to all UAT receivers, although this data may not be particularly useful to the average hobbyist.

It should be noted that pucks are only generated when aircraft of any type is located within 15 nm and 3,500 vertical feet of GA aircraft that are reporting their position via UAT. Receiving UAT services from a tower does not guarantee "seeing" every single aircraft in your area. Also, the signals being broadcast by ADS-B ground stations typically only reach 5 nm from the tower at ground level. Therefore, most UAT receivers will only receive position data from nearby airborne GA aircraft unless they are themselves airborne or located especially close to a ground station.
