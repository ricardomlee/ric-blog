---
description: Podman YES
pubDatetime: 2022-09-04
tags:
- Others
title: Use docker in WSL2 (Arch Linux)
---

Since docker requires systemd to run and WSL2 doesn’t have it, podman is the better option.

Podman is a daemon-free container manager, which is perfect for WSL!

## Install Podman

`podman-compose` is like `docker-compose`, `podman-docker` allows you to run `podman` with docker commands.

```bash
sudo pacman -S podman podman-compose podman-docker
```

## Change registry setting for China

Use `/etc/containers/registries.conf` for global podman setting.

I refered to this post: [podman 配置国内镜像_adi291的博客](https://blog.csdn.net/leave00608/article/details/114156354)

## Run Hello-world!

```bash
sudo podman run hello-world
# or 'sudo docker run hello-world'
```

## Container DNS server

When containers want to communicate with each other with names, a dns server is needed.

Podman uses aardvark-dns as default dns server, before using we need to install it first.

```bash
sudo pacman -S aardvark-dns
```

## TODO

- [ ]  non-sudo user
- [ ]  youki runtime