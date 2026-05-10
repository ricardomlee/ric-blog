---
description: First step to try BPF
pubDatetime: 2022-07-03
tags:
- Others
title: Build and use latest WSL2 kernel from Microsoft
---

## Build WSL Kernel

```bash
git clone https://github.com/microsoft/WSL2-Linux-Kernel.git
cd WSL2-Linux-Kernel
cp Microsoft/config-wsl .config
# make -j<num_threads>
make -j12
# copy to windows folder
cp arch/x86/boot/bzImage /mnt/c/Users/$USER/Linux/kernels/linux-msft-wsl
```

## Edit C:\Users\$USER\.wslconfig

```
# Settings apply across all Linux distros running on WSL 2
[wsl2]

# Specify a custom Linux kernel to use with your installed distros. The default kernel used can be found at https://github.com/microsoft/WSL2-Linux-Kernel
kernel=C:\\Users\\ric\\Linux\\kernels\\linux-msft-wsl\\bzImage

# Disable page reporting so WSL retains all allocated memory claimed from Windows and releases none back when free
pageReporting=false

# Turn off default connection to bind WSL 2 localhost to Windows localhost
localhostforwarding=true

# Disables nested virtualization
nestedVirtualization=false

# Turns on output console showing contents of dmesg when opening a WSL 2 distro for debugging
debugConsole=false
```

## Restart WSL2 distro

in powershell

```powershell
wsl --shutdown
wsl --distribution <Distribution Name>
```

in wsl

```bash
uname -r
```

## Build and install kernel modules

```bash
cd WSL2-Linux-Kernel
make oldconfig && make prepare
make scripts
make modules && sudo make modules_install

ls /lib/modules/
```