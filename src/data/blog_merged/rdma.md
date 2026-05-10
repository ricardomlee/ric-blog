---
description: RDMA saved my CPU.
pubDatetime: 2024-06-06
tags:
- Network
- RDMA
title: Using RDMA for Media Transport - Intro
---

RDMA, or Remote Direct Memory Access, isn't a new technology. However, its relevance has increased recently due to the rise of ChatGPT (LLM). RDMA's primary function is to enable data transfer between machines without CPU intervention. In the training of LLM, the model is divided into sections and trained across multiple GPUs or nodes due to its large size. RDMA facilitates the transfer of mathematical data between these nodes.

**Simplifying Media Transport with the Media Transport Library**

The [Media Transport Library (MTL)](https://github.com/OpenVisualCloud/Media-Transport-Library) was developed to streamline the process of sending and receiving media over standard network interfaces, specifically focusing on the ST2110 standard. ST2110 is crucial for transmitting high-quality, uncompressed video and audio over IP networks. Traditionally, cameras used Serial Digital Interface (SDI) to send footage for processing. However, with the emergence of ST2110, media data can now be encapsulated in RTP packets and transmitted via Ethernet, making the process more straightforward and less reliant on specialized hardware. With MTL, you can transmit or receive ST2110 standard-compliant streams, eliminating the need for specialized and expensive ST2110 hardware. This technology can also transport raw video/audio data between data center machines, and MTL already supports AWS.

One challenge with MTL's ST2110 stack is its reliance on DPDK (Data Plane Development Kit), which demands dedicated CPU resources for network packet handling. This is where RDMA comes into play, offering a solution to offload these tasks from the CPU. Specifically, the Intel E810 Network Interface Card (NIC) supports RoCEv2, an RDMA implementation that uses the UDP protocol. By leveraging RDMA and its interrupt mode, significant CPU resources can be conserved, which is particularly advantageous in cloud computing environments where resource optimization is critical.

**Our Journey with RDMA**

Embarking on the RDMA journey was an enlightening experience. Initially unfamiliar with the technology, we quickly appreciated its well-designed architecture, impressive performance, and reliability. Remarkably, when compared to traditional ST2110 methods, RDMA required only a fraction of the CPU resources for handling a single 1080p video session.

In this series, we'll delve into our exploration of RDMA for media transport, covering both the architectural design and the actual coding process. Whether you're new to RDMA or looking to deepen your understanding, we hope this series will provide valuable insights and inspire you to start your own RDMA projects.