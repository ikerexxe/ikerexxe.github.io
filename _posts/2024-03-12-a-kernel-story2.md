---
layout:     post
title:      "A Kernel story II: Compositing and mode setting"
date:       2024-03-12 12:00:00 +0100
categories: kernel
---

# Introduction

As mentioned in the [first post of this series](/kernel/2024/01/27/a-kernel-story1.html), DRM is a subsystem of the Linux kernel responsible for interfacing with GPUs of modern video cards. It exposes an API that user-space programs can use to send commands and data to the GPU and perform operations such as configuring the mode setting of the display and hardware-accelerated 3D rendering.

This post will focus on  DRM's KMS subsystem.

# Kernel Mode Setting (KMS)

KMS<sup>[1](#references)</sup> is a component of the DRM subsystem that is responsible for setting the display mode<sup>[2](#references)</sup> and configuration for the graphics hardware. It is used to initialize the graphics hardware at boot time and to change the display configuration dynamically as needed.

KMS is composed of the following data structures:
* Framebuffers<sup>[3](#references)</sup> are a memory buffer containing data representing all the pixels in an on-screen image, plus information about the image's color format and size. The framebuffer **stores the compositor’s (i.e. Wayland) image**.
* Planes<sup>[4](#references)</sup> represent an image source that **specifies the position, orientation, and scaling factors**. They are exposed by the hardware and exposed by the Kernel driver.
* Cathode-ray tube controller (CRTC)<sup>[5](#references)</sup> represent the overall display pipeline. It fetches pixel data from one or more planes (or even no), **blends overlapping planes** where necessary, and forwards the result to its outputs. CRTC are created and set by the driver.
* Encoders<sup>[6](#references)</sup> are the hardware components that **encode pixel data into the physical signal understood by the connector**. They are created and set by the driver. Encoders are associated with a specific connector.
* Connectors<sup>[7](#references)</sup> represent the **physical connection to an output device**, such as HDMI or VGA ports with a connected monitor. The connector also provides information on the output device's supported display modes, physical resolution, color space, and the like. Connectors are created and set by the driver.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2024-03-12-KMS_Display_Pipeline_Overview.svg){: .img-fluid}
| *Source: https://lwn.net/Articles/955708/* |
</div>

# Conclusion

This is the most basic stuff needed to understand KMS and start working with the modern Linux Kernel graphic stack. If you want to know more about KMS I recommend you to read the *The Linux graphics stack in a nutshell, part 2* <sup>[8](#references)</sup> written by Thomas Zimmermann. It is very good! By the way, there's no need to read part 1 to understand part 2.

# Next

[A Kernel story III: Presenting the hardware](/kernel/2024/04/24/a-kernel-story3.html)

# References

1. [KMS](https://wiki.archlinux.org/title/kernel_mode_setting)
2. [Mode setting](https://en.wikipedia.org/wiki/Mode_setting)
3. [Framebuffer](https://docs.kernel.org/gpu/drm-kms.html#frame-buffer-abstraction)
4. [Plane](https://docs.kernel.org/gpu/drm-kms.html#plane-abstraction)
5. [CRTC](https://docs.kernel.org/gpu/drm-kms.html#crtc-abstraction)
6. [Encode](https://docs.kernel.org/gpu/drm-kms.html#encoder-abstraction)
7. [Connector](https://docs.kernel.org/gpu/drm-kms.html#connector-abstraction)
8. [The Linux graphics stack in a nutshell, part 2)](https://lwn.net/Articles/955708/)