---
description: RDMA saved my CPU

pubDatetime: 2024-06-06
tags:
- Network
- RDMA
title: Using RDMA for Media Transport - Draft
---

RDMA is not a new technology, but recently it is mentioned a lot thanks to the emerging of ChatGPT (LLM). The main idea of RDMA is to transfer data between machines without the intervention of CPU (in data path). In LLM training, since the model is very large, it is divided into pieces and trained on multiple GPUs or even multiple nodes. RDMA is used to transfer the mathmatical data between the nodes.

The Media Transport Library is initially designed for handling ST2110 transmitting and receiving on foundational NICs. ST2110 is the standard for IP based media such as uncompressed video and audio. For example, the traditional cameras using SDI to transport the captured signal to the backend for further proccessing. Nowadays, some cameras support ST2110 which encapsulate media data within RTP packet and send out using ethernet cables. With MTL, one can send or receive the standard compliant ST2110 streams, so specialized and expensive ST2110 hardware not needed. This can also be used to transport raw video/audio data between data center machines and MTL already has support for AWS. However, the ST2110 stack is based on DPDK which uses poll mode driver for sending/receiving network packet, so it requires dedicated CPU for the transport tasks. Luckily we have RDMA to offload this work from CPU. The intel E810 NIC can support RoCEv2 which is an RDMA implementation based on the UDP protocol. With RDMA and its interrupt mode, a lot of CPU resources can be saved especailly for cloud  where any computing resource costs.

Before implementing this feature, we are totally new to RDMA. As we get deeper, we find the design of RDMA is really clear. What suprises us is the performance and reliability. Compare to the ST2110 based way, the CPU for a single 1080p video session is only 1/20.

In this series I will go through the journey of RDMA for media transport including the architechture design and the source code. I hope this small series will help anyone who would like to start coding with RDMA🙂