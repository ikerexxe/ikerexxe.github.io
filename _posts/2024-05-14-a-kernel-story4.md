---
layout:     post
title:      "A Kernel story IV: Build and install the kernel"
date:       2024-05-14 11:00:00 +0100
categories: kernel
background: '/assets/figures/2024-05-14-Build_and_install_the_kernel.png'
---

# Introduction

This post is more practical as it aims to install a Linux kernel that we have built in advance. This kernel should boot and work without any problems.

As mentioned in the previous post the platform for which we are going to build it is the Raspberry Pi 4, as it is the one I have. The instructions should be similar for other boards with ARM architecture.

# Instructions

First of all let's install the build dependencies, clone the [kernel-ark](https://gitlab.com/cki-project/kernel-ark) project and create a branch. This project contains the kernel source tree for the Fedora distribution.

```console
dnf install ccache gcc-aarch64-linux-gnu
git clone https://gitlab.com/cki-project/kernel-ark.git
git checkout -b os-build
cd kernel-ark
```

As mentioned above, we are going to build the kernel for the Raspberry Pi 4, so we need the configuration file for this board. The best way to get it is to copy the existing one from the Fedora distribution that is already installed.

```console
scp /lib/modules/$(uname -r)/config USERNAME@IP:/FOLDER/.config
```

Note: Replace capitalised letters with actual values.

The next step is to prepare the configuration for cross-compilation.

```console
export ARCH=arm64 CROSS_COMPILE="ccache aarch64-linux-gnu-"
```

Now we will build and create the kernel image.

```console
make prepare modules_prepare && make -j16 Image modules
```

Create temporary directory and store the artifacts from the modules and kernel compilation in it.

```console
mkdir /tmp/kernel
make modules_install INSTALL_MOD_PATH=/tmp/kernel/
kver=6.9.0-rc5+
cp arch/arm64/boot/Image /tmp/kernel/lib/modules/$kver/vmlinuz
```

Note: `kver` needs to be set to the compiled kernel version. In my case this is `6.9.0-rc5+`.

Compress the image and copy it to the Raspberry Pi.

```console
cd /tmp/kernel/lib/modules/
tar cfz kernel.tar.gz $kver
scp kernel.tar.gz root@$IP:~
ssh USERNAME@$IP
```

Note: Replace capitalised letters with actual values.

Let's install the kernel in the Raspberry Pi and reboot.

```console
sudo su -
cp /home/USERNAME/kernel.tar.gz /lib/modules/
cd /lib/modules/ && tar -zxvf kernel.tar.gz
kver=6.9.0-rc5+
kernel-install add $kver /lib/modules/$kver/vmlinuz
reboot
```

Finally, let's check that the new kernel is correctly installed.

```console
ssh USERNAME@$IP
uname -r
```

# Next

[A Kernel story V: Playing with the display from user-space](/kernel/2024/06/07/a-kernel-story5)