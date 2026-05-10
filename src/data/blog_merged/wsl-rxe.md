---
description: ''
pubDatetime: 2024-07-31
tags:
- Network
- RDMA
title: WSL 搭建 Soft-RoCE RDMA 开发环境
---

## Build WSL kernel with RXE

Download new kernel source code from: 

[The Linux Kernel Archives](https://kernel.org/)

<!-- Image missing: 屏幕截图 2024-08-04 194149.png -->

```bash
# Arch Linux
sudo pacman -S base-devel flex bison pahole openssl libelf bc
# Ubuntu
sudo apt install build-essential flex bison pahole openssl libelf-dev bc libncurses-dev pkg-config libssl-dev

# Download WSL kernel config
wget https://raw.githubusercontent.com/microsoft/WSL2-Linux-Kernel/linux-msft-wsl-6.1.y/arch/x86/configs/config-wsl -O arch/x86/configs/config-wsl

# Edit config
make menuconfig KCONFIG_CONFIG=arch/x86/configs/config-wsl
# type: /RDMA_RXE
# 1
# choose soft roce, Infinibad userspace, RDMA verbs transport library
# General setup –>Local version -> custom version

# Build
make -j$(nproc) KCONFIG_CONFIG=arch/x86/configs/config-wsl
```

Copy `arch/x86/boot/bzImage` to custom kernel folder.
Edit kerenl path in `C:\Users\<username>\.wslconfig`, restart WSL.

```
[wsl2]
kernel=C:\\Users\\<username>\\Linux\\kernel\\rxe\\bzImage
```

## Network interfaces setup

Add virtual network interface and ip addresses

```bash
sudo ip link add veth0 type veth peer name veth1
sudo ip addr add 192.168.96.110/24 dev veth0
sudo ip addr add 192.168.96.111/24 dev veth1
sudo ip link set veth0 up
sudo ip link set veth1 up
ip a
```

Allow loopback test

```bash
sudo sysctl -w net.ipv4.conf.veth0.accept_local=1
sudo sysctl -w net.ipv4.conf.veth1.accept_local=1
sudo ping -I veth0 192.168.96.111
```

## RDMA devices setup

Install RDMA toolset.

```bash
# Arch Linux
yay -S rdma-core perftest
# Ubuntu
sudo apt install rdma-core perftest infiniband-diags ibverbs-utils
```

Add rxe devices.

```bash
sudo rdma link add rxe_0 type rxe netdev veth0
sudo rdma link add rxe_1 type rxe netdev veth1

[ric@Ric-PC ~]$ rdma link
link rxe_0/1 state DOWN physical_state DISABLED netdev veth0
link rxe_1/1 state DOWN physical_state DISABLED netdev veth1

[ric@Ric-PC ~]$ ibstat
CA 'rxe_0'
        CA type:
        Number of ports: 1
        Firmware version:
        Hardware version:
        Node GUID: 0x74ab3ffffe42e7cf
        System image GUID: 0x74ab3ffffe42e7cf
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 10 (FDR10)
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x74ab3ffffe42e7cf
                Link layer: Ethernet
CA 'rxe_1'
        CA type:
        Number of ports: 1
        Firmware version:
        Hardware version:
        Node GUID: 0x7cc8b7fffee4486b
        System image GUID: 0x7cc8b7fffee4486b
        Port 1:
                State: Active
                Physical state: LinkUp
                Rate: 10 (FDR10)
                Base lid: 0
                LMC: 0
                SM lid: 0
                Capability mask: 0x00010000
                Port GUID: 0x7cc8b7fffee4486b
                Link layer: Ethernet

[ric@Ric-PC ~]$ ibv_devices
    device                 node GUID
    ------              ----------------
    rxe_0               74ab3ffffe42e7cf
    rxe_1               7cc8b7fffee4486b
    
[ric@Ric-PC ~]$ ibv_devinfo -v
hca_id:	rxe_0
	transport:			InfiniBand (0)
	fw_ver:				0.0.0
	node_guid:			88d2:b2ff:fec9:ea5a
	sys_image_guid:			88d2:b2ff:fec9:ea5a
	vendor_id:			0xffffff
	vendor_part_id:			0
	hw_ver:				0x0
	phys_port_cnt:			1
	max_mr_size:			0xffffffffffffffff
	page_size_cap:			0xfffff000
	max_qp:				1048560
	max_qp_wr:			1048576
	device_cap_flags:		0x01223c76
					BAD_PKEY_CNTR
					BAD_QKEY_CNTR
					AUTO_PATH_MIG
					CHANGE_PHY_PORT
					UD_AV_PORT_ENFORCE
					PORT_ACTIVE_EVENT
					SYS_IMAGE_GUID
					RC_RNR_NAK_GEN
					SRQ_RESIZE
					MEM_WINDOW
					MEM_MGT_EXTENSIONS
					MEM_WINDOW_TYPE_2B
	max_sge:			32
	max_sge_rd:			32
	max_cq:				1048576
	max_cqe:			32767
	max_mr:				524287
	max_pd:				1048576
	max_qp_rd_atom:			128
	max_ee_rd_atom:			0
	max_res_rd_atom:		258048
	max_qp_init_rd_atom:		128
	max_ee_init_rd_atom:		0
	atomic_cap:			ATOMIC_HCA (1)
	max_ee:				0
	max_rdd:			0
	max_mw:				524287
	max_raw_ipv6_qp:		0
	max_raw_ethy_qp:		0
	max_mcast_grp:			8192
	max_mcast_qp_attach:		56
	max_total_mcast_qp_attach:	458752
	max_ah:				32767
	max_fmr:			0
	max_srq:			917503
	max_srq_wr:			1048576
	max_srq_sge:			27
	max_pkeys:			64
	local_ca_ack_delay:		15
	general_odp_caps:
	rc_odp_caps:
					NO SUPPORT
	uc_odp_caps:
					NO SUPPORT
	ud_odp_caps:
					NO SUPPORT
	xrc_odp_caps:
					NO SUPPORT
	completion_timestamp_mask not supported
	core clock not supported
	device_cap_flags_ex:		0x1C001223C76
					Unknown flags: 0x1C000000000
	tso_caps:
		max_tso:			0
	rss_caps:
		max_rwq_indirection_tables:			0
		max_rwq_indirection_table_size:			0
		rx_hash_function:				0x0
		rx_hash_fields_mask:				0x0
	max_wq_type_rq:			0
	packet_pacing_caps:
		qp_rate_limit_min:	0kbps
		qp_rate_limit_max:	0kbps
	tag matching not supported
	num_comp_vectors:		4
		port:	1
			state:			PORT_ACTIVE (4)
			max_mtu:		4096 (5)
			active_mtu:		1024 (3)
			sm_lid:			0
			port_lid:		0
			port_lmc:		0x00
			link_layer:		Ethernet
			max_msg_sz:		0x800000
			port_cap_flags:		0x00010000
			port_cap_flags2:	0x0000
			max_vl_num:		1 (1)
			bad_pkey_cntr:		0x0
			qkey_viol_cntr:		0x0
			sm_sl:			0
			pkey_tbl_len:		1
			gid_tbl_len:		1024
			subnet_timeout:		0
			init_type_reply:	0
			active_width:		1X (1)
			active_speed:		10.0 Gbps (8)
			phys_state:		LINK_UP (5)
			GID[  0]:		fe80::88d2:b2ff:fec9:ea5a, RoCE v2
			GID[  1]:		::ffff:192.168.96.110, RoCE v2

hca_id:	rxe_1
	transport:			InfiniBand (0)
	fw_ver:				0.0.0
	node_guid:			c4a2:c9ff:fe03:cf0b
	sys_image_guid:			c4a2:c9ff:fe03:cf0b
	vendor_id:			0xffffff
	vendor_part_id:			0
	hw_ver:				0x0
	phys_port_cnt:			1
	max_mr_size:			0xffffffffffffffff
	page_size_cap:			0xfffff000
	max_qp:				1048560
	max_qp_wr:			1048576
	device_cap_flags:		0x01223c76
					BAD_PKEY_CNTR
					BAD_QKEY_CNTR
					AUTO_PATH_MIG
					CHANGE_PHY_PORT
					UD_AV_PORT_ENFORCE
					PORT_ACTIVE_EVENT
					SYS_IMAGE_GUID
					RC_RNR_NAK_GEN
					SRQ_RESIZE
					MEM_WINDOW
					MEM_MGT_EXTENSIONS
					MEM_WINDOW_TYPE_2B
	max_sge:			32
	max_sge_rd:			32
	max_cq:				1048576
	max_cqe:			32767
	max_mr:				524287
	max_pd:				1048576
	max_qp_rd_atom:			128
	max_ee_rd_atom:			0
	max_res_rd_atom:		258048
	max_qp_init_rd_atom:		128
	max_ee_init_rd_atom:		0
	atomic_cap:			ATOMIC_HCA (1)
	max_ee:				0
	max_rdd:			0
	max_mw:				524287
	max_raw_ipv6_qp:		0
	max_raw_ethy_qp:		0
	max_mcast_grp:			8192
	max_mcast_qp_attach:		56
	max_total_mcast_qp_attach:	458752
	max_ah:				32767
	max_fmr:			0
	max_srq:			917503
	max_srq_wr:			1048576
	max_srq_sge:			27
	max_pkeys:			64
	local_ca_ack_delay:		15
	general_odp_caps:
	rc_odp_caps:
					NO SUPPORT
	uc_odp_caps:
					NO SUPPORT
	ud_odp_caps:
					NO SUPPORT
	xrc_odp_caps:
					NO SUPPORT
	completion_timestamp_mask not supported
	core clock not supported
	device_cap_flags_ex:		0x1C001223C76
					Unknown flags: 0x1C000000000
	tso_caps:
		max_tso:			0
	rss_caps:
		max_rwq_indirection_tables:			0
		max_rwq_indirection_table_size:			0
		rx_hash_function:				0x0
		rx_hash_fields_mask:				0x0
	max_wq_type_rq:			0
	packet_pacing_caps:
		qp_rate_limit_min:	0kbps
		qp_rate_limit_max:	0kbps
	tag matching not supported
	num_comp_vectors:		4
		port:	1
			state:			PORT_ACTIVE (4)
			max_mtu:		4096 (5)
			active_mtu:		1024 (3)
			sm_lid:			0
			port_lid:		0
			port_lmc:		0x00
			link_layer:		Ethernet
			max_msg_sz:		0x800000
			port_cap_flags:		0x00010000
			port_cap_flags2:	0x0000
			max_vl_num:		1 (1)
			bad_pkey_cntr:		0x0
			qkey_viol_cntr:		0x0
			sm_sl:			0
			pkey_tbl_len:		1
			gid_tbl_len:		1024
			subnet_timeout:		0
			init_type_reply:	0
			active_width:		1X (1)
			active_speed:		10.0 Gbps (8)
			phys_state:		LINK_UP (5)
			GID[  0]:		fe80::c4a2:c9ff:fe03:cf0b, RoCE v2
			GID[  1]:		::ffff:192.168.96.111, RoCE v2
```

## Run perftest with rdma_cm (-R)

[https://github.com/SoftRoCE/rxe-dev/issues/47](https://github.com/SoftRoCE/rxe-dev/issues/47)

```bash
ib_write_bw -d rxe_0 -R
ib_write_bw -d rxe_1 -R 192.168.96.110
```

Update:

25/03/11: install ibverbs-utils, show idv_devinfo

24/11/25: add WSL kernel config steps

24/11/24: add Ubuntu dependencies