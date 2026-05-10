---
description: Caddy ❤️ Warp

pubDatetime: 2023-04-17
tags:
- Others
title: “Warp” Around Naive / V2ray
---

## Setup Warp (Ubuntu 22.04)

```bash
curl https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list
sudo apt update
sudo apt install cloudflare-warp

systemctl status warp-svc.service

warp-cli register
warp-cli set-mode proxy
warp-cli connect
# the proxy opened at socks5://127.0.0.1:40000

# reconnect
warp-cli  disconnect && sleep 1  &&warp-cli connect

# status
warp-cli status
warp-cli warp-stats
```

## Set upstream for Naiveproxy

install [naive](https://ricli.ml/naive)

edit Caddyfile

```
:443, xxx.xxx.xxx
tls /etc/caddy/cert.crt /etc/caddy/private.key
route {
        forward_proxy {
                basic_auth xxx xxxxxx
                hide_ip
                hide_via
                probe_resistance
                upstream        socks5://127.0.0.1:40000
        }
        reverse_proxy https://xxx.ml {
                header_up Host {upstream_hostport}
        }
}
```

restart caddy

```bash
sudo systemctl restart caddy
```

## Set forward proxy for v2ray

edit config

```bash
sudo vim /usr/local/etc/v2ray/config.json
```

add "sniffing" to inbound, add socks proxy to outbound, add “routing”:

```bash
{
  "inbounds": [
		{
	    ...,
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      }
		}
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "warp",
      "protocol": "socks",
      "settings": {
        "servers": [
          {
            "address": "127.0.0.1",
            "port": 40000,
            "users": []
          }
        ]
      }
    }
  ],
    "routing": {
       "rules": [
        {
          "type": "field",
          "domain": [
              "openai.com",
              "ai.com",
              "geosite:netflix"
          ],
          "outboundTag": "warp"
       }
      ]
  }
}
```

restart v2ray

```bash
sudo systemctl restart v2ray
```