---
title: 「笔记」 给 Hexo 添加 abbrlink 功能
description: 使用 abbrlink 插件为 Hexo 博客站点的文章添加永久链接功能
date: 2025-01-01 18:30:00
abbrlink: db69886d
categories: 笔记
tags: [hexo]
---

## 何为 abbrlink

Qwen2.5-72B 的回复：

> "abbrlink" 是一种用于生成文章或页面唯一标识符的插件或工具，常见于博客系统或内容管理系统中。它通过将文章的标题或其他信息转换为一个简短且具有可读性的字符串，来作为文章的链接标识。这种方式生成的链接不仅便于用户记忆和分享，还能保持链接的整洁和美观。

总结来说就是短链接，便于记忆这一点我不是很认同，但是便于分享是真的，能保证你文章标题发生变化时，文章的链接依旧不变。

## 如何使用

- 项目地址: <https://github.com/JunKuangKuang/hexo-abbrlink3>

### 安装插件

```bash
pnpm install hexo-abbrlink3 --save
```

### 配置插件

编辑 `_config.yml`

```diff
- permalink: :year/:month/:day/:title/
+ permalink: p/:abbrlink/

+ abbrlink:
+   alg: crc32      #support crc16(default) and crc32
+   rep: hex        #support dec(default) and hex
+   #allow: ['post','school','cpp',"java","blog"] #Allowed template types. The default is (" post ")
+   #disallow: [] #Unallowed template type. Default is an empty list
```

修改完后再运行就好了，就比如这篇文章的永久链接是： [db69886d](/p/db69886d/)

立个 flag，下次说说怎么给 Hugo 添加类似 abbrlink 的功能
