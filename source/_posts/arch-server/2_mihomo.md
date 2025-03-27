---
title: 「笔记」 Arch Linux 服务器「二」：开眼
description: 'GET 200 OK'
categories: 笔记
tags:
  - Arch Linux
  - Server
  - Headless
abbrlink: c30d47d9
date: 2025-03-19 21:45:00
---

## 配置 Mihomo

### 下载 mihomo

从 mihomo 的 Release 页面下载最新版本 `mihomo-linux-amd64-{version}.gz`

![GitHub Release](https://img.shields.io/github/v/release/MetaCubeX/mihomo?display_name=tag&style=flat-square&label=mihomo)

- <https://github.com/MetaCubeX/mihomo/releases/latest>

例如当前版本为 `v1.19.3`，下载链接为：

- <https://github.com/MetaCubeX/mihomo/releases/download/v1.19.3/mihomo-linux-amd64-v1.19.3.gz>

若无法访问 Github，可通过加速链接下载：

- <https://ghfast.top/https://github.com/MetaCubeX/mihomo/releases/download/v1.19.3/mihomo-linux-amd64-v1.19.3.gz>

若链接失效，则可通过 <https://ghproxy.link/> 获取

### 安装 mihomo

```bash
gunzip mihomo-linux-amd64-v1.19.3.gz
chmod 755 mihomo-linux-amd64-v1.19.3

sudo mkdir -p /opt/mihomo
sudo mv mihomo-linux-amd64-v1.19.3 /opt/mihomo

cd /opt/mihomo
sudo ln -s -T mihomo-linux-amd64-v1.19.3 mihomo
./mihomo -v
```

### 配置文件

- 文件位置: `/opt/mihomo/config.yaml`
- 其中替换 Dashboard 为 Zashboard

![GitHub Release](https://img.shields.io/github/v/release/Zephyruso/zashboard?display_name=tag&style=flat-square&label=zashboard)

```yaml
allow-lan: true
bind-address: "*"
mode: rule
log-level: info
ipv6: true
external-controller: 0.0.0.0:9090
external-controller-cors:
  allow-origins:
    - '*'
  allow-private-network: true
external-ui: .
external-ui-name: zd
external-ui-url: "https://github.com/Zephyruso/zashboard/archive/refs/heads/gh-pages.zip"
profile:
  store-selected: true
unified-delay: true
tcp-concurrent: true
global-client-fingerprint: chrome
geodata-loader: memconservative
geo-auto-update: true
geo-update-interval: 24
geox-url:
  geoip: "https://testingcf.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geoip.dat"
  geosite: "https://testingcf.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geosite.dat"
  mmdb: "https://testingcf.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@release/geoip.metadb"
etag-support: true

dns:
  cache-algorithm: arc
  enable: true
  prefer-h3: true
  default-nameserver:
    - 119.29.29.29
    - system
  ipv6: true
  nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query
  proxy-server-nameserver:
    - https://doh.pub/dns-query
    - https://dns.alidns.com/dns-query

sniffer:
  enable: true
  force-dns-mapping: true
  parse-pure-ip: true
  override-destination: true
  sniff:
    HTTP:
      ports: [80]
      override-destination: true
    TLS:
      ports: [443]
    QUIC:
      ports: [443]

mixed-port: 7890
tproxy-port: 7880
```

至此，基础配置已设置完毕，剩下填入自己的节点配置即可。验证配置：

```bash
sudo /opt/mihomo/mihomo -d /opt/mihomo -t
```

WebUI 界面在： `http://ip:9090/ui/zd`

需要自己填入 API IP 及端口或者使用 GET 参数：`http://host:port/ui/zd/#/setup?hostname=ipordomain&port=9090&secret=123456`

例如：<http://192.168.1.20:9090/ui/zd/#/setup?hostname=192.168.1.20&port=9090>

### 安装服务

- 创建用户，并记住 uid/gid，用于 nftables 分流，防止流量回环

```bash
sudo useradd -r -M -s /usr/sbin/nologin mihomo
sudo -u mihomo id
```

- 创建 `/etc/systemd/system/mihomo.service`

```ini
[Unit]
Description=Mihomo Proxy Daemon
After=network.target systemd-networkd.service

[Service]
User=mihomo
Group=mihomo
Type=simple
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE
ExecStart=/opt/mihomo/mihomo -d /opt/mihomo
ExecReload=/bin/kill -HUP 
Restart=on-failure
RestartSec=7s

[Install]
WantedBy=multi-user.target
```

- 设置自启并启动

```bash
sudo systemctl enable --now mihomo
```

## 透明代理

> !!! 透明代理会修改 nftables 配置，若本机设置过 nftables，请勿直接使用本文设置，请酌情融合配置

### 模块化 nftables 配置

将 nftables 配置模块化，方便以后实现插装设计

- `/etc/nftables.conf`

```conf
#!/usr/sbin/nft -f
flush ruleset
include "/etc/nftables.d/S*.rules"
```

### 获取大陆IP表

写入并运行一次

- `/etc/nftables.d/C00-update_cnsets.sh`

```bash
#!/bin/bash

if [ ! -f "/tmp/delegated-apnic-latest" ]; then
    curl -fL "https://ftp.nic.ad.jp/mirror/ftp.apnic.net/pub/apnic/stats/apnic/delegated-apnic-latest" -o /tmp/delegated-apnic-latest
fi

sudo mkdir -p /etc/nftables.d

cat /tmp/delegated-apnic-latest | \
awk -F '|' '/CN/&&/ipv4/ {print $4 "/" 32-log($5)/log(2)}' | \
sed ':label;N;s/\n/, /;b label' | \
sed 's/$/& }}/g' | \
sed 's/^/set chinalist_v4 {\n    type ipv4_addr; flags interval;\n    elements = { /' | \
sudo tee /etc/nftables.d/chinalist_v4.nftsets > /dev/null

cat /tmp/delegated-apnic-latest | \
awk -F '|' '/CN/&&/ipv6/ {print $4 "/" $5}' | \
sed ':lable;N;s/\n/, /;b lable' | \
sed 's/$/& }}/g' | \
sed 's/^/set chinalist_v6 {\n    type ipv6_addr; flags interval;\n    elements = { /' | \
sudo tee /etc/nftables.d/chinalist_v6.nftsets > /dev/null

rm /tmp/delegated-apnic-latest
```

### 写入保留地址表

- `/etc/nftables.d/reserved.nftsets`

```conf
set reserved_v4 {
    type ipv4_addr;
    flags interval;
    elements = {
        0.0.0.0/8,
        10.0.0.0/8,
        100.64.0.0/10,
        127.0.0.0/8,
        169.254.0.0/16,
        172.16.0.0/12,
        192.0.0.0/24,
        192.0.2.0/24,
        192.88.99.0/24,
        192.168.0.0/16,
        198.18.0.0/15,
        198.51.100.0/24,
        203.0.113.0/24,
        224.0.0.0/4,
        240.0.0.0 - 255.255.255.255
    }
}

set reserved_v6 {
    type ipv6_addr;
    flags interval;
    elements = {
        ::/128,
        ::1/128,
        ::ffff:0:0/96,
        ::ffff:0:0:0/96,
        64:ff9b::/96,
        64:ff9b:1::/48,
        100::/64,
        2001::/32,
        2001:20::/28,
        2001:db8::/32,
        2002::/16,
        3ffe::/16,
        fc00::/7,
        fe80::/10,
        ff00:: - ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff
    }
}
```

### 透明代理规则

后面使用 service 实现动态加载，所以用 M 前缀

- `/etc/nftables.d/M01-tproxy.rules`

```conf
#!/usr/sbin/nft -f

define TPROXY_PORT = 7880;
define TPROXY_USER = mihomo;
define TPROXY_MARK = 0x0300;
define BYPASS_MARK = 0x0200;

destroy table inet proxy
table inet proxy {
    include "/etc/nftables.d/chinalist_v4.nftsets"
    include "/etc/nftables.d/chinalist_v6.nftsets"
    include "/etc/nftables.d/reserved.nftsets"

    chain output {
        type route hook output priority filter; policy accept;
        meta l4proto { tcp, udp } jump output_proxy
    }

    chain output_proxy {
        skuid $TPROXY_USER        return
        ip  daddr @reserved_v4    return
        ip6 daddr @reserved_v6    return
        ip  daddr @chinalist_v4   return
        ip6 daddr @chinalist_v6   return
        meta l4proto { tcp, udp } meta mark set $TPROXY_MARK return
    }

    chain prerouting {
        type filter hook prerouting priority mangle; policy accept;
        meta l4proto { tcp, udp } jump preroute_proxy
    }

    chain preroute_proxy {
        meta l4proto tcp socket transparent 1 meta mark set $TPROXY_MARK return

        ip  daddr @reserved_v4    return
        ip6 daddr @reserved_v6    return
        ip  daddr @chinalist_v4   return
        ip6 daddr @chinalist_v6   return
        meta l4proto { tcp, udp } meta mark set $TPROXY_MARK tproxy to :$TPROXY_PORT return
    }
}
```

再写个删除规则的配置

- `/etc/nftables.d/D01-tproxy.rules`

```conf
#!/usr/sbin/nft -f

destroy table inet proxy
```

> 部分老版本系统不支持 `destroy` 关键字，请将 `destroy table inet proxy`  
> 变为 `table inet proxy` 和 `delete table inet proxy` 两条指令

### 安装透明代理服务

首先得把部分系统参数写入内核

- `/etc/sysctl.d/01-net.conf`

```ini
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
net.ipv4.conf.all.route_localnet = 1
```

立即生效: `sudo sysctl -p /etc/sysctl.d/01-net.conf`

service 服务: `/etc/systemd/system/tproxy.service`

```ini
[Unit]
Description= TProxy Route
Documentation= https://www.netfilter.org/projects/nftables/manpage.html
Requires=mihomo.service
After=mihomo.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStartPost=ip rule add fwmark 0x0300 table 30 proto kernel
ExecStartPost=ip route add local default dev lo table 30 proto kernel
ExecStartPost=ip -6 rule add fwmark 0x0300 table 30 proto kernel
ExecStartPost=ip -6 route add local default dev lo table 30 proto kernel
ExecStart=nft -f /etc/nftables.d/M01-tproxy.rules
ExecStop=nft -f /etc/nftables.d/D01-tproxy.rules
ExecStopPost=ip route del local default dev lo table 30
ExecStopPost=ip rule del fwmark 0x0300 table 30
ExecStopPost=ip -6 route del local default dev lo table 30
ExecStopPost=ip -6 rule del fwmark 0x0300 table 30

[Install]
WantedBy=multi-user.target
```

操作指令

```bash
# 立即启动并设置开机自启
sudo systemctl enable --now tproxy
```
