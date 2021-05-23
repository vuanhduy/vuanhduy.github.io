---
layout: post
title:  "Pixel 5 (code `redfin`) as AOSP Development Platform"
date:   2021-05-22 10:56:33 -0400
categories: ["Embedded system", "Android"]
tags: ["Raspberry Pi", "Linux Kernel", "Android"]
toc: true
---

This note includes a set of steps to build the Android Open Source Project (AOSP) for Google Pixel 5 device.

## Setting up `udev` rules:


## Downloading source code:
1. AOSP:
    - Using [this](https://source.android.com/setup/build/initializing) to setup your local PC.
    - Downloading AOSP code. I am using the branch `android-11.0.0_r37`, which is the latest release at the time writing this article.
1. Vendor image and Qualcomm driver
   - Location: https://developers.google.com/android/drivers
   - Remember to select the binary version compatible with the AOSP downloaded above. (I'm using [this](https://developers.google.com/android/drivers#redfinrq2a.210505.003), which is compatible with `android-11.0.0_r37`.)
   - In `<AOSP-root>/vendor/qcom/redfin/proprietary/Android.mk` file, remove `$(LOCAL_PATH)/../COPYRIGHT` in following lines:
   ```
   ...
   LOCAL_NOTICE_FILE := $(LOCAL_PATH)/../COPYRIGHT $(LOCAL_PATH)/../LICENSE
   ...
   LOCAL_NOTICE_FILE := $(LOCAL_PATH)/../COPYRIGHT $(LOCAL_PATH)/../LICENSE
   ...
   LOCAL_NOTICE_FILE := $(LOCAL_PATH)/../COPYRIGHT $(LOCAL_PATH)/../LICENSE
   ...
   ```

## Buiding AOSP

Execute the followings commands
```shell
cd <AOSP>
source build/envsetup.sh
lunch aosp_redfin-userdebug
m -j`nproc`
```

## Flashing:

These steps are done on your Pixel 5 device
1. Enable developer mode:
    1. Go to `Settings > About Phone`,
    1. Tap the `Build number` entry until a series of toast messages appear, informinig about the developer mode, and you will be ask to enter your lock screen passcode,
    1. Enter the passcode and you will get a toast message saying that "You are now a developer!".
  
1. OEM unlocking the device:
    1. Go to `Settings > System > Developer options`,
    1. Enable `USB debugging` and `OEM unlocking`.

These steps are done on your PC:
1. Booting into fastboot mode:
    1. `adb reboot bootloader`
    2. `fastboot flashing unlock`
    3. On the device, use **Volumn buttons** to navigate to **Unload the Bootloader**, and **Power button** to select.

1. Fashing the build:
    ```
    fastboot -w flashall
    ```

1. Enabling remount:
    ```
    adb root && sleep 1 && adb disable-verity && sleep 1 && adb reboot && adb wait-for-device root && sleep 1 &&  adb remount
    ```

Notes:
1. Unloading the Bootloader also erases all data on the device!
1. If you have more than 1 Android device connected, use `adb devices` to list them all and `adb -s <device_id> <command>` to send `command` to specific device.
1. If you haven't set up udev rules or don't know how to do so, use `sudo <environment-variables> $(which fastboot) <command>` instead of `fasboot <command>`.

## Restoring device to factory state:
https://developers.google.com/android/images

## Reference:
1. Flashing devices: https://source.android.com/setup/build/running#selecting-device-build
2. Automotive development platforms: https://source.android.com/devices/automotive/start/pixelxl
