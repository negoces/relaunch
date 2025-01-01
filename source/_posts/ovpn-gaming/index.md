---
title: OpenVPN 局域网游戏
abbrlink: af9d3670
---

![OpenVPN](https://img.shields.io/github/v/release/OpenVPN/openvpn?style=flat-square)

- Version: <https://github.com/OpenVPN/openvpn/tags>
- Manpage: <https://build.openvpn.net/man/openvpn-2.6/openvpn.8.html>
- Wiki: <https://community.openvpn.net/openvpn>
- Examples: <https://github.com/OpenVPN/openvpn/tree/master/sample/sample-config-files>
- Client: <https://build.openvpn.net/downloads/releases/latest>

游戏发现多半是二层广播，我们这边采用 Bridge(TAP) 网络，二层和三层的对比：<https://community.openvpn.net/openvpn/wiki/BridgingAndRouting>

## PKI 证书准备

和 Nebula 一样，OpenVPN 需要一个 CA 证书，并为每台机器(无论是服务器还是客户端)签发一张证书，用于双向认证

```bash
# Gen CA Cert & Key
openssl req \
-x509 -new -nodes -sha256 \
-days 365000 -newkey rsa:4096 \
-keyout rootCA.key -out rootCA.crt \
-subj "/C=CN/O=MisakaNet/OU=Network/CN=MisakaOVPN"
openssl x509 -noout -text -in rootCA.crt

# dh
# if slow, download: https://ssl-config.mozilla.org/ffdhe2048.txt
openssl dhparam -out dh2048.pem 2048

# Server
openssl req \
-new -nodes -newkey rsa:4096 \
-keyout server.key -out server.csr \
-subj "/C=CN/O=MisakaNet/OU=Network/CN=OVPN Server"
openssl x509 -req -days 365000 \
-in server.csr -out server.crt \
-CA rootCA.crt -CAkey rootCA.key -CAcreateserial
openssl x509 -noout -text -in server.crt
```
