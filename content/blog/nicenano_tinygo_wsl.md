+++
draft = true
+++

# nice!nano TinyGo Development in WSL(2)

In this first blog I would like to write on how to program nice!nano using
TinyGo in WSL(2) environment.

## Why?

First why is "why nice!nano?". nice!nano (I will call it n!n from now on) is a
Pro-micro replacement board that has Bluetooth capability. Both are mostly used
for custom mechanical keyboard. I bought a bunch of n!n to save shipping cost
per unit as I need to buy them from Taobao.

Second why is "why TinyGo?". Recently I am exploring embedded programming and
found several cool frameworks:

1. Zephyr RTOS

   Zephyr is what the keyboard firmware for n!n, the ZMK, based on. As we need
   to setup Zephyr too for ZMK, I feel the setup process need to be followed
   cautiously. It seems it has the most supported boards.

2. Embassy

   Embassy is the most recent compared to the other two. It has the goodness of
   Rust, but board support is still not as many as others. Embassy also requires
   debug probe such as CMSIS-DAP.

3. TinyGo

   Written in Golang, I found that TinyGo setup is the most straightforward
   among the three as we only need to install one package and we are ready to
   go. The official docs also have entry for n!n. It supports flashing through
   UF2 bootloader. Due to that I proceed with TinyGo.

Third why is "why WSL(2)?". Currently I am using Windows and although I really
want to switch to fulltime Linux I am too afraid as my laptop has Nvidia dGPU
:')

Alright so that is all the whys let us continue to the setup.

## Requirement

1. nice!nano
2. WSL
3. USB cable, to connect n!n with PC
4. Tweezer, to put n!n into bootloader mode

## Compile WSL kernel with USB Mass Storage support

The default WSL kernel is already have support for (most) USB peripherals.
However, we also need USB storage support. Hence we need to compile WSL
kernel with `CONFIG_USB_STORAGE` enabled.

To compile WSL kernel, we will use
[Nevuly/WSL2-Linux-Kernel-Rolling-LTS](https://github.com/Nevuly/WSL2-Linux-Kernel-Rolling-LTS).

1. Clone the kernel repo. We use `--depth 1` for shallow clone to save space and time.

   ```bash
   git clone https://github.com/Nevuly/WSL2-Linux-Kernel-Rolling-LTS.git --depth 1
   cd WSL2-Linux-Kernel-Rolling-LTS
   ```

2. In WSL kernel configuration, update
   [`CONFIG_USB_STORAGE`](https://www.kernelconfig.io/config_usb_storage?q=&kernelversion=6.6.14&arch=x86)
   to `m` (module). We will install this as kernel module later.

   ```bash
   sed -i 's/# CONFIG_USB_STORAGE is not set/CONFIG_USB_STORAGE=m/' arch/x86/configs/config-wsl-x86
   ```

3. You may follow [Build
   Instruction](https://github.com/Nevuly/WSL2-Linux-Kernel-Rolling-LTS#build-instructions),
   but I will put it here too to save your time a bit. The `-j$(nproc)` here will
   make the build to use all of available cores. This might make your PC lag during
   compilation and you may adjust the option as you wish.

   During the compilation you may be asked several additional options for
   `CONFIG_USB_STORAGE_*`. You may choose default/no.

   ```bash
   make KCONFIG_CONFIG=arch/x86/configs/config-wsl-x86 -j$(nproc)
   ```

4. The compiled kernel will be in `arch/x86/boot/bzImage`. We need to copy the
   kernel to Windows partition to be used by WSL. For this we may use File
   Explorer and copy the kernel file. For myself, I organize custom WSL kernel
   in `D:\mangkoran\vmachines\wsl\kernels` but you may use different path.
