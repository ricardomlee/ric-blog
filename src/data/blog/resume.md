---
description: ''

pubDatetime: 2023-07-06
tags:
- Website
title: Résumé - Ric (Ming) Li
---

<aside>
👋 I'm a **system software engineer** based in **Shanghai** with **4 years** of experience in the software industry. My focus area for the past few years has been **Media Transport Library** development which is written in **C/C++** based on **DPDK, RDMA** and **XDP**, but I'm also fund of coding with **Rust**.

</aside>

> 👉[Chinese résumé](https://ricli.tech/resume-cn)
> 

### **Contact**

[📧](mailto:adalovelace@mail.com) [ricmli@outlook.com](mailto:ricmli@outlook.com)

[🐦 twitter.com/ricmli](http://twitter.com/ricmli)

[🔗 linkedin.com/in/ric-ming-li](https://linkedin.com/in/ric-ming-li)

[👾 github.com/ricmli](https://github.com/ricmli)

# Experience

### System **Software Engineer**

***Intel**, Shanghai, China – (July 2020 - Present)*

- [Media Transport Library](https://github.com/OpenVisualCloud/Media-Transport-Library) (2021 - Present)
    - Implemented the **SMPTE ST2110** standards.
    - Utilize **DPDK** and **XDP** for ~**100Gbps** uncompressed video packet processing.
    - Utilize **RDMA (RoCEv2)** for low-overhead inter-node lossless media data transport.
    - Wrote ergonomic **Rust** bindings for the library.
    - Developed **RTCP** protocol for packet retransmission, make the session recoverable from up to **10%** packet loss.
    - Optimized resource usage by leveraging **DMA** engine (DSA) for memory copy, saving **50%** of CPU resources.
    - Achieved up to **8x** speed increase in YUV color format conversion through **AVX512** optimization.
    - Implemented control plane protocols (**ARP, DHCP, IGMP**) with DPDK.
    - Improved **PTP** algorithm for enhanced accuracy within **50ns**.
    - Enabled **KVM** deployment for both Linux and Windows VM.
    - Extended support for **AWS EC2** instances using **ENA** interfaces.
    - Created a DPDK-based ld_preload **UDP stack** to support **QUIC** and other protocols.
    - Contributed to building an ecosystem with Intel **MSDK**, **OBS Studio**, and more.
- Android Cloud Gaming (2020 - 2021)
    - Developed a virtual camera input for **Android client**, streamlining capture and encode processes.
    - Profiled and optimized the cloud camera side latency.
    - Orchestrated Kubernetes deployment, developed **metrics exporter** for Intel dGPU, and established a **Prometheus** monitoring system.
    - Enabled Intel **GPU device plugin** for scalable Android container deployment.

---

# Education

### Master**'s Degree in Control Engineering**

***HUST**, Wuhan, China – (September 2018 - June 2020)*

### **Bachelor's Degree in Automation**

***HUST**, Wuhan, China – (September 2014 - June 2018)*

---

# Skills

### C

My foundational programming language, mastered over 5 years of hands-on experience. I'm adept in system programming, with extensive use on Linux and some on Windows.

### DPDK

I have a solid two-year background in developing DPDK-based software and am adept with DPDK's intricacies on both Linux and Windows platforms.

### eBPF/XDP

Actively expanding my knowledge in eBPF and the kernel networking stack.

### RDMA

Familiar with the user space APIs, I have implemented some features based on RoCEv2 NIC.

### K8S

I have practical experience with Kubernetes deployment and development, contributing to my understanding of container orchestration.

### C++

My experience with system programming also extends to C++, utilized in both Linux and Windows systems.

### Rust / Zig

I'm tackling the learning curve of Rust and Zig, embracing the challenges they bring. While I'm not yet an expert, I've independently developed some features and fixes.

### Technical writing

I have a genuine passion for demystifying complex technical concepts through writing, aiming to simplify and clarify for the benefit of others.

### Generative AI

I harness AI to boost productivity in tasks such as document and script writing (including this page🤗). My interest also lies in deploying AI services, including GPT for chat applications and VITS for text-to-speech synthesis.

---

# Languages

### Chinese 🇨🇳

Native speaker

### English 🇺🇸

Proficient speaker

### Spanish 🇪🇸

Estudio con Duolingo

---

# Publications

- ACM MHV’23 [RMTS: A Real-time Media Transport Stack Based on Commercial Off-the-shelf Hardware](https://dl.acm.org/doi/10.1145/3588444.3591002)
- DTTC 2022 Kahawai: A SMPTE ST 2110-Compliant Development Kit Based on IA Platform
- DTTC 2024 Leveraging XDP for Effcient and Cost-Effective Media Transport
- Patent: [Enhanced image slice reconstruction for video streams](https://patents.google.com/patent/WO2024065451)
- Patent: Latency-sensitive RTP packet recovery method for the SMPTE ST2110 standard

---