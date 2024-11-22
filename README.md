# ADS-B MicroOS Config Generator

This web app allows you to configure a simple ADS-B receiver based on openSUSE MicroOS
and openSUSE Tumbleweed-based containers.  
Heavily based on openSUSE's [Fuel Ignition](https://opensuse.github.io/fuel-ignition/).

To access the web app, open this link: https://g7.github.io/adsbreceiver/

## Current status

* Able to feed to various feeds using the Beast protocol (Airplanes.live, ADSB.fi, ADSB.lol, ADSBExchange.com, Flyitalyadsb.com, Planespotters.net)
* Able to feed to Flightradar24 (initial-setup available but NOT tested! I've only tested using a key I already had)
* Local web interface ([tar1090](https://github.com/wiedehopf/tar1090)) to easily visualize the received data
* Cockpit pre-configured to ease maintenance

[`readsb`](https://github.com/wiedehopf/readsb) is the dumper/forwarder used.

## What is needed

* One RTL-SDR capable ADS-B receiver (RTL2832U DVB-T dongles might work)
* One Raspberry Pi or other SBC supported by openSUSE MicroOS
* One microSD card where to install openSUSE MicroOS
* One USB drive to use for the initial ignition+combustion bootstrap
* Working, wired internet connection

Open the [Web app](https://g7.github.io/adsbreceiver/) for further instructions

## What's missing

* MLAT (Multilateration) support. I've packaged mlat-client already, but I have to wire it up
* FlightAware support

## Acknowledgments

I just put all of these things together as a fun [SUSE Hack Week 24](https://hackweek.opensuse.org/24/projects/ads-b-receiver-with-microos) project.

If you want to have something more ready and battle-tested, consider using these instead:
* [ADSB.im](https://www.adsb.im/home)
* [SDR-Enthusiasts Docker guide](https://sdr-enthusiasts.gitbook.io/ads-b)

Thanks a lot to everyone who contributed to dump1090/readsb, tar1090, mlat-client
and the tar1090-db database!

Nevertheless, I will continue using MicroOS + my containers on my Raspberry Pi, and
will try my best to keep all the tools up-to-date :)
