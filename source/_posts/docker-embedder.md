---
title: 「笔记」 Docker 部署 BGE-M3 Embedder 模型
description: 使用 Docker 部署 llama.cpp 运行 BGE-M3 Embedder 模型
categories: 笔记
tags:
  - docker
  - rag
  - llama
abbrlink: 4f9e6d87
date: 2025-02-11 22:00:00
---

## 模型地址

<https://huggingface.co/bbvch-ai/bge-m3-GGUF/tree/main>

下载 `bge-m3-q4_k_m.gguf` 放到 `models` 文件夹

## Compose

文档: <https://github.com/ggerganov/llama.cpp/blob/master/examples/server/README.md>

- NVIDIA Runtime

```yaml
services:
  embedder-server:
    runtime: nvidia
    image: ghcr.io/ggerganov/llama.cpp:server-cuda
    container_name: librechat-embedder
    restart: unless-stopped
    networks:
      - traefik
    command: ["--keep", "-1"]
    environment:
      TZ: "Asia/Shanghai"
      LLAMA_ARG_HOST: "0.0.0.0"
      LLAMA_ARG_PORT: "8080"
      LLAMA_ARG_NO_WEBUI: true
      LLAMA_ARG_MODEL: "/models/bge-m3-q4_k_m.gguf"
      LLAMA_ARG_CTX_SIZE: 0
      LLAMA_ARG_N_PARALLEL: 4
      LLAMA_ARG_N_GPU_LAYERS: 99
      LLAMA_ARG_ALIAS: "bge-m3-q4"
      LLAMA_ARG_EMBEDDINGS: true
      LLAMA_API_KEY: "sk-llama"
    volumes:
      - ./models:/models
```

- CPU Runtime

```yaml
services:
  embedder-server:
    image: ghcr.io/ggerganov/llama.cpp:server
    container_name: librechat-embedder
    restart: unless-stopped
    networks:
      - traefik
    command: ["--keep", "-1"]
    environment:
      TZ: "Asia/Shanghai"
      LLAMA_ARG_HOST: "0.0.0.0"
      LLAMA_ARG_PORT: "8080"
      LLAMA_ARG_NO_WEBUI: true
      LLAMA_ARG_MODEL: "/models/bge-m3-q4_k_m.gguf"
      LLAMA_ARG_CTX_SIZE: 0
      LLAMA_ARG_N_PARALLEL: 4
      LLAMA_ARG_N_GPU_LAYERS: 99
      LLAMA_ARG_ALIAS: "bge-m3-q4"
      LLAMA_ARG_EMBEDDINGS: true
      LLAMA_API_KEY: "sk-llama"
    volumes:
      - ./models:/models
```
