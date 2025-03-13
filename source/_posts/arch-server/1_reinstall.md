---
title: 「笔记」 Arch Linux 服务器「一」：重生
description: 'XFS: log I/O error -5'
categories: 笔记
tags:
  - "Arch Linux"
  - "Server"
  - "Headless"
abbrlink: '97325808'
date: 2025-03-11 22:45:00
---

## 死亡回放

我有一台 `Z2 Air Series GK5CP5X` 笔记本，大学后就闲置了。但因为性能还行，就一直作为一台服务器在运行，期间也没出啥问题，直到今年过年。

人还在老家，突然发现远程连不上了，由于重要服务不在上面，也没怎么关心。过两天回来一看 `XFS: log I/O error -5` 就 xfs_repair 了一下。没过几天，又 error 了，只能找块盘重装一下了，刚好在这做一下记录。

先留一个 fastfetch 在这

```bash
root@archiso ~ # fastfetch
                  -`                     root@archiso
                 .o+`                    ------------
                `ooo/                    OS: Arch Linux x86_64
               `+oooo:                   Host: Z2 Air Series GK5CP5X (v1.1)
              `+oooooo:                  Kernel: Linux 6.11.5-arch1-1
              -+oooooo+:                 Uptime: 1 hour, 5 mins
            `/:-:++oooo+:                Packages: 405 (pacman)
           `/++++/+++++++:               Shell: zsh 5.9
          `/++++++++++++++:              Display (BOE0747): 1920x1080 @ 60 Hz in 16’ [Built-in]
         `/+++ooooooooooooo/`            Terminal: /dev/pts/0
        ./ooosssso++osssssso+`           CPU: Intel(R) Core(TM) i7-9750H (12) @ 4.50 GHz
       .oossssso-````/ossssss+`          GPU 1: NVIDIA GeForce GTX 1650 Mobile / Max-Q [Discrete]
      -osssssso.      :ssssssso.         GPU 2: Intel UHD Graphics 630 @ 1.15 GHz [Integrated]
     :osssssss/        osssso+++.        Memory: 1.86 GiB / 31.21 GiB (6%)
    /ossssssss/        +ssssooo/-        Swap: Disabled
  `/ossssso+/:-        -:/+osssso+-      Disk (/): 11.74 MiB / 256.00 MiB (5%) - overlay
 `+sso+:-`                 `.-/+oso:     Local IP (enp4s0): 192.168.1.20/24
`++:.                           `-/+/    Battery (standard): 100% [AC Connected]
.`                                 `/    Locale: C.UTF-8
```

## 安装 | Install

### 准备

1. 进入 Arch Linux LiveCD
1. 关闭自动查找镜像
    ```bash
    systemctl stop reflector.service
    ```
1. 设置安装镜像
    ```bash
    tee /etc/pacman.d/mirrorlist <<EOF
    Server = https://mirrors.ustc.edu.cn/archlinux/\$repo/os/\$arch
    EOF
    ```

### 分区与格盘

1. 根据下表对磁盘进行分区
    | 设备 | 文件系统 | 大小 | 挂载点 | 类型(fdisk) |
    |:-:|:-:|:-:|:-:|:-:|
    | nvme1n1p1 | fat | 1024MB | /boot | 1(EFI) |
    | nvme1n1p2 | xfs | ALL | /boot | 20(Linux) |
1. 设置环境变量，后期有用
    ```bash
    export DEV_EFI="/dev/nvme1n1p1"
    export DEV_ROOT="/dev/nvme1n1p2"
    ```
1. 格式化磁盘
    ```bash
    mkfs.vfat ${DEV_EFI} -n EFI -v
    mkfs.xfs ${DEV_ROOT} -f
    ```

### 挂载及安装

1. 挂载磁盘
    ```bash
    mount --mkdir ${DEV_ROOT} /mnt
    mount --mkdir ${DEV_EFI} /mnt/boot
    df -hT | grep /mnt
    ```
1. 安装系统
    ```bash
    pacman -Syy
    pacman -S archlinux-keyring --noconfirm --needed
    pacstrap -K /mnt \
    base linux-firmware \
    linux-zen linux-zen-headers \
    intel-ucode amd-ucode \
    nftables sudo vim \
    xfsprogs btrfs-progs \
    efibootmgr openssh
    ```

### 配置系统

1. chroot
    ```bash
    arch-chroot /mnt
    ```
1. 设置时区
    ```bash
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    hwclock --systohc
    ```
1. 设置地区
    ```bash
    sed -i 's/.*\(zh_CN\.UTF-8 UTF-8\).*/\1/g' /etc/locale.gen
    sed -i 's/.*\(en_US\.UTF-8 UTF-8\).*/\1/g' /etc/locale.gen

    locale-gen

    tee /etc/locale.conf <<EOF
    LANG=zh_CN.UTF-8
    EOF
    ```
