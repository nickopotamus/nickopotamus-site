---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Flight tracking using a Raspberry Pi"
subtitle: "Because it's about time I went full dad"
summary: "Because it's about time I went full dad"
authors: []
tags: ["nerdiness", "planes", "pi"]
categories: []
date: 2021-01-04T20:00:24Z
lastmod: 2021-01-04T20:00:24Z
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
The only people cooler than those who hack [Raspberry Pi](https://www.raspberrypi.org/)'s for fun are [plane spotters](https://en.wikipedia.org/wiki/Aircraft_spotting). I grew up with a techy dad who not only helped me build (and repeatedly break) computers and electronics, but also spent his weekends chasing aircraft with a hand-held scanner. So now I'm a dad myself with kids to embarrass (and unable to chase airplanes due to lockdown) I thought: Why not combine the both from the comfort of my own sofa?

I bought myself a shiny new [RPi 4B 8Gb](https://www.raspberrypi.org/blog/8gb-raspberry-pi-4-on-sale-now-at-75/) for Christmas, because I'm really getting into [computer vision on the Pi](https://www.pyimagesearch.com/raspberry-pi-for-computer-vision/) and need the power (and memory). This frees up my old [RPi 3B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) for experimenting with, and given that I live on the flight path for two airports, adding [ADS-B](https://en.wikipedia.org/wiki/Automatic_Dependent_Surveillance%E2%80%93Broadcast) tracking seemed like a neat little project.

## Step 1: Acquire some stuff

* A Raspberry Pi (3B or above will handle the processing better, but it [can be run on a B](https://ianrenton.com/hardware/flight-tracker/) if you've got an old one knocking about). If starting from scratch get it to the point that it's running [Raspian](https://www.raspberrypi.org/software/operating-systems/), passwords set, and connected to the internet.
* A USB RTL-[SDR](https://en.wikipedia.org/wiki/Software-defined_radio) reliever which can be tuned to 1090 MHz. [Realtek RTL2832U](https://www.realtek.com/en/products/communications-network-ics/item/rtl2832u) is a common chipset with the more than capable E4000 tuner, but buyer beware, plenty of these advertised on Amazon etc actually come with a Fitipower FC0012 tuner, which will pick up digital TV but not much else. Read the reviews carefully! I'm currently rocking [this one](https://www.amazon.co.uk/gp/product/B00VZ1AWQA/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1) which came with a telescopic antenna for just shy of £20.
* An antenna. Many USB SDR sticks come with antennas included, though unless they are telescopic are probably better suited to digital TV (but [see this link](https://lucsmall.com/2017/02/06/making-antennas-for-1090mhz-ads-b-aircraft-tracking/) if you wish to convert it with a pair of scissors). 

## Step 2: Setting it up

Unfortunately, it's not _quite_ as simple as plug and play. You'll need to have access to the RPi terminal (either via a keyboard, or [over ssh](https://www.raspberrypi.org/documentation/remote-access/ssh/)). Start by connecting up the USB stick and antenna, plugging it into the Pi, and turning it on. Hopefully everything will light up, but we can't quite find planes yet.

You need a) to decode the signals and b) a way to visualise this. Luckily there are multiple companies out there who provide software to facilitate this. Even better, if you send them your data, in return they give you premium access to all their other data worldwide - so if you want to show off to your equally nerdy friends your ad-free [Flightradar24](https://www.flightradar24.com/) or enterpise [FlightAware](https://uk.flightaware.com/) access this kills two birds with one stone.
 
### FlightAware
[dump1090](https://github.com/flightaware/dump1090) is the key bit of software to decode 1090 MHz signals. The most up to date version is the one provided by FlightAware, so the easiest way to turn your RPi into a fully featured flight tracker is to [install their software bundle](https://flightaware.com/adsb/piaware/install) _PiAware_:

```
wget https://flightaware.com/adsb/piaware/files/packages/pool/piaware/p/piaware-support/piaware-repository_4.0_all.deb
sudo dpkg -i piaware-repository_4.0_all.deb
sudo apt-get update
sudo apt-get install piaware
sudo apt-get install dump1090-fa
sudo apt-get install dump978-fa
sudo reboot
```

If all's gone to plan, your RPi will reboot, and be finding planes! You can check this by running ```piaware-status``` which should say something along the lines of ```dump1090 is producing data on localhost:30005```.

Finally, head to the FlightAware website [to "claim" your reciever](https://flightaware.com/adsb/piaware/claim) and set it up (longitude, latitude, nearest airport, etc) for [MLAT](http://www.multilateration.com/surveillance/multilateration.html), and browse to ```http://[your_RPi_IP_address]:8080``` (find it using ```hostname -I```) to see what you're spotting in real time.


### Flightradar24
[Flightradar24](https://www.flightradar24.com/build-your-own)'s software can then be installed on top of PiAware using their shell script:
```
sudo bash -c "$(wget -O -https://repo-feed.flightradar24.com/install_fr24_rpi.sh)"
```
...however this often fails to work due to PGP errors, so can be bypassed by installing the package directly ([check the website](https://www.flightradar24.com/share-your-data) for the most up to date version):

```
wget https://repo-feed.flightradar24.com/rpi_binaries/fr24feed_1.0.26-9_armhf.deb
sudo dpkg --install fr24feed_1.0.26-9_armhf.deb 
fr24feed --signup
sudo systemctl start fr24feed
```
Ignore any warnings on install about dump1090 not being detected - the signup system will find the FlightAware version just fine. Be prepaed to know your longitude and latitude (find these in your FA setup page) and height above sea level in feet (I used [FreeMapTools](https://www.freemaptools.com/elevation-finder.htm) then added about 9 feet for the height to my office window). Finally [activate your reciever](https://www.flightradar24.com/activate-raspberry-pi) and you're good to go.

### ADS-B Exchange

The final of the "big three" in nerdy plane tracking is [ADS-B Exchange](https://www.adsbexchange.com/faq/) who don't filter any of the info (eg military aircraft) but are also a bit less user friendly! [There's another script](https://www.adsbexchange.com/how-to-feed/#scriptmethod) which installs their software over the top of PiAware:

```
wget -O /tmp/axfeed.sh https://adsbexchange.com/feed.sh
sudo bash /tmp/axfeed.sh
```

[Check that it's working](https://www.adsbexchange.com/myip/), then install the 'stats' package to view your data via the same link:

```
wget -O /tmp/axstats.sh https://adsbexchange.com/stats.sh
sudo bash /tmp/axstats.sh
```

## Step 3: Get your geek on

There's so much geekiness you can do from here. Currently my antenna in the window of my office has a range of 60nm to the south-west, so that's my next thing to work on. There's plenty of external antennas for sale, or you can [build your own out of a coffee tin](https://cromwell-intl.com/open-source/raspberry-pi/ads-b-antenna.html) or even put together a more [high-end self-build complete with lightning protection](https://www.balarad.net/) to mount on your TV arial. You can also consider more fancy (and hence expensive) [USB sticks](https://flightaware.com/adsb/prostick/) which will extend range and improve filtering, or even [hacking together your own server](https://ianrenton.com/hardware/flight-tracker/). The sky is the limit (and I'm not even sorry for that awful pun).


