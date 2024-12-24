---
layout:     post
title:      "A Kernel story III: Presenting the hardware"
date:       2024-04-24 20:00:00 +0100
categories: kernel
---

# Introduction

As I mentioned in the first post of this series the intention is to port an fbdev driver to DRM. In this article I will present the hardware I will use for that purpose.

# Raspberry Pi 4

The [Raspberry Pi](https://en.wikipedia.org/wiki/Raspberry_Pi) is a small board, the first version of which was introduced in 2012. This board is very well known and well supported. I was lucky enough to buy one two years ago while there was a shortage, so I will use it for development purposes.

The next diagram shows the pinout for the board. I'll use it later on to show how to connect everything.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2024-04-24-Presenting_the_hardware_1.png){: .img-fluid}
| *Source: [https://www.raspberrypi.com/documentation/computers/raspberry-pi.html](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html)* |
</div>

# AZDelivery 12864

The [AZDelivery 12864](https://www.az-delivery.de/en/products/128x64-lcd-blaues-display) is a monochromatic LCD Display Module. It has a resolution of 128x64 and it's very easy to find and buy. The controller is compatible with KS0108.

The display diagram:

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2024-04-24-Presenting_the_hardware_2.png){: .img-fluid}
</div>

# How to connect it

Below is the table explaining how to connect the PINs to each other.

| Raspberry Pi 4    | AZDelivery 12864 |
| ----------------- | ---------------- |
| 6 (Ground)        | 1 (GND)          |
| 4 (5V power)      | 2 (VCC)          |
| 4 (5V power)      | 3 (V0)           |
| 24 (GPIO 8 CE0)   | 4 (RS)           |
| 19 (GPIO 10 MOSI) | 5 (R/W)          |
| 23 (GPIO 11 SCLK) | 6 (E)            |
| 6 (Ground)        | 15 (PSB)         |
| 4 (5V power)      | 19 (BLA)         |
| 6 (Ground)        | 20 (BLK)         |

These are some picture of the final result:

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2024-04-24-Presenting_the_hardware_3.png){: .img-fluid}
</div>

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2024-04-24-Presenting_the_hardware_4.png){: .img-fluid}
</div>

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2024-04-24-Presenting_the_hardware_5.png){: .img-fluid}
</div>

# Next

[A Kernel story IV: Build and install the kernel](/kernel/2024/05/14/a-kernel-story4)