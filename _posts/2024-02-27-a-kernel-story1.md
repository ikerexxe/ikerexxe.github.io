---
layout:     post
title:      "A Kernel story I: fbdev and DRM"
date:       2024-01-27 12:00:00 +0100
categories: kernel
---

# Introduction

I am starting this post with the idea of doing a series on Linux Kernel driver development. The Linux Kernel is a project that has fascinated me for years. As you may know, I love low-level software development, and this project is the pinnacle of this world. Unfortunately, between one thing and another, I have never done anything serious on it, but this is something I want to change. This is why I am now going to invest some time working on this topic and will try to summarize what I learn in this series of blog posts.

For a full immersion in this topic, the idea is to port a display driver from fbdev to DRM. So in this first post I'll explain those two terms and what is the difference between them.

Before getting down to business, I would like to mention here [Javier Martinez Canillas](https://github.com/martinezjavier) for all the help and mentoring he is giving me.

# The Linux framebuffer (fbdev)

fbdev<sup>[1](#references)</sup> is a linux subsystem used to show graphics on a computer monitor, typically on the system console. It was designed as a hardware-independent API to give user space software access to the framebuffer using only the Linux kernel's own basic facilities and its device file system interface. Avoiding the need for external libraries which effectively implemented video drivers in user space.

# Direct Rendering Manager (DRM)

DRM<sup>[2](#references)</sup> is a subsystem of the Linux kernel responsible for interfacing with GPUs of modern video cards. DRM exposes an API that user-space programs can use to send commands and data to the GPU and perform operations such as configuring the mode setting of the display and hardware-accelerated 3D rendering.

# fbdev vs DRM

fbdev can’t be used to handle the needs of modern 3D-accelerated GPU-based video hardware. These devices usually require setting and managing a command queue in their own memory to dispatch commands to the GPU and also require management of buffers and free space within that memory. User-space programs directly managed these resources and they acted as if they were the only ones with access to them. This used to lead to two or more programs to compete for this resource, which usually ended catastrophically.

DRM was initially created to allow multiple programs to use video hardware resources cooperatively. DRM gets exclusive access to the GPU and is responsible for initializing and maintaining the command queue, memory, and any other hardware resource. Applications that want to use the GPU communicate with DRM, which acts as an arbitrator and takes care to avoid possible conflicts.

Since then DRM has been expanded to include additional functionality, such as dynamic display configuration changes and multiple GPU management.

# References

1. [fbdev](https://en.wikipedia.org/wiki/Linux_framebuffer)
2. [DRM](https://en.wikipedia.org/wiki/Direct_Rendering_Manager)