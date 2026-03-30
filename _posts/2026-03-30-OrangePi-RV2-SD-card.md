---
layout:     post
title:      "Implementing SD card support for the OrangePi RV2"
date:       2026-03-30 09:00:00 +0200
categories: kernel
---

# Introduction

Following my work on the Linux Kernel, I have started working on developing drivers for the **Orange Pi RV2**. This board is based on the **SpacemiT K1** SoC, a powerful 8-core RISC-V processor.

The Orange Pi RV2 is a very interesting piece of hardware for the RISC-V ecosystem. It is small but powerful, featuring up to 8GB of RAM, Gigabit Ethernet, and USB 3.0. However, to make it truly useful with a mainline kernel, we need support for its primary storage: the SD card.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2026-03-30-OrangePi-RV2-SD-card-1.png){: .img-fluid}
| *OrangePi RV2* |
</div>

# Developing the driver

I've spent time implementing the support for the SD host interface for the mainline kernel. This wasn't just about porting a vendor driver, it was about creating a clean, upstream-compliant implementation that allows the OrangePi RV2 to finally utilize its primary expandable storage slot.

A major part of this work was enabling **UHS-I (SDR104)** support. This allows the SD card to run at a high frequency of **208 MHz**. With this high-speed support, the board feels much faster. You will notice that applications open quickly and large files move without any lag.

<div style="text-align: center;" markdown="1">
![Github Actions](/assets/figures/2026-03-30-OrangePi-RV2-SD-card-2.png){: .img-fluid}
| *Enabling high-speed SD support on OrangePi RV2* |
</div>

# Enabling support for other boards

The SpacemiT K1 SoC is the core of several other boards, such as the **Banana Pi BPI-F3** and the **Muse Pi Pro**. Since they share the same internal hardware, they can all benefit from this driver.

If you are working on a different board using the K1 SoC, you can enable SD card support by updating your Device Tree (`.dts`) file. You will need to map the correct pins and link your power regulators.

Below is an example of the configuration I used for the Orange Pi RV2:

```dts
&sdhci0 {
    /* Pin configuration and bus width */
    pinctrl-names = "default", "state_uhs";
    pinctrl-0 = <&mmc1_cfg>;
    pinctrl-1 = <&mmc1_uhs_cfg>;
    bus-width = <4>;

    /* Card detection settings */
    cd-gpios = <&gpio K1_GPIO(80) GPIO_ACTIVE_HIGH>;
    cd-inverted;

    /* Device capabilities and limits */
    no-mmc;                       /* Only look for SD cards, not eMMC */
    no-sdio;                      /* No SDIO support needed */
    disable-wp;                   /* No write-protect pin on this slot */
    cap-sd-highspeed;             /* Enable basic high-speed support */

    /* Power regulators from the PMIC */
    vmmc-supply = <&buck4>;
    vqmmc-supply = <&aldo1>;

    /* High-speed capabilities */
    sd-uhs-sdr25;
    sd-uhs-sdr50;
    sd-uhs-sdr104;

    status = "okay";
};
```

# Conclusion

The patches for this driver are currently under review in the [Linux kernel mailing lists](https://lore.kernel.org/all/20260330-orangepi-sd-card-uhs-v5-0-bd853604322d@gmail.com/). As is common with mainline development, I expect some additional changes and feedback from other maintainers to make the code even better.

Once these changes are merged, the SpacemiT K1 will be one step closer to having full mainline support, making these RISC-V boards much easier for everyone to use.
