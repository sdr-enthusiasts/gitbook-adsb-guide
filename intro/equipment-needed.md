# Equipment Needed

To get started, you'll need:

### **A SDR that can receive 1090MHz**

I started with a [FlightAware Pro Stick Plus](https://www.amazon.com/dp/B07J2RJQ9Z/ref=cm_sw_em_r_mt_dp_U_uwltEbJ8ER2KN). However, eventually you may wish to get into other areas of SDR. I have moved onto a [KerberosSDR](http://kerberossdr.com/), which is four RTL-SDRs in one. This lets me dedicate one SDR to ADS-B reception, and three others for things like [AirBand](https://en.wikipedia.org/wiki/Airband), [ACARS](https://app.airframes.io), etc.

If you're just getting started and don't want to spend a lot of cash, a [$20 USB DVB-T RTL2832U dongle](https://www.aliexpress.com/item/32259584047.html) will do the job.

### An antenna optimised for 1090MHz

I use an eBay version of [this](https://www.amazon.com/dp/B00WZL6WPO/ref=cm_sw_em_r_mt_dp_U_CxltEb9JS155W). You could also [make your own](https://discussions.flightaware.com/t/three-easy-diy-antennas-for-beginners/16348).

### A computer running Linux

Capable of running Docker, with at least one free USB port. This can be a Raspberry Pi or an x86.

### Cables

USB cable to connect the SDR to the computer. I use a 20m active USB cable \(similar to [this](https://www.amazon.com/BlueRigger-Female-Active-Extension-Repeater/dp/B005LJKEXS/ref=sr_1_4?keywords=active+usb+cable&qid=1582085965&sr=8-4)\) which runs from the Linux computer in my study up into my roof void, where the SDR and antenna are located.

Coaxial cable to connect the antenna to the SDR. As my SDR is located just below my antenna, I use a "pigtail" similar to [this](https://www.amazon.com/DZS-Elec-Connecting-Coaxial-Extender/dp/B072C6CJBC/ref=sr_1_13?keywords=SMA+to+N+male&qid=1582086024&sr=8-13).

### Other Stuff

There's a whole bunch of additional equipment that you could purchase and use, such as 1090MHz bandpass filters \(recommended\), amplifiers, etc etc. This is somewhat outside the scope of this document. If you want more information, I'd refer you to: [https://flightaware.com/adsb/piaware/build](https://flightaware.com/adsb/piaware/build)

