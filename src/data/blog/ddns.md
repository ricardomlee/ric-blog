---
description: ''

pubDatetime: 2019-11-03 20:01:05+08:00
tags:
- 笔记
title: 为实验室服务器添加动态域名解析
---

实验室的GPU服务器一般都是通过ssh登录使用的，在自己电脑上打开ssh客户端，输入服务器ip，个人用户的账号密码，就可以在服务器上进行各种骚操作了。
但是目前实验室的服务器还存在一些问题：

1.服务器开关机或者认证校园网后ip可能会被重新分配，这种情况下每个人的ssh客户端保存的ip都可能会失效，需要手动修改ip。
2.服务器通过网线连接到交换机后会被分配一个教育网的“公网ip”，实验室（子网）内的电脑通过这个ip可以访问到服务器。但是未认证校园网的情况下服务器无法访问互联网，因此不能直接使用普通的ddns方法。

### 解决思路：
0. 申请免费域名,添加域名解析服务
域名申请地址：[Freenom](http://freenom.com/)
申请到的域名为：lab1102.ml
域名解析服务：[腾讯云(dnspod)](https://www.dnspod.cn/)
查阅API文档：[API docs](https://www.dnspod.cn/docs/info.html#) 注意：腾讯云官方API已失效，需要使用dnspod的API
添加解析记录：这里仅添加name为 @ 的A记录，即只解析 lab1102.ml
1. 服务器获取本机ip
这里使用一行命令
`IP=$(ip a | grep inet | grep -v 127.0.0.1 | grep -v inet6 | awk '{print $2}' | cut -d "/" -f1)`
2. 服务器登录到一台能访问外网的机器
这台机器需要满足：
(1)长期开启
(2)Linux系统
这里选择Padavan系统的newifi-mini路由器，需要在Padavan后台防火墙中设置允许外网访问ssh服务。
在服务器上生成ssh-key：`ssh-keygen -t rsa`
将公钥放到路由器中：`ssh-copy-id user@newifi`
3. 调用路由器中保存的ddns脚本(参考脚本：[kkkgo](https://github.com/kkkgo/dnspod-ddns-with-bashshell))
```shell
#CONF START
API_ID=123456
API_Token=abcdefghijklmn123456789
domain=lab1102.ml
host=@
#CONF END
. /etc/profile
date
IPREX='([0-9]{1,2}|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\.([0-9]{1,2}|1[0-9][0-9]|2[0-4][0-9]|25[0-5])'
ipcmd="ip addr show";type ip >/dev/null 2>&1||ipcmd="ifconfig"
if [ $# == 1 ]; then
    DEVIP=$1
else
    echo "缺少IP信息"
    exit
fi
if (echo $DEVIP |grep -qEvo "$IPREX");then
DEVIP="Get $DOMAIN DEVIP Failed."
fi
echo "[DEV IP]:$DEVIP"
token="login_token=${API_ID},${API_Token}&format=json&lang=en&error_on_empty=yes&domain=${domain}&sub_domain=${host}"
Record="$(curl -4 -k $(if [ -n "$OUT" ]; then echo "--interface $OUT"; fi) -s -X POST https://dnsapi.cn/Record.List -d "${token}")"
iferr="$(echo ${Record#*code}|cut -d'"' -f3)"
if [ "$iferr" == "1" ];then
record_ip=$(echo ${Record#*value}|cut -d'"' -f3)
echo "[API IP]:$record_ip"
if [ "$record_ip" == "$DEVIP" ];then
echo "IP SAME IN API,SKIP UPDATE."
exit
fi
record_id=$(echo ${Record#*\"records\"\:\[\{\"id\"}|cut -d'"' -f2)
record_line_id=$(echo ${Record#*line_id}|cut -d'"' -f3)
echo Start DDNS update...
ddns="$(curl -4 -k $(if [ -n "$OUT" ]; then echo "--interface $OUT"; fi) -s -X POST https://dnsapi.cn/Record.Ddns -d "${token}&record_id=${record_id}&record_line_id=${record_line_id}&value=$DEVIP")"
ddns_result="$(echo ${ddns#*message\"}|cut -d'"' -f2)"
echo -n "DDNS upadte result:$ddns_result "
echo $ddns|grep -Eo "$IPREX"|tail -n1
else echo -n Get $host.$domain error :
echo $(echo ${Record#*message\"})|cut -d'"' -f2
fi
```
4. 前面三步可以写成一个脚本文件，然后服务器设置定时启动这个脚本
```shell
#!/bin/bash

IP=$(ip a | grep inet | grep -v 127.0.0.1 | grep -v inet6 | awk '{print $2}' | cut -d "/" -f1)

ssh ricjm@115.156.xxx.xxx "./ddns.sh $IP"
```
定时执行脚本：
查看当前用户定时任务 `crontab -l`
修改定时任务 `crontab -e`
crontab任务格式 `* * * * * program` 
前五个参数分别代表分，时，日，月，星期，最后是要执行的任务
例如这里设置每小时的30分时自动执行update的脚本
`30 * * * * /home/lab1102/update.sh`
修改完成保存后cron服务会自动更新

### 实现效果
在实验室内使用域名即可登录到服务器，服务器每小时自动检查ip是否改变，如果发生变化就自动更新dnspod中的解析记录。