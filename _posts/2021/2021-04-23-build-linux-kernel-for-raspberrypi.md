---
layout: post
title:  "Building Linux Kernel for Raspberry Pi"
date:   2021-04-23 10:56:33 -0400
categories: ["Embedded system"]
tags: ["Raspberry Pi", "Linux Kernel"]
toc: true
---

This note includes a set of steps to build the Linux kernel locally for Raspberry Pi SBC (i.e., building the kernel on the board.)

## Preparation
1. Update the system

    ```shell
    sudo apt update
    sudo apt upgrade
    sudo rpi-update
    sudo reboot
    ```
    
2. Install necessary tools

    ```shell
    sudo apt install git bc bison flex libssl-dev build-essential
    ```

3. Create a workspace and download the kernel code

    ```shell
    mkdir ~/workspace; cd ~/workspace
    git clone --depth=1 https://github.com/raspberrypi/linux
    ```
The `git` command will create a `linux` directory, download and put the code inside the directory.

## Kernel configuration

First, prepare the default configuration by running the following commands, depending on the Raspberry Pi version:
```shell
cd ~/workspace/linux
# For Raspberry Pi 1, Pi Zero, Pi Zero W, and Compute Module
make bcmrpi_defconfig
# For Raspberry Pi 2, Pi 3, Pi 3+, and Compute Module 3
make bcm2709_defconfig
# For Raspberry Pi 4
make bcm2711_defconfig 
```

Next, we need to adjust the `LOCALVERSION` to ensure the new kernel does not have the same version string as the upstream. This helps to clarify that we are running our kernel by invoking `uname` command and ensures existing modules in `/lib/modules` are not overwritten. To do so, change the following line in `.config`

```shell
CONFIG_LOCALVERSION="-v7l-<your own string>"
```

## Building and installation

Build and install the kernel, modules, and device tree blobs:

```shell
make -j4 zImage modules dtbs
sudo make modules_install
```

This step can take a long time (depending on the Pi model in use). Next, we install the newly-created module into `/lib/modules`:

```shell
sudo make modules_install
```

Then copy the kernel and device tree blobs files to the `/boot` partition:

```
sudo cp arch/arm/boot/dts/*.dtb /boot/
sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
sudo cp arch/arm/boot/zImage /boot/$KERNEL.img
```

Note that we can copy the kernel file (i.e., `kernel.img` file) into the same place, but with a different filename - for instance, `my_new_kernel.img`. We then edit the `config.txt` file to select the kernel that the Pi will boot into:

```
kernel= my_new_kernel.img
```

Finally, reboot the Pi

```shell
sudo reboot
```