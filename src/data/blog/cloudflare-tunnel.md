---
description: A free alternative for frp

pubDatetime: 2022-03-20
tags:
- Others
title: Use Cloudflare Tunnel for NAS
---

## Download Cloudflared

[Downloads · Cloudflare Zero Trust docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/)

Download the binary for your NAS regarding your arch

mov it to an executable path like /usr/local/bin/cloudflare (bin)

## Login

```bash
cloudflared tunnel login
```

open browser to login

## Create a Tunnel

```bash
cloudflared tunnel create <tunnel name>
```

## Add DNS Record

go to your DNS dashboard, add a CNAME record, name your app, for example, app0. content is <tunnel_uuid>.cfargotunnel.com

## Create a Config File

create a config file in ~/.cloudflared/config.yaml

```bash
tunnel: <tunnel name or uuid>
credentials-file: /root/.cloudflared/<tunnel uuid>.json

ingress:
  - hostname: app0.<domain>
    service: http://localhost:80
  - hostname: app1.<domain>
    service: http://localhost:8096
  - hostname: app2.<domain>
    service: http://localhost:9091
  - service: http_status:404
```

## Run Your Tunnel

```bash
cloudflared tunnel --config ~/.cloudflared/config.yaml run

# to run in backgroud
nohup cloudflared tunnel --config ~/.cloudflared/config.yaml run >> cfd.log 2>&1 &
```

## Enjoy the Traveling

now open your browser and type “app0.<domain>”