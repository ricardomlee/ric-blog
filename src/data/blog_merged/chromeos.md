---
description: ''
pubDatetime: 2020-05-26 17:08:00+08:00
tags:
- 笔记
- ChromeOS
title: NEC二合一平板电脑安装Chrome OS（适用于大部分PC）
---

> 感谢油管老哥的教程！
[Install Chrome OS On Your Laptop / PC Access Google Play and Linux on Chrome!](https://www.youtube.com/watch?v=ROBpXNonVxc)

### 准备文件
1. Linux Mint: https://www.linuxmint.com/edition.php?id=274
一般安装最新版即可，教程中使用的是Linux Mint 19.3 "Tricia" - Cinnamon (64-bit)。
（我的NEC用U盘启动Linux Mint会出现花屏，无法正常使用，可能这个系统缺少某些驱动，于是我重新下载了[Ubuntu 18.04](http://releases.ubuntu.com/18.04/)成功启动。）

2. Rufus: https://rufus.ie/
单文件，直接打开使用。

创建一个文件夹，命名为 “Chrome OS”

3. Brunch Files: https://github.com/sebanc/brunch/releases
下载最新的stable版 .tar.gz文件，将解压后的文件放入Chrome OS文件夹。
此时文件夹中有4个文件，分别为 chromeos-install.sh，efi_legacy.img，efi_secure.img，rootc.img。

4. Install.Sh File: https://raw.githubusercontent.com/shrikant2002/ChromeOS/master/install.sh
下载后放入Chrome OS文件夹

5. Chrome OS Files: https://cros-updates-serving.appspot.com/
4代以及4代以后的intel CPU选择rammus分支最新版；
3代以及之前的intel CPU选择samus分支最新版；
AMD CPU选择grunt分支（成功率较低）。
Install.Sh文件中默认安装的是rammus，需根据下载的系统自行更改相应命令。
系统下载后解压得到.bin文件，将文件名修改为rammus_recovery.bin（根据下载的版本修改），放入Chrome OS文件夹。

最终Chrome OS文件夹内有6个文件。

### 正式安装
1. 准备一个16G以上的U盘，打开Rufus，选择U盘以及Linux Mint（或Ubuntu）的iso镜像文件，点击开始，出现提示选择OK。启动盘制作完成后，将Chrome OS文件夹放入U盘根目录。

2. 打开要安装Chrome OS的PC，进入BIOS关闭Secure Boot，选择UEFI启动方式。插入安装盘并重启，在启动项中选择U盘启动。选择第一项进入Linux系统。

3. 成功进入桌面后连接网络，可能还需要更换apt源，可参考[Ubuntu18.04更换国内源（阿里，网易，中科大，清华等源）](https://www.cnblogs.com/boundless-sky/p/11576373.html)。

4. 在系统菜单中找到GParted，查看系统盘的名称，若为“/dev/sda”，可进行下一步。若系统盘名称不是“/dev/sda”，需要关闭PC，将U盘重新插入Windows电脑，修改Chrome OS下的install.sh文件，将最后一行末尾修改为相应名称，然后重新用U盘启动PC。我的NEC显示的磁盘名称为“/dev/mmcblk0”。

5. 打开文件管理器，进入cdrom文件夹下的Chrome OS文件夹，右键打开terminal，输入`sudo sh install.sh`回车，出现提示，输入yes（确认清空系统盘全部内容）。等待安装完毕重启即可。

（图片待添加）