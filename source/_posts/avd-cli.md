---
title: 「笔记」 使用 Android Studio 命令行工具启动 Android 模拟器
description: 有时候并不需要UI界面
categories: 笔记
tags:
  - Adnroid
abbrlink: 51f48180
date: 2025-03-09 17:00:00
---

> 只做了验证，其实并不好用

## 相关链接

- Android 开发工具 CLI Windows 下载链接: <https://dl.google.com/android/repository/commandlinetools-win-11076708_latest.zip?hl=zh-cn>

## tips

### CLI 工具安装

CLI 工具的放置目录需要套一层 `latest` 文件夹，否则 SDKManager 寻找不到自身路径

```bash
${ANDROID_HOME}
├─cmdline-tools
│  └─latest
│      ├─bin
│      └─lib
├─emulator
└─system-images
```

### SDK 安装

由于不想对系统做过多的改动，所以使用 ps1 脚本启动环境

```powershell
$env:Path = "${PSScriptRoot}\jdk\ms_jdk-21.0.6+7\bin;$env:Path"
$env:Path = "${PSScriptRoot}\cmdline-tools\latest\bin;$env:Path"
$env:Path = "${PSScriptRoot}\emulator;$env:Path"
$env:JAVA_HOME = "${PSScriptRoot}\jdk\ms_jdk-21.0.6+7"
$env:ANDROID_HOME = "${PSScriptRoot}"
$env:ANDROID_USER_HOME = "${PSScriptRoot}"
$env:ANDROID_AVD_HOME = "${PSScriptRoot}\avd"
powershell
```

```bash
# 列出所有可安装的包
sdkmanager.bat --list
# 列出所有已安装的包
sdkmanager.bat --list_installed
# 安装指定的包
sdkmanager.bat "system-images;android-35;google_apis_playstore;x86_64"
```

### 创建 AVD 设备

```bash
# 列出所有 AVD 设备
avdmanager.bat list avd
# 创建 AVD 设备
avdmanager.bat create avd -n avd_name -d pixel_tablet -c 32768M -k "system-images;android-35;google_apis_playstore;x86_64"
# 删除 AVD 设备
avdmanager.bat delete avd -n avd_name
```

### 启动 AVD 设备

```bash
emulator.exe -avd avd_name
```
