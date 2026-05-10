---
description: 你们啊，Naive！
pubDatetime: 2023-02-19
tags:
- Others
title: Naive
---

# Use naive to pass wall

This is a technical guide for installing and setting up a server running Caddy, a web server and proxy. The guide provides instructions for installing server oracle linux or server ubuntu, installing caddy and forwardproxy, configuring the Caddyfile, and running Caddy as a daemon.

## Install server oracle linux

### install ssl licence (ref: [Duang's Blog - X-ui面板安装以及ssl申请 (duangvps.com)](https://www.duangvps.com/archives/1776))

```bash
yum update -y 
yum install -y curl 
yum install -y socat

curl https://get.acme.sh | sh

~/.acme.sh/acme.sh --register-account -m user@example.com 

~/.acme.sh/acme.sh --issue -d example.ml --standalone

~/.acme.sh/acme.sh --installcert -d example.ml --key-file /root/private.key --fullchain-file /root/cert.crt
```

### install caddy

```bash
wget https://go.dev/dl/go1.19.linux-amd64.tar.gz
tar -zxvf go1.19.linux-amd64.tar.gz -C /usr/local/

echo export PATH=$PATH:/usr/local/go/bin  >> /etc/profile
source /etc/profile
go version

go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest
~/go/bin/xcaddy build --with github.com/caddyserver/forwardproxy@caddy2=github.com/klzgrad/forwardproxy@naive
cp caddy /usr/bin/
/usr/bin/caddy version
setcap cap_net_bind_service=+ep /usr/bin/caddy
#使用setcap命令设置 /usr/bin/caddy 可以非ROOT用户启动1024以下端口。

bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p

#新建存放Caddyfile的目录
mkdir /etc/caddy/
#使用VI命令在/etc/caddy/目录中，新建Caddyfile配置文件
vi /etc/caddy/Caddyfile

cat /dev/urandom | tr -dc a-zA-Z0-9 | head -c32; echo;
```

```
:443, example.com
tls youremail@example.com
route {
  forward_proxy {
    basic_auth yourname pass
    hide_ip
    hide_via
    probe_resistance
  }
   reverse_proxy https://example2.com {
    header_up Host {upstream_hostport}
  }
}
```

```bash
#格式化配置文件
caddy fmt --overwrite /etc/caddy/Caddyfile
#启动配置文件
caddy run --config /etc/caddy/Caddyfile

# 以后台的方式启动 caddy
caddy start --config  /etc/caddy/Caddyfile
# 以前台的方式启动 caddy
caddy run --config  /etc/caddy/Caddyfile
# 停止
caddy stop 
# reload 配置文件 
caddy reload  --config  /etc/caddy/Caddyfile
# 安装 CA 证书到本地目录
caddy trust
# 格式化 Caddyfile 
caddy fmt --overwrite /etc/caddy/Caddyfile
# 将标准的 Caddyfile 转成 json 格式的等效配置文件， 一般不用这种配置文件
caddy adapt --config  /etc/caddy/Caddyfile --pretty

vi /etc/systemd/system/naive.service
```

```
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Requires=network-online.target

[Service]
Type=notify
User=root
Group=root
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

```bash
#daemon-reload 加载新的 unit 配置文件
systemctl daemon-reload
#enable 创建 unit 配置文件的软链
systemctl enable naive
#start 启动配置文件
systemctl start naive
#status 查看配置文件当前状态
systemctl status naive

ss -tulpn | grep caddy
```

## Install server ubuntu

### install go

```
wget https://go.dev/dl/go1.19.linux-amd64.tar.gz
tar -zxvf go1.19.linux-amd64.tar.gz -C /usr/local/

echo export PATH=$PATH:/usr/local/go/bin  >> /etc/profile
source /etc/profile
go version
```

### install xcaddy

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-xcaddy-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-xcaddy.list
sudo apt update
sudo apt install xcaddy
```

### low-end vm need to use swap

```
dd if=/dev/zero of=/swapfile count=2048 bs=1M
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

#disable
swapoff -a
```

### build

```bash
xcaddy build --with github.com/caddyserver/forwardproxy@caddy2=github.com/klzgrad/forwardproxy@naive
```

### **Run Caddy as a daemon**

```bash
chmod +x caddy
mv caddy /usr/bin/

mkdir /etc/caddy
mv Caddyfile /etc/caddy/

#test
/usr/bin/caddy run --config /etc/caddy/Caddyfile

groupadd --system caddy

useradd --system \
    --gid caddy \
    --create-home \
    --home-dir /var/lib/caddy \
    --shell /usr/sbin/nologin \
    --comment "Caddy web server" \
    caddy
```

Next, create `caddy.service`
 under `/etc/systemd/system/`
 with the following contents:

```
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Requires=network-online.target

[Service]
User=caddy
Group=caddy
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable caddy
systemctl start caddy

systemctl status caddy

systemctl reload caddy
```