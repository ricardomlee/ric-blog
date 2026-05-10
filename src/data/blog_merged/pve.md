---
description: N100 YES!
pubDatetime: 2024-05-11
tags:
- Network
title: PVE + iKuai + OpenWrt on N100 Mini PC
---

PVE is a free virtual platform, I installed it on my newly buyed N100 mini PC. Then I installed a iKuai VM to run as my main router, and a OpenWrt lxc container to run small applications.

Here are the simplified steps:

# Install PVE

Download latest iso from: 

[Downloads](https://www.proxmox.com/en/downloads)

Write to Udisk using etcher.

Choose xfs for disk, add swap

Choose an interface as mgmt, also will be used for LAN

Set IPs

Change PVE repos

# Install iKuai

Download iso from

[固件下载-爱快 iKuai-商业场景网络解决方案提供商 (ikuai8.com)](https://www.ikuai8.com/component/download)

Create VM in PVE:

CPU: set same core as host, choose host type

Memory: 4GB

Storage: 8GB

BIOS: q35

Network: default

Add PCI device passthrough: choose another interface for WAN

Start VM

## Configure iKuai

In WAN setting, complete the pppoe password

In LAN setting, set iKuai IP

In DHCP, set range and default DNS

223.5.5.5, 180.76.76.76

# Install OpenWRT

Downlaod the latest rootfs.tar.gz from [http://downloads.openwrt.org/releases/23.05.2/targets/x86/64/](http://downloads.openwrt.org/releases/23.05.2/targets/x86/64/)

Upload to pve’s CT template

open pve terminal, run:

```bash
pct create 101 \
  local:vztmpl/openwrt-23.05.2-x86-64-rootfs.tar.gz \
  --rootfs local-lvm:2 \
  --ostype unmanaged \
  --hostname OpenWrt \
  --arch amd64 \
  --cores 2 \
  --memory 512 \
  --swap 0 \
  --unprivileged 1 \
  -net0 bridge=vmbr0,name=eth0 \
  -net1 bridge=vmbr1,name=eth1
```

Then run:

```bash
vi /etc/pve/lxc/101.conf
```

add a line:

```bash
lxc.include /usr/share/lxc/config/openwrt.common.conf
```

Start container

In openwrt terminal, run:

```bash
vim /etc/config/network
```

Edit the network setting:

```bash
config interface 'loopback'
	option device 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'
	option ula_prefix ''

config interface 'wan'
	option device 'eth0'
	option proto 'dhcp'
	option peerdns '0'
	list dns '223.5.5.5'
	list dns '180.76.76.76'

config device
	option name 'br-lan'
	option type 'bridge'
	list ports 'eth1'

config interface 'lan'
	option device 'br-lan'
	option proto 'static'
	option ip6assign '60'
	option ipaddr '192.168.96.1'
	option netmask '255.255.255.0'
	option defaultroute '0'

```

Then run:

```bash
service network restart
```

You also need to change DNS manually:

```bash
vim /etc/resolv.conf

nameserver 223.5.5.5
nameserver 180.76.76.76
```

Change the mirror:

```bash
sed -i 's_downloads.openwrt.org_mirrors.tuna.tsinghua.edu.cn/openwrt_' /etc/opkg/distfeeds.conf
```

## Install packages

# References

[畅网X86-P5 N100显卡直通 PVE+iKuai+iStore OS+Linux/Windows Server AlO 踩坑记-软路由,x86系统,openwrt(x86),Router OS 等-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-8307360-1-1.html)

[旁路由新思路：配置更少，效果更佳！_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1AF4m1A7MP/)

[爱快 & OpenWrt 分流拓扑，旁路由模式从此扔进垃圾堆-软路由,x86系统,openwrt(x86),Router OS 等-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-8252571-1-2.html)

[PVE下使用LXC容器配置openwrt旁路由-软路由,x86系统,openwrt(x86),Router OS 等-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-8284348-1-1.html)

[2024原生openwrt通过PVE的LXC安装，添加iStoreOS界面和quicksstarts，小白慎入教程-OPENWRT专版-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-8352956-1-1.html)

[OPenWRT 旁路由+MosDNS+open克拉什+AdGuard Home 傻瓜配置图文教程-软路由,x86系统,openwrt(x86),Router OS 等-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-8284982-1-1.html#google_vignette)