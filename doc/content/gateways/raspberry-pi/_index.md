---
title: "Bulding a gateway with Raspberry Pi and IC880A"
description: ""
weight: 1
---

This guide can help you build your own LoRaWAN gateway using a Raspberry Pi and an iC880A LoRa concentrator board, and run {{% lbs %}} on it.

<!--more-->

## Requirements

{{< figure src="hardware.png" alt="Required hardware parts" class="float plain" >}}

For building this gateway you will need the following hardware elements:

1. [iC880A-SPI concentrator board](https://shop.imst.de/wireless-modules/lora-products/8/ic880a-spi-lorawan-concentrator-868-mhz)
2. 3.5dBi - 7.5dBi antenna
3. [iC880A pigtail for antenna](https://shop.imst.de/wireless-modules/accessories/20/u.fl-to-sma-pigtail-cable-for-ic880a-spi)
4. Raspberry Pi Model 2 or newer	
5. 2.5A power supply with micro USB connector	
6. MicroSD Card with minimum 4GB of storage
7. 7x dual female jumper wires
8. Ethernet cable or WiFi dongle (if using Raspberry PI 3+ this isn't required, because it has an integrated WiFi interface)

## Gateway Assembly

First, attach the antenna on the iC880A board using the pigtail cable.

Then use jumper cables to connect the iC880A pins to Raspberry Pi pins. Refer to the table below for connections between pins:

|iC880A pin | Raspberry Pi pin | Description|
|--- | --- | ---|
|21 | 2 | 5V power supply|
|22 | 6 | GND|
|13 | 22 | Reset|
|14 | 23 | SPI Clock|
|15 | 21 | MISO|
|16 | 19 | MOSI|
|17 | 24 | NSS|

Your assembled gateway should look like on the image below.

{{< figure src="cables.jpg" alt="Gateway assembly" >}}

{{< warning >}} Do not power up your Raspberry Pi if you haven't connected the antenna to iC880A. If you power it up and the antenna is not connected, all transmit energy that was supposed to be radiated through antenna will be reflected back from an unterminated antenna port and dissipated as heat, and might damage your Raspberry Pi. {{</ warning >}}

## Installing Raspberry Pi OS on Raspberry Pi

In this step, you need to insert your Raspberry PI's SD card into your computer. Check out the official Raspberry Pi documentation for steps to [install the Raspberry Pi 0S](https://www.raspberrypi.com/software/) on the SD card. The most common method to install the Raspberry Pi OS is using the Raspberry Pi Imager.

We also recommend to enable a default SSH access on your Raspberry Pi, in order to avoid connecting it to an external screen for the initial setup. Just mount the boot partition of your Raspberry Pi's SD card and create an empty file called `ssh`. For example, if using Linux:

```bash
touch /media/$USER/boot/ssh
```

Visit the official Raspberry Pi documentation page on detailed guide for [enabling remote access](https://www.raspberrypi.com/documentation/computers/remote-access.html).

## Boot and Configure Your Raspberry Pi

Plug the SD card back into your Raspberry Pi and power it up.

If you haven't enabled SSH access to your Raspberry Pi, you will need to connect an external screen and an external keyboard to it in order to configure it.

If you enabled SSH in the previous step, you can connect to your Raspberry Pi remotely from your computer - find out your Raspberry Pi's IP address by listing devices on your local network (for example with `nmap`) and connect via SSH. The default username is `pi` and the default password is `raspberry`.

```bash
ssh pi@192.168.1.2
```

After logging in in your Raspberry Pi, upgrade the system packages to the latest versions:

```bash
sudo apt-get update
sudo apt-get upgrade
```

Enable the SPI interface by running the `raspi-config` tool:

```bash
sudo raspi-config
```

A `raspi-config` wizard will appear, so use the arrow keys and the Enter key to navigate through it. Choose **Interface Options &#8594; SPI**, then select **<Yes>** to enable the SPI interface. After enabling the SPI interface, hit the Escape key to exit the `raspi-config` tool.

Now install packages needed to build the packet forwarder:

```bash
sudo apt-get install git gcc make
```

## Build the {{% lbs %}} Packet Forwarder

First, clone the {{% tts %}} repository:

```bash
git clone https://github.com/lorabasics/basicstation
```

Then build the {{% lbs %}} binary:

```bash
cd basicstation
make platform=rpi variant=std ARCH=$(gcc --print-multiarch)
```

Make sure the binary was successfully built:

```bash
./build-rpi-std/bin/station --version
```

The binary was successfully built if you see something like this:

```bash
Station: 2.0.6(rpi/std) 2022-03-26 17:43:16
Package: (null)
```

Now install the {{% lbs %}} binary:

```bash
sudo mkdir -p /opt/ttn-station/bin
sudo cp ./build-rpi-std/bin/station /opt/ttn-station/bin/station
```