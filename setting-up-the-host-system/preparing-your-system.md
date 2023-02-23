# Preparing Your System

## Raspberry Pi

Raspberry Pi single-board computers (SBCs) are a popular choice for flight data enthusiasts running this software.

If you're generally unfamiliar with Linux or working with Raspberry Pi, the [official Getting started guide](https://www.raspberrypi.com/documentation/computers/getting-started.html) is a great place to start, and will walk you through the process of installing Raspberry Pi OS - you'll want to enable SSH.

If SSH'ing into a server and editing a text file with nano or vi is something you've done before, you might consider installing [DietPi](https://dietpi.com/docs/install/).

## Hints for Linux Newbies

Collecting ADS-B data with an SDR is a great project to learn about Linux. While this guide isn't a Linux crash course, it's worth pointing out a few basic concepts to ease into things.

### Entering Commands

Throughout this guide, you'll be presented with commands to copy/paste into your new device. If you're accessing your new Linux device directly with a keyboard, mouse, and display like a traditional desktop PC, you'll be entering these via the "terminal," "command line," or "command prompt." If you're using something with a fancy graphical user interface, such as Raspberry Pi OS with a mouse, keyboard, and display plugged in like a traditional desktop computer, it's likely there's a "Terminal" app you can access with a few clicks.

### Remote Access: SSH

Once you've connected your Linux device to your network, it's possible to access and manage it with SSH. On macOS, you can do this via the Terminal app, on Windows, you can use [PuTTY](https://www.putty.org/). You can connect to your new device via SSH entering this command `ssh username@new.device.ip.addresss` in Terminal/PuTTY.
