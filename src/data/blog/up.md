---
description: ''

pubDatetime: 2020-03-21 18:28:09+08:00
tags:
- 笔记
title: Unraid+Transmission 记录第一次制种
---

unraid-docker-transmission 
右键，打开Console
<!-- Image missing: C2cJgLpUZmbRd5T.png -->

transmission-create 命令说明

命令行语法
```shell
transmission-create [-h]
transmission-create -V
transmission-create [-p] [-o file] [-c comment] [-t tracker] <source file or directory>
```
选项参数说明
-h, --help : 显示一个帮助后退出
-p, --private : 包含私人追踪信息(private trackers)的bt种子标志，即pt种子
-c, --comment : 为种子文件添加一个说明
comment ： 一段字符串，是需要添加的说明内容
-t, --tracker ：为种子添加追踪用的网址，大多数种子至少需要一个追踪用的网址（通常是发布地址）。要添加不止一个，可以多次使用这个选项
tracker : 一段URI字符串，是需要添加的追踪用网址
source file or directory : 要提供下载的源文件或者目录
-o ： 种子输出到文件,如果没有-o选项，将输出到标准输出设备，可以用来测试
file ： 种子文件路径
-V, --version ： 显示版本信息后退出
例子
```shell
transmission-create -p -o /tmp/test.torrent -c "a test BitToorent seed file" -t http://www.test1.me/ -t http://www.testbt.me/ test.txt
```

<!-- Image missing: TIuEGdCUOfVQpH1.png -->