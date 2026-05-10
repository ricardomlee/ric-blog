---
description: ''
pubDatetime: 2023-11-07
tags:
- Website
title: 个人简历 - 李明
---

<aside>
👋 我是一名现居**上海**，练习时长**4年**的**系统软件工程师**。在过去的几年里，我专注于**媒体传输库（Media Transport Library）**开发，主要运用 **C/C++** 语言，并且通过 **DPDK，RDMA** 和 **XDP** 等技术来优化性能。除此之外，我也热衷于 **Rust** 编程，积极探索其提供的现代化特性以提升我的编程技术。

</aside>

> 👉[英文简历](https://ricli.tech/resume)
> 

### 联系方式

[📧](mailto:adalovelace@mail.com) [ricmli@outlook.com](mailto:ricmli@outlook.com)

[👾 github.com/ricmli](https://github.com/ricmli)

# 工作经历

### **系统软件工程师**

***英特尔**, 上海 – (2020年7月 - 至今)*

- **媒体传输库** [Media Transport Library](https://github.com/OpenVisualCloud/Media-Transport-Library) (2021 - 至今)
    - 实现了 **SMPTE ST2110** 标准。
    - 使用 **DPDK** 和 **XDP** 以支持 **100Gbps** 无压缩视频流收发包处理。
    - 开发了 **RTCP** 协议以改善数据包丢包重传，最高可支持 **10%** 的丢包恢复。
    - 利用 **DMA** 引擎（DSA）减少内存拷贝开销，节省了 **50%** 的 CPU 资源。
    - 通过 **AVX512** 优化实现了 YUV 颜色格式转换的最高 **8 倍**速度提升。
    - 基于 **DPDK** 实现了控制平面协议（**ARP, DHCP, IGMP**）。
    - 微调 **PTP** 算法，实现了 **50 纳秒以内**的精确度。
    - 实现了基于 **KVM** 的 Linux 和 Windows 虚拟机的部署。
    - 添加了对 **AWS EC2** 实例 ENA 接口的支持。
    - 构建了基于 DPDK 的 ld_preload **UDP 协议栈**以支持 **QUIC** 等协议。
    - 编写了英特尔 **MSDK**、**OBS Studio** 等插件以扩展生态系统。
- **Android 云游戏** (2020 - 2021)
    - 虚拟摄像头输入
        - 开发了 **Android 客户端**的虚拟摄像头输入：相机捕捉和编码。
        - 对云端延迟进行了分析和优化。
    - Kubernetes 部署
        - 为英特尔 dGPU 开发了**指标导出器**，并建立了 **Prometheus** 监控系统。
        - 实现了基于英特尔 **GPU 设备插件**的 Android 容器部署。

---
# 教育经历
### 硕士 - 控制工程
***华中科技大学，人工智能与自动化学院，**武汉 – (2018 - 2020)*
### 本科 - 自动化
***华中科技大学，自动化学院，**武汉 – (2014 - 2018)*
---

# 个人技能

### C

有超过五年的 C 语言编程经验，广泛应用于 Linux 和 Windows 平台的系统编程。

### DPDK

有超过两年基于 DPDK 的软件开发经验，熟悉在 Linux 和 Windows 平台上的 DPDK 编程。

### eBPF/XDP

正在学习 eBPF 和 Linux 内核网络栈，基于 XDP 实现了高速数据包处理。

### RDMA

熟悉 RDMA Verbs API 编程模式，基于 RoCEv2 网卡实现了高性能数据传输。

### K8S

有 Kubernetes 部署和开发的实际经验。

### C++

熟悉在 Linux 和 Windows 环境下使用 C++ 进行系统编程。

### Rust / Zig

最近开始学习 Rust 和 Zig。

### **技术写作**

热衷于撰写技术内容，旨在将复杂概念简化为清晰、实用的操作指南。

### 生成式 AI

利用 AI 提高文档和脚本编写的生产力。同时对部署自己的 AI 服务，如 GPT（聊天机器人）和 VITS（文本到语音模型）充满兴趣。

---
# 语言
### 中文 🇨🇳
普通话 二甲
### 英文 🇺🇸
六级 604
---

# 论文

- [RMTS: A Real-time Media Transport Stack Based on Commercial Off-the-shelf Hardware | Proceedings of the 2nd Mile-High Video Conference (acm.org)](https://dl.acm.org/doi/10.1145/3588444.3591002)

---