---
title: 优雅处理npm私有源并用问题
tags: npm
categories: 前端
date: 2018-01-06
---

本文阐述了如何优雅处理npm私有源和公有源并用的问题。

<!-- more -->

## 前言

因涉及到业务相关的模块需要发布在内部npm上，其他依赖还是从npm上拉取，如何区分多个源的依赖是一个问题，下文找到了自认为优雅的解决方案。

核心原理是根据[scope](https://docs.npmjs.com/misc/scope)区分npm源，下文举例scope=sc。

下文假设Project MAIN依赖Project DEP。


## 发布模块

因为需要根据不同的scope判断npm源，需要为DEP的`package.json`添加以下字段。

```json
{
    "name": "@sc/demo",
    "publishConfig": {
        "registry": "http://npm.sc.com"
    }
}
```

因为`yarn login`不支持`--registry`参数，所以发布的时候需要直接使用`npm publish`。

```bash
npm login --registry http://npm.sc.com
npm publish
```

## 安装模块

在MAIN**项目根目录**创建`.yarnrc`和`.npmrc`配置文件，内容均为以下配置。

```
"@sc:registry" "http://npm.sc.com"
```

配置后安装依赖方式照旧，注意包名中要有scope。

## 开发模块

如开发MAIN的时候需要修改调试DEP，可以使用[npm link](https://docs.npmjs.com/cli/link)。

```bash
# dep project root
sudo npm link

# main project root
# npm link [dep package name]
npm link @sc/demo

# 取消link
npm unlink && yarn
```