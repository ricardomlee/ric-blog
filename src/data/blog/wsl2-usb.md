---
description: use WSL to flash your devices

pubDatetime: 2023-03-29
tags:
- Others
title: Attach USB in WSL2
---

## Windows

install usbipd

Go to the [latest release page for the usbipd-win project](https://github.com/dorssel/usbipd-win/releases).

list usb

```bash
usbipd wsl list
BUSID  VID:PID    DEVICE                                                        STATE
1-5    1bcf:08a0  USB 输入设备                                                  Not attached
1-6    05ac:024f  USB 输入设备                                                  Not attached
1-14   8087:0033  英特尔(R) 无线 Bluetooth(R)                                   Not attached
1-19   0781:5581  USB 大容量存储设备                                            Not attached
```

attach usb

```bash
usbipd wsl attach --busid 1-19
```

## WSL

install usbip, lsusb

```bash
yay -S usbip usbutils

lsusb
Bus 002 Device 002: ID 0781:5581 SanDisk Corp. Ultra
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

## AI says:

To detach the USB device, run `usbipd wsl detach --busid 1-19`. Once the device is detached, it can be safely removed from the computer. Note that if the USB device was mounted in WSL, it must be unmounted before detaching it.

To use the USB device in WSL2, it must first be attached to Windows using usbipd. Once the device is attached, list the available USB devices using the command `lsusb`. Select the device to use and attach it to WSL2 using the command `sudo usbip attach -r <remote-host> -b <bus-id>`. The USB device should now be usable within WSL2. When finished, detach the device using the command `sudo usbip detach -p <port-number>`.

Here are the steps to use USB devices in WSL2:

1. Install `usbip` and `usbutils` in WSL2 using the command `yay -S usbip usbutils`.
2. Attach the USB device to Windows using `usbipd wsl attach --busid <busid>`.
3. List the available USB devices using the command `lsusb`.
4. Select the device to use and attach it to WSL2 using the command `sudo usbip attach -r <remote-host> -b <bus-id>`.
5. The USB device should now be usable within WSL2.
6. When finished, detach the device using the command `sudo usbip detach -p <port-number>`.

Note that if the USB device was mounted in WSL2, it must be unmounted before detaching it.

I hope this helps! Let me know if you have any other questions or concerns.