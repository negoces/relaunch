---
title: 「笔记」 Docker 部署 Mosquitto
description: 使用 Docker 部署 Mosquitto MQTT broker
categories: 笔记
tags:
  - docker
  - mosquitto
abbrlink: b923dd5e
date: 2025-01-01 22:00:00
---

> Eclipse Mosquitto is an open source (EPL/EDL licensed) message broker that implements the MQTT protocol versions 5.0, 3.1.1 and 3.1. Mosquitto is lightweight and is suitable for use on all devices from low power single board computers to full servers.
>
> Eclipse Mosquitto 是一个开源（EPL/EDL 许可）的消息代理，实现了 MQTT 协议的 5.0、3.1.1 和 3.1 版本。Mosquitto 轻量级，适用于从低功耗单板计算机到全功能服务器的各种设备。

## Docker Compose

```yaml
# Port: 1883
services:
  mosquitto:
    image: eclipse-mosquitto:2
    container_name: mosquitto
    network_mode: host
    restart: unless-stopped
    environment:
      TZ: "Asia/Shanghai"
    volumes:
      - ./config:/mosquitto/config
      - ./data:/mosquitto/data
```

## Configure

`./config/mosquitto.conf`

```ini
allow_anonymous true
listener 1883 0.0.0.0
protocol websockets
persistence true
persistence_location /mosquitto/data
log_dest stdout
```

## 其他资源

- 调试工具:
  - Serial-Studio: <https://github.com/Serial-Studio/Serial-Studio>
  - MQTTX: <https://github.com/emqx/MQTTX>
- 其他替代:
  - NanoMQ: <https://github.com/nanomq/nanomq>