1. 配置主机名
    ```bash
    export HOST_NAME="Turing"
    tee /etc/hostname <<EOF
    ${HOST_NAME}
    EOF
    tee /etc/hosts <<EOF
    127.0.0.1   localhost
    ::1         localhost
    127.0.0.1   ${HOST_NAME}
    EOF
    ```
1. 配置网络
    ```bash
    tee /etc/systemd/network/01-dhcp.network <<EOF
    [Match]
    Name = e*

    [Network]
    DHCP = yes
    EOF

    tee /etc/resolv.conf <<EOF
    nameserver 119.29.29.29
    options edns0 trust-ad
    search .
    EOF
    ```
1. 修改电源策略，若当前系统已启动，需要重启 logind
    ```bash
    # HandleLidSwitch -> ignore
    # HandleLidSwitchExternalPower -> ignore
    sed -i 's/^#\?HandleLidSwitch=.*/HandleLidSwitch=ignore/g' /etc/systemd/logind.conf
    sed -i 's/^#\?HandleLidSwitchExternalPower=.*/HandleLidSwitchExternalPower=ignore/g' /etc/systemd/logind.conf
    ```
1. 创建用户
    ```bash
    export NEW_USER="neko"
    /usr/sbin/useradd -m -U -G wheel ${NEW_USER} && passwd ${NEW_USER}

    mkdir -p /home/${NEW_USER}/.ssh
    tee /home/${NEW_USER}/.ssh/authorized_keys <<EOF
    ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILDDsjpT5cawNYpT1eWXgDBGqA5WyNe+PF4R/uKTZxgn neko@Hyper
    EOF

    chown -R ${NEW_USER}:${NEW_USER} /home/${NEW_USER}/.ssh
    chmod 700 /home/${NEW_USER}/.ssh
    chmod 600 /home/${NEW_USER}/.ssh/authorized_keys

    pacman -Sy bash-completion
    tee /home/${NEW_USER}/.bashrc <<EOF
    [[ \$- != *i* ]] && return
    source /usr/share/bash-completion/bash_completion
    alias ls='ls -lh --color=auto'
    alias grep='grep --color=auto'
    alias ip='ip -c'
    if [ -n "\$BASH_VERSION" ]; then
        export PS1='\[\e[01;32m\]\u@\h\[\e[00m\]:\[\e[01;34m\]\w\[\e[00m\]\\$ '
    else
        if [ "\$UID" -eq 0 ]; then
            export PROMPT='%F{10}%n@%m%f:%F{12}%~%f%# '
        else
            export PROMPT='%F{10}%n@%m%f:%F{12}%~%f\\$ '
        fi
    fi
    export PS1="\$PS1\[\e]1337;CurrentDir="'\$(pwd)\a\]'
    EOF
    ```
1. 修改sudo配置
    ```bash
    tee /etc/sudoers.d/wheel <<EOF
    Defaults env_reset,pwfeedback
    %wheel ALL=(ALL:ALL) ALL
    EOF
    ```
1. 修改 pacman 配置
    ```bash
    sed -i 's/^#\?Color.*/Color/g' /etc/pacman.conf
    sed -i 's/^#\?VerbosePkgLists.*/VerbosePkgLists/g' /etc/pacman.conf
    sed -i 's/^#\?ParallelDownloads.*/ParallelDownloads = 8/g' /etc/pacman.conf
    ```
1. 修改 ssh 端口
    ```bash
    # Port 51022
    sed -i 's/^#\?Port.*/Port 51022/g' /etc/ssh/sshd_config
    ```
1. 启用和禁用服务
    ```bash
    systemctl enable systemd-networkd.service
    systemctl enable sshd.service
    ```

### 安装引导

```bash
mkdir -p /etc/cmdline.d
tee /etc/cmdline.d/root.conf <<EOF
root=UUID=$(/usr/sbin/blkid -s UUID -o value ${DEV_ROOT}) rw loglevel=3
EOF

cp -v /etc/mkinitcpio.d/linux-zen.preset{,.bak}
tee /etc/mkinitcpio.d/linux-zen.preset <<EOF
ALL_kver="/boot/vmlinuz-linux-zen"
PRESETS=('default')
default_uki="/boot/EFI/Linux/arch-linux-zen.efi"
EOF
mkdir -p /boot/EFI/Linux
mkinitcpio -P

rm 
```

```bash
efibootmgr -B -b 0000
efibootmgr --create --disk /dev/nvme1n1 --part 1 --label "Arch Linux Zen" --loader '\EFI\Linux\arch-linux-zen.efi' --unicode
```

## 注意事项

由于某些原因，有些地方还可以优化

1. 建议将 ESP 挂载到 `/efi` 而不是 `/boot`，否则内核全部会保存到 EFI 分区
