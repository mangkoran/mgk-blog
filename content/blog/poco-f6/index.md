+++
title = "My POCO F6 Setup"
date = 2024-11-08
draft = true
[taxonomies]
tags = ["android", "root"]
+++

## Finally, a new phone

I've been eyeing POCO F6 ever since its release in China as Redmi Turbo 3. The
performance is really good considering its price which is all I care about. So
after saving my first paychecks, I decided it's a good time to upgrade my phone
from Redmi Note 10 Pro. I bought F6 for IDR 4.9m (~USD 309 as of 2024-12-09)
which is a steal for last-gen flagship performance and 12/512GB.

## Stock? In this economy?

My first thoughts after receiving the phone is how do I root this phone. Firstly
we need to unlock the bootloader. For Xiaomi phones we need to request for the
bootloader unlock through [Mi
Community](https://play.google.com/store/apps/details?id=com.mi.global.bbs) app.
Navigate to "Me" > "Unlock bootloader". Make sure your Mi account is over 30
days old.

<!--TODO: add mi app screenshot-->

After we have been granted the permission, we can proceed to unlock the phone
using [Mi Unlock](https://en.miui.com/unlock/index.html) app.

<!--TODO: flash xiaomi.eu-->

Now the bootloader has been unlocked, we can proceed to flash xiaomi.eu ROM.
xiaomi.eu ROM is based on Xiaomi CN ROM with all ads have been removed. To flash
the ROM, we need to put the phone in Fastboot mode. To have easy access to
Fastboot we can enable "Extended power menu" in Developer Option.

<!--TODO: boot + install KernelSU-->

For the root method we are going to use KernelSU. KernelSU is a new root
approach which works in kernel mode. Basically it should be better at root
hiding.

<!-- 1. Install KernelSU app -->
<!-- 2. Boot to Fastboot -->
<!-- 3. fastboot boot kernelsu -->
<!-- 4. KernelSU app > Install (top right, arrow down box) > Direct install > Reboot -->

## Current setup

- ROM: HyperOS 2 by xiaomi.eu
- Root: KernelSU
- Modules
  - Zygisk Next
  - Shamiko
  - zygisk-detach
  - bindhost
  - Revanced Google Photos
  - Revanced Youtube
  - Revanced Youtube Music
  - LSPosed (JingMatrix)
    - HyperCeiler
    - RevengeXposed
    - Wa Enhancer
