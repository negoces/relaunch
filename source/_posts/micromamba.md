---
title: 「笔记」 Micromamba 试水
description: 貌似是个 Conda 替代品
date: 2025-02-27 22:20:00
abbrlink: b6638d5f
categories: 笔记
tags: [Python]
---

## Windows 手动安装

Reference: <https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html#manual-installation>

1. 前往: <https://github.com/mamba-org/micromamba-releases/releases/latest>
1. 手动下载二进制文件: `micromamba-win-64.exe` 改名 `micromamba.exe`
1. 将二进制文件放入用户程序文件夹: 比如，`C:\UserProgram\bin` 并加入环境变量
1. 查看帮助:
    ```
    .\micromamba.exe --help
    ```
1. 使用:
    1. 临时注册使用:
        ```pwsh
        $Env:MAMBA_ROOT_PREFIX="C:\UserProgram\Mamba"
        .\micromamba.exe shell hook -s powershell | Out-String | Invoke-Expression
        ```
    1. 持久注册到 `profile.ps1`
        ```pwsh
        .\micromamba.exe shell init -s powershell -r C:\UserProgram\Mamba
        ```
1. 配置镜像: `C:\UserProgram\Mamba\.mambarc`
    ```yaml
    channels:
      - defaults
    show_channel_urls: true
    default_channels:
      - https://mirror.nju.edu.cn/anaconda/pkgs/main
      - https://mirror.nju.edu.cn/anaconda/pkgs/r
      - https://mirror.nju.edu.cn/anaconda/pkgs/msys2
    custom_channels:
      conda-forge: https://mirror.nju.edu.cn/anaconda/cloud
      pytorch: https://mirror.nju.edu.cn/anaconda/cloud
    ```

> 解决 `python` 指令跳转应用商店的问题:
>
> 1. 前往 `设置` -> `应用` -> `高级应用设置` -> `应用执行别民`
> 1. 关闭 `应用安装程序(python.exe)`

## 示例:

### 创建一个 Python 3.12 环境

```bash
micromamba create -n py312
micromamba activate py312
micromamba install python=3.12
python --version
micromamba deactivate
micromamba remove -n py312 --all
```
