---
title: "Headless Raspberrypi Setup"
date: 2017-08-08T00:00:47+08:00
draft: false
tags: ["Raspberrypi"]
author: "Peter Yang"
---


[Raspberrypi][1] is a small computer, but it don't have screen.Here is how I configure my PI with a single ethernet cable, after this setup , you can login your PI using ssh from your computer.
# What you need
You will need a Raspberrypi, a SD card (and card reader) , an ethernet cable and a Wifi USB dongle
# Steps
## Flash Respbian on the SD card
you can use [Etcher][2] to burn [Resbian][3] to your card. and after that done , Insert the SD card into your PI,you are ready to rock
## Ssh into your PI with cable
Power on your PI, connect the PI with your router with a normal ethernet cable , and after 1 minute , try execute below command from the computer that connect to the same router


    ssh pi@raspberrypi.local  
    (password:raspberrypi)  



Bingo! your are in RPI now!
## Config wifi
Now let's config the wifi setting , so we can throw the ethernet cable away. in RPI add your wifi information in `/etc/wpa_supplicant/wpa_supplicant.conf`


    network={  
    ssid="your-network-ssid-name"  
    psk="your-network-password"  
    }  



Attach the wifi USB dongle and unplug the ethernet cable, and reboot the RPI(power off and then power on) and wait 1 minute , try `ssh pi@raspberrypi.local`. Now you should be able to login with wifi

[1]: https://www.raspberrypi.org/
[2]: https://etcher.io/
[3]: https://www.raspberrypi.org/downloads/raspbian/
