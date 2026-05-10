---
description: DPDK fast setup on Windows
pubDatetime: 2023-09-05
tags:
- DPDK
- Network
title: Build and install DPDK on Windows
---

This post is about how to **easily** build and run DPDK natively on Windows platform. The test-pmd app is tested with ring-based PMD since I don’t have a physical Intel NIC on my PC.

<aside>
💡 Estimated time: 10 min

</aside>

<aside>
💡 Windows version supported: Windows 11, Windows 10 (1909 and newer), Windows Server (2019 and newer)

</aside>

<aside>
💡 To use a real NIC such as Intel I225/I226, X710, E810 or E1000, please build and install virt2phy and netuio drivers following readme guides here: [https://git.dpdk.org/dpdk-kmods/](https://git.dpdk.org/dpdk-kmods/)

</aside>

## Install MSYS2

MSYS2 provides the collection of tools to make you fell like running native Linux on Windows, also you can use **pacman** to install up-to-date software packages which are natively built on Windows. In short, it’s a powerful version of MinGW!

Download and install from the official site: [https://www.msys2.org/](https://www.msys2.org/)

Open the **UCRT64** terminal for the following steps.

> UCRT64 environment is recommended because it has better compatibility with MSVC, we can directly develop C/C++ applications in MSVC while including the DPDK headers and libs.
> 

## Install dependencies

```bash
pacman -Syuu
pacman -S git base-devel unzip pactoys
pacboy -S meson:p gcc:p
```

## Build and install DPDK

```bash
git clone https://github.com/DPDK/dpdk.git
cd dpdk
meson setup build -Denable_apps='test-pmd'
meson install -C build
```

## Run test-pmd with rings-based PMD

Ring-pmd uses software FIFOs to act like a physical NIC.

More about ring-based PMD: [https://doc.dpdk.org/guides/nics/pcap_ring.html](https://doc.dpdk.org/guides/nics/pcap_ring.html)

```bash
$ dpdk-testpmd -l 1-3 -n 4 --vdev=net_ring0 --vdev=net_ring1 -- -i
EAL: Detected CPU lcores: 12
EAL: Detected NUMA nodes: 1
EAL: Multi-process support is requested, but not available.
Interactive-mode selected
testpmd: create a new mbuf pool <mb_pool_0>: n=163456, size=2176, socket=0
testpmd: preferred mempool ops selected: ring_mp_mc
Configuring Port 0 (socket 0)
Port 0: 00:00:00:00:00:00
Configuring Port 1 (socket 0)
Port 1: 00:00:00:00:00:00
Checking link statuses...
Done
testpmd> start tx_first
io packet forwarding - ports=2 - cores=1 - streams=2 - NUMA support enabled, MP allocation mode: native
Logical Core 2 (socket 0) forwards packets on 2 streams:
  RX P=0/Q=0 (socket 0) -> TX P=1/Q=0 (socket 0) peer=02:00:00:00:00:01
  RX P=1/Q=0 (socket 0) -> TX P=0/Q=0 (socket 0) peer=02:00:00:00:00:00

  io packet forwarding packets/burst=32
  nb forwarding cores=1 - nb forwarding ports=2
  port 0: RX queue number: 1 Tx queue number: 1
    Rx offloads=0x0 Tx offloads=0x0
    RX queue: 0
      RX desc=0 - RX free threshold=0
      RX threshold registers: pthresh=0 hthresh=0  wthresh=0
      RX Offloads=0x0
    TX queue: 0
      TX desc=0 - TX free threshold=0
      TX threshold registers: pthresh=0 hthresh=0  wthresh=0
      TX offloads=0x0 - TX RS bit threshold=0
  port 1: RX queue number: 1 Tx queue number: 1
    Rx offloads=0x0 Tx offloads=0x0
    RX queue: 0
      RX desc=0 - RX free threshold=0
      RX threshold registers: pthresh=0 hthresh=0  wthresh=0
      RX Offloads=0x0
    TX queue: 0
      TX desc=0 - TX free threshold=0
      TX threshold registers: pthresh=0 hthresh=0  wthresh=0
      TX offloads=0x0 - TX RS bit threshold=0
testpmd> stop
Telling cores to stop...
Waiting for lcores to finish...

  ---------------------- Forward statistics for port 0  ----------------------
  RX-packets: 2150563104     RX-dropped: 0             RX-total: 2150563104
  TX-packets: 2150563136     TX-dropped: 0             TX-total: 2150563136
  ----------------------------------------------------------------------------

  ---------------------- Forward statistics for port 1  ----------------------
  RX-packets: 2150563104     RX-dropped: 0             RX-total: 2150563104
  TX-packets: 2150563136     TX-dropped: 0             TX-total: 2150563136
  ----------------------------------------------------------------------------

  +++++++++++++++ Accumulated forward statistics for all ports+++++++++++++++
  RX-packets: 4301126208     RX-dropped: 0             RX-total: 4301126208
  TX-packets: 4301126272     TX-dropped: 0             TX-total: 4301126272
  ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Done.
```

You can run more tests with test-pmd, read more: [https://doc.dpdk.org/guides/testpmd_app_ug/index.html](https://doc.dpdk.org/guides/testpmd_app_ug/index.html)

## What’s next

Now let’s try to write some simple applications with DPDK! 

This post does not include how to develop MSVC applications for DPDK, If you are interested in this topic, you can check how Intel Media Tranport Library does: [Media-Transport-Library/app/sample/msvc/README.md at main · OpenVisualCloud/Media-Transport-Library (github.com)](https://github.com/OpenVisualCloud/Media-Transport-Library/blob/main/app/sample/msvc/README.md), the trick is to generate .def file (add this to your library meson file: `c_args += ['-Wl,--output-def,libxxx.def']`) then convert it to the .lib file for VS to use. The DPDK main branch is also adding more native MSVC support in another way (maybe LLVM 🤔).