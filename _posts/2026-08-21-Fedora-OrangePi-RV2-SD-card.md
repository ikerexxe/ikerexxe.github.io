---
layout:     post
title:      "Booting Fedora on the OrangePi RV2 from an SD card"
date:       2026-08-21 09:00:00 +0200
categories: test
background: '/assets/figures/2026-08-21-Fedora-OrangePi-RV2-SD-card.png'
---

After my [SD card driver for the OrangePi RV2 merged](/kernel/2026/03/30/OrangePi-RV2-SD-card) into a new Linux Kernel release, I wanted to boot Fedora on the device to test it out. I tried both `Fedora 44 Omni` and `Fedora 44 Generic`. Unfortunately, the Generic image still uses kernel v6.19, which doesn't include my driver changes yet.

# Media preparation & installation

You can follow the standard [Fedora RISC-V media preparation guide](https://fedoraproject.org/wiki/Architectures/RISC-V/Installing#Media_preparation) to prepare your SD card. Since the driver is already in the mainline kernel, Fedora 44 Omni works out-of-the-box. You don't need to install a vendor-customized SpacemiT image.

# Troubleshooting: Device Tree loading failure

If you are using an old vendor version of U-Boot, you might see this error during boot:

```console
Failure: Failed to load 'dtb/spacemit/x1_orangepi-rv2.dtb'
```

This happens because the older U-Boot version tries to load the old Device Tree name (`x1_orangepi-rv2.dtb`) instead of the updated one (`k1-orangepi-rv2.dtb`). You can fix this by mounting your SD card boot partition and creating a symbolic link pointing the old name to the new file:

```console
cd $(MOUNT_POINT)/dtb-7.1.5-201.0.riscv64.omni.fc44.riscv64/spacemit/
sudo ln -s k1-orangepi-rv2.dtb x1_orangepi-rv2.dtb
```

There is a [patch set](https://lore.kernel.org/all/20260819-m4-v9-0-6e66016392b9@riscstar.com/) that adds SD card support for SpacemiT K1 boards to mainline U-Boot, but it still needs to be merged. However, that patch set doesn't include the OrangePi RV2 Device Tree yet. Hopefully someone will take care of adding it soon.

# Logging in

Once you power on the board, you will see the boot messages scrolling on your screen. After it finishes booting, you will arrive at the login prompt where you can enter the default credentials:

* Username: fedora
* Password: linux
