---
layout:     post
title:      "A Kernel story V: Playing with the display from user-space"
date:       2024-06-07 10:00:00 +0100
categories: kernel
---

# Introduction

The intention of this post is to start up the display and check that it works correctly. To do this I will use a project in user-space because it will make everything faster.

| ![Github Actions](/assets/figures/2024-06-07-Playing_with_the_display_1.jpg) | 
|:--:| 
| *Generated by DALL·E 3* |

# User-space projects

## pyLCD

The first project I tried was [pyLCD](https://github.com/Mezgrman/pyLCD). It looked like a very promising project as it implemented everything I needed to control the display. I just had to install the project, install the dependencies, configure the PINs and run it.

Unfortunately, the installation of the project isn't as easy as it looks. As you can see in the git history the project is quite old and the pip installation doesn't work. So, I was [recommended by the author](https://github.com/CatoLynx/pyLCD/issues/13#issuecomment-2063918499) to clone the project and run it from there.

The next step is to install the dependencies. And here I ran into new problems. pyLCD depends on [WiringPi](https://github.com/WiringPi/WiringPi.git), which uses the old `SYSFS` interface to work (the use of the new one is under development at the time of writing this blog post). This interface is disabled by default in Fedora, so I had to enable `CONFIG_GPIO_SYSFS` in the kernel, build it and install it. This seemed to work as I was able to run the commands provided by WiringPi to check the GPIOs.

[WiringPi2-Python](https://github.com/Gadgetoid/WiringPi2-Python)  provides a python interface for the WiringPi project, so you also have to install it if you want to use it from pyLCD. But I am unable to get it to work. Using python 3.12 reports that `imp` is missing, which is no longer supported for this version of python. So you need to use python3.11 in a virtual environment, reinstall all the dependencies in this environment and try to import `wiringpi2`, but it fails with an error stating `_wiringpi2` is missing. From what I understand this module is provided by this project, so I really don't see why it fails.

As you can see using pyLCD led me to use several projects that were deprecated or abandoned and even after several hours of investigation I was unable to make it work, so I decided to abandon this path and try a new one.

## KS0108_RPI_Driver

The second project I tried was [KS0108_RPI_Driver](https://github.com/leonyuhanov/KS0108_RPI_Driver). Again, I just had to install the dependencies, configure it, compile it and run it.

This project depends on the [BCM C++ driver library](http://www.airspayce.com/mikem/bcm2835/). You can follow the instructions in the [README](https://github.com/leonyuhanov/KS0108_RPI_Driver/blob/main/README.md) to compile and install it.

Now, it is time to configure the PINOUT, and this is where I understood that something was wrong. This project uses an SPI interface to communicate with the display, but the `AZDelivery 12864` display does not offer this interface. I could add an expansion board to make it work, but it is not worth it because the display is sold as is and offers both a parallel and a serial interface.

At that point I decided to check other projects.

## Other projects

I have investigated other projects and they either use the parallel port, or an SPI interface with an expansion board. This seems wrong because as it’s more efficient to minimize the number of GPIOs when using the Linux Kernel and let the Kernel manage communication with the serial interface. Besides, the display already includes a serial interface, so why use an expansion board with SPI?!

| ![Github Actions](/assets/figures/2024-06-07-Playing_with_the_display_2.jpg) | 
|:--:| 
| *Generated by DALL·E 3* |

# Conclusion

As you can see, I've poured countless hours into researching how to make this work from user-space, only to hit an insurmountable wall of frustration. After a lengthy conversation with my mentor, I've decided to take a different approach. I'm going to test the serial interface using a small microcontroller. Maybe, this will shed some light on whether the `AZDelivery 12864` serial interface actually functions. And perhaps, in the process, I'll unravel the mysteries of the communication protocol.