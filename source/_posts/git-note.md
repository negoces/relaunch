---
title: 「备忘」 git 常用指令
description: 有些指令用的少是真记不住啊
categories: 备忘
tags:
  - git
abbrlink: 1414e7ff
date: 2025-01-06 13:55:00
---

## 设置 HTTPS 代理

```bash
git config --global http.proxy "http://127.0.0.1:7890"
git config --global https.proxy "http://127.0.0.1:7890"
# Query
git config --global --get http.proxy
git config --global --get https.proxy
# Cancel
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## 设置提交用户信息

```bash
# Global
git config --global user.name "User"
git config --global user.email "user@email.com"
# Repo
git config user.name "User"
git config user.email "user@email.com"
# Query
git config user.name
git config user.email
```

## Clone 子模块项目

```bash
# Exist Repo
git submodule init
git submodule update
# Remote Repo
git clone --recursive ${REPO_URL}
```

## 查看 lfs 版本

```bash
git lfs version
```
