---
layout:     post
title:      "A Kernel story VIII: Document how to set CS active-high"
date:       2024-12-18 10:00:00 +0100
categories: kernel
background: '/assets/figures/2024-12-18-Document-how-to-set-CS-active-high.jpg'
---

In my quest to [set SPI CS active-high on a Raspberry Pi with the Linux Kernel](/kernel/2024/11/12/a-kernel-story7.html), I encountered several challenges due to the lack of clear documentation. To address this, I've contributed improvements to the existing documentation and provided practical examples.

* A new overlay example in the Raspberry Pi repository demonstrating how to invert the CS signal using `cs-gpio` with `GPIO_ACTIVE_HIGH` in the SPI controller and `spi-cs-high` in the peripheral property.
* Linux Kernel documentation updates explaining the above method and offering a simple devicetree example.

Find the details here:

* [Raspberry Pi repository PR](https://github.com/raspberrypi/linux/pull/6477)
* [Linux Kernel documentation changes](https://lkml.org/lkml/2024/12/16/476)

I'm grateful for the assistance I received from the community. By contributing these improvements, I hope to give back and help others facing similar challenges.
