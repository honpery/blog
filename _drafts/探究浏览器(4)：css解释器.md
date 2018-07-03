---
title: 探究浏览器(4)：css解释器
categories: 前端
tags: 浏览器
date: 2018-07-01 19:52:57
---

<p></p>
<!-- mroe -->

## 前言

## 包含块模型

具体定义：

- 根元素的包含块称为初始包含块，通常为可视区域大小。
- 位置为static或relative的元素，包含块是最近祖先内容区域。
- 位置为fixed的元素，包含块脱离HTML文档，固定在可视区域的某个特定位置。
- 位置为absolute的元素，包含块由最近的absolute、fixed或relative祖先决定。
    - 具有inline属性：包含改祖先的第一个和最后一个inline框的内边距的区域。
    - 不具有inline属性：该祖先的内边距所包围的区域。

## CSSOM

## 布局计算

## 优化

## 小结