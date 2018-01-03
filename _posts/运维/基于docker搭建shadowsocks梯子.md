---
title: 基于docker搭建shadowsocks梯子
tags:
---

本文详细记录了借助docker搭建shadowsocks翻墙服务器的全过程，仅供学习交流使用。

<!-- more -->

## 代理服务器操作

### 安装docker

使用官方的安装脚本即可自动安装docker。
```bash
wget -qO- https://get.docker.com/ | sh

# 墙墙专供
curl -sSL https://get.daocloud.io/docker | sh
```

### 拉取shadowsocks镜像

我使用的是[mritd/shadowsocks](https://hub.docker.com/r/mritd/shadowsocks)，非常稳定，推荐。如果被墙可以参考[这里](https://www.docker-cn.com/registry-mirror)的官方国内源。
```bash
docker pull mritd/shadowsocks

# 墙墙专供
docker pull registry.docker-cn.com/mritd/shadowsocks
```

### 