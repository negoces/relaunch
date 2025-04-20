---
title: 「笔记」 OpenVPN 局域网游戏
description: 使用 OpenVPN 组件二层网络
categories: 笔记
tags:
  - OpenVPN
  - Network
abbrlink: af9d3670
date: 2025-01-06 18:20:00
---

## 相关资料

![OpenVPN](https://img.shields.io/github/v/release/OpenVPN/openvpn?style=flat-square)

- Version: <https://github.com/OpenVPN/openvpn/tags>
- Manpage: <https://build.openvpn.net/man/openvpn-2.6/openvpn.8.html>
- Wiki: <https://community.openvpn.net/openvpn>
- Examples: <https://github.com/OpenVPN/openvpn/tree/master/sample/sample-config-files>
- Client: <https://build.openvpn.net/downloads/releases/latest>

游戏发现多半是二层广播，我们这边采用 Bridge(TAP) 网络，二层和三层的对比：<https://community.openvpn.net/openvpn/wiki/BridgingAndRouting>

## 安装

```bash
# Arch
sudo pacman -Sy openvpn
# Debian
sudo apt install openvpn
sudo apt install policykit-1
```

## 生成 PKI 证书

和 Nebula 一样，OpenVPN 需要一个 CA 证书，并为每台机器(无论是服务器还是客户端)签发一张证书，用于双向认证

```bash
# 生成 CA 证书
openssl req -new -x509 -days 365250 -noenc -sha256 -newkey ed25519 \
-subj "/C=CN/O=MisakaNet/OU=Network/CN=MisakaOVPN CA" \
-keyout rootCA.key -out rootCA.crt
openssl x509 -noout -text -in rootCA.crt

# 修改环境变量，可生成不同节点的证书
export OVPN_NODE_NAME="server"
openssl x509 -req -days 365250 -CA rootCA.crt -CAkey rootCA.key -CAcreateserial \
-in <(openssl req -new -noenc -newkey ed25519 \
-keyout ${OVPN_NODE_NAME}.key \
-subj "/C=CN/O=MisakaNet/OU=Network/CN=OVPN ${OVPN_NODE_NAME}") \
-out ${OVPN_NODE_NAME}.crt
openssl x509 -noout -text -in ${OVPN_NODE_NAME}.crt
openssl verify -CAfile rootCA.crt ${OVPN_NODE_NAME}.crt

# dh
# if slow, download: https://ssl-config.mozilla.org/ffdhe2048.txt
openssl dhparam -out dh2048.pem 2048
```

## 配置服务端

在 `/etc/openvpn/server` 文件夹中创建配置文件，并放入证书，例如：

```bash
/etc/openvpn/server
├── access.conf
├── Access.crt
├── Access.key
├── dh2048.pem
└── rootCA.crt
```

示例的配置文件为：`/etc/openvpn/server/access.conf`

```conf
mode server
proto udp
port 1194
dev tapovpn
ifconfig 10.25.0.1 255.255.255.0
keepalive 10 120
verb 3

server-bridge 10.25.0.1 255.255.255.0 10.25.0.2 10.25.0.254
client-to-client
duplicate-cn

cipher AES-256-GCM

ca rootCA.crt
cert Access.crt
key Access.key
dh dh2048.pem
```

启动，以及设置自启。`access` 替换为自己的配置文件名称

```bash
sudo systemctl start openvpn-server@access
sudo systemctl status openvpn-server@access
sudo systemctl enable openvpn-server@access
```

## 配置客户端

在 `/etc/openvpn/client` 文件夹中创建配置文件，并放入证书，例如：

```bash
/etc/openvpn/client
├── rootCA.crt
├── turing.conf
├── Turing.crt
└── Turing.key
```

示例的配置文件为：`/etc/openvpn/client/turing.conf`

```conf
remote ddns.hf.rootless.cc 1194
proto udp
connect-retry 5 60
resolv-retry infinite
nobind
dev tapovpn
keepalive 10 120
verb 3

client

cipher AES-256-GCM

ca rootCA.crt
cert Turing.crt
key Turing.key
```

启动，以及设置自启。`turing` 替换为自己的配置文件名称

```bash
sudo systemctl start openvpn-client@turing
sudo systemctl status openvpn-client@turing
sudo systemctl enable openvpn-client@turing
```
