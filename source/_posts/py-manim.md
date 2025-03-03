---
title: 「笔记」 Manim 试水
description: 3Blue1Brown 同款，绘制你自己的数学动画
categories: 笔记
tags:
  - Python
abbrlink: 94b9047d
date: 2025-03-03 21:20:00
---

## 目标

使用 Manim 绘制一个简单的数轴动画

## 过程

```bash
mkdir -p manim
cd manim

micromamba create -n manim
micromamba activate manim
micromamba install -c conda-forge manim

manim init project sample --default

cd sample
manim -p -r 1920,1080 main.py DefaultTemplate
```

## Reference

Guide: <https://github.com/cai-hust/manim-tutorial-CN>
