---
description: ''

pubDatetime: 2020-01-01
tags:
- ''
title: cloudflare cdn vless + select ip + v2raya routing
---

# Cloudflare CND VLESS

code: https://github.com/zizifn/edgetunnel

Go to clouflare → workers & pages

create application → create worker → deploy

quick edit, copy and replace the code

generate new uuid and replace “userID”

let proxyIP = '[cdn.anycast.eu.org](http://cdn.anycast.eu.org/)';

save and deploy

Custom Domains → view

add Custom Domains

add xx.xxx.link

open xx.xxx.link/<uuid>

copy the vless link

go to your v2ray client, import ndoe

if not use tls

unchoose tls, change port to 80

# Select IP

read: https://github.com/XIU2/CloudflareSpeedTest

# v2raya routing

in v2raya, add new outbound, like “nologin”, choose the cdn node

the original proxy use self node

设置 → **规则端口的分流模式 → RoutingA**

配置 → add the site need to use cdn

`domain(geosite:youtube)->nologin`