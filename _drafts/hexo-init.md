---
title: 搭建自认为完美的hexo博客
date: 2018-01-28 23:07:48
tags: hexo
categories: hexo
---

本文记录了搭建hexo博客的全过程，

<!-- more -->

## 前言

之前一直想自己写一套blog系统作为新技术练手，后来学啥都想塞进去，所以迟迟未能完工，最后痛定思痛，重新思考写博客真正的意义不在于网站多牛逼，应该内容才是核心，所以直接放弃自己写的想法，换成了hexo，本文记录了整个搭建过程。

## 项目初始化

考虑到把hexo构建、主题和文章分离开，所以使用[git submodule](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)进行仓库的分离。

1. 在github上创建两个仓库，创建的时候记得勾选创建`README.md`文件，避免添加submodule报错。

- `blog-builder`: 用于静态文件的构建。
- `blog`: 用户保存文章等数据。

2. 安装[hexo](https://hexo.io/)，初始化项目：

```bash
npm install hexo-cli -g
hexo init blog
cd blog && npm install
```

3. 初始化git、初始化文章submodule：

```bash
# 删除source、themes目录
rm -rf ./source ./themes

# 初始化git目录
git init
git remote add origin git@github.com:honpery/hexo-demo-builder.git

# 初始化文章资源的submodule
git submodule add git@github.com:honpery/hexo-demo-blog.git source
```

4. 主题相关

考虑到主题官方的升级，所以主题也是用submodule，采用[hexo 数据文件]特性将主题配置分离到`source/_data/[theme_name].yml`下。我的博客采用的是[next](http://theme-next.iissnan.com/)主题，详细主题配置可以参考文档。

```bash
# 添加主题
git submodule add https://github.com/iissnan/hexo-theme-next themes/next

# 复制主题配置文件
mkdir source/_data && cp themes/next/_config.yml source/_data/next.yml
```

之后切换主题、配置主题参考主题的官方文档，主题配置文件直接配置`source/_data/[theme_name].yml`即可。

## 配置ci

1. 配置travis ci

使用ci可以每次push提交后自动构建部署，在项目根目录创建`.travis.yml`文件，配置如下，过程中遇到一个坑需要注意，travis的submodule默认是不拉取远程的最新提交，所以需要单独配置。

```yml
language: node_js

node_js:
    - "9"

branches:
    only:
        - master

git:
    submodules: false
  
before_install:
    - git submodule update --init --remote --recursive

script:
    - yarn test & yarn run build && yarn run deploy

notifications:
    email:
        on_success: never
        on_failure: always
```
