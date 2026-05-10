---
description: Learn OpenWRT VLAN
pubDatetime: 2025-10-21
tags:
- Network
title: OpenWRT IPTV 配置笔记
---

### 一、交换机 VLAN 设置

<!-- Image missing: %E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE_2025-10-21_173316.png -->

| VLAN ID | CPU (eth0) | LAN1 | LAN2 | LAN3 | LAN4 | WAN |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | tagged | untagged | untagged | untagged | untagged | - |
| 2 | tagged | - | - | - | - | untagged |
| 3014 | tagged | - | - | - | tagged | tagged |

---
### 二、配置逻辑与原理分析
### VLAN 1：LAN 内部通信网络
VLAN 1 是默认 LAN 网络，所有 LAN 口（LAN1–LAN4）都在 VLAN 1 中。
- **untagged + PVID=1** 意味着：这些 LAN 口收到的普通以太帧会被交换机自动打上 VLAN 1 标签。
- 在交换机内部，LAN 口之间互通的数据都带 VLAN 1 标签，但发到外部设备时（untagged 出口）标签会被剥离。
- 这样 LAN1 与 LAN2、LAN3 等之间可以直接通信，而不会经过光猫或 WAN。
### VLAN 2：互联网拨号网络 (PPPoE)
- WAN 口属于 VLAN 2，并设置为 **untagged**。这表示光猫侧期望接收的是不带 VLAN 标签的以太帧。
- CPU (eth0) 端口设置为 **tagged**，这样系统内部可以通过 VLAN ID 2 来区分这类数据，并将其交给 PPPoE 客户端接口处理。
- 当报文从 CPU → WAN 时，交换机会去除 VLAN 2 标签；从 WAN → CPU 时，交换机会自动加上 VLAN 2 标签，系统即可识别。
### VLAN 3014：IPTV 专用 VLAN
- 光猫下发 IPTV 流量时通常会带 VLAN 3014 标签。
- WAN 和 LAN4 均为 **tagged**，表示 VLAN 3014 的帧可以双向通过这两个端口，且标签保持不变。
- CPU (eth0) 同样为 tagged，这样 OpenWrt 系统可以创建一个 VLAN 接口（如 eth0.3014）来监听或转发 IPTV 数据。
- 因为 LAN4 是 tagged 口，连接下级路由时不会剥离标签，下级设备可以识别 VLAN 3014 并进一步转发给 IPTV 盒子。
---

### 三、下级路由器的 IPTV 配置逻辑

下级路由器中用于 IPTV 的 LAN 口通常会：

- 设为 **VLAN 3014 untagged**；
- PVID 设为 3014。

这样 IPTV 盒子发送/接收的帧无需 VLAN 标签，由下级路由器的交换机负责加/解标签（3014），实现与上级 OpenWrt 的 VLAN 3014 互通。

---
### 四、总结
- VLAN 1：LAN 内部通信（默认局域网）
- VLAN 2：互联网拨号（WAN ↔ PPPoE）
- VLAN 3014：IPTV 通道（WAN ↔ LAN4 ↔ 下级 IPTV 网络）
LAN4 同时承担：
- VLAN 1（untagged）→ 普通内网；
- VLAN 3014（tagged）→ IPTV 流量。
这样 LAN4 下接的下级路由既能上网，又能透传 IPTV 流量。
### Note:
(一些版本的OpenWRT系统会把tagged/untagged翻译成关联/不关联或已标记/未标记，对于初学者可能会产生混淆)
1️⃣ Tagged / Untagged
- **Tagged（保留标签 ）**
    - 意思是：报文出去时**保留 VLAN 标签**。
    - 适用于交换机间或支持 VLAN 的上联端口。
    - 只有端口允许的 VLAN ID 才能通过这个端口，否则丢弃。
- **Untagged（移除标签 / 去标签）**
    - 意思是：报文出去时**去掉 VLAN 标签**，变成普通以太网帧。
    - 适用于接入普通终端（PC、机顶盒等）。
    - 同样，只有端口允许的 VLAN ID 才能出去，否则丢弃。
> 🔹 重点：Tagged/Untaged 描述的是报文出去时的状态，不是进来的处理逻辑。
> 
---

2️⃣ PVID（Port VLAN ID）

- **作用**：当端口收到**没有 VLAN 标签**的报文时，自动打上这个 VLAN ID。
- **限制**：一个端口只能有一个 PVID。
- **与 Tagged/Untagged 的关系**：
    - 如果端口设置了 Untagged 对应某个 VLAN，那么这个 VLAN 通常就是 PVID（即端口默认 VLAN）。
    - 收到无标签报文 → 自动打上 PVID → 按 VLAN 转发逻辑处理。

---
3️⃣ 总结逻辑图示（简化）：
| 端口方向 | 收到报文 | 处理方式 |
| --- | --- | --- |
| Ingress（收） | Tagged VID=允许的VLAN | 按 VID 转发 |
| Ingress | Untagged | 打上 PVID → 按 VID 转发 |
| Egress（发） | Tagged | 保留 VLAN 标签 |
| Egress | Untagged | 移除 VLAN 标签 |