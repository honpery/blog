---
title: css中的居中
tags: css
categories: 前端
date: 2018-09-03 13:23:21
---

<p></p>
<!-- more -->

## 前言

css中布局居中是常见的需求和面试题，虽然工作中常用的是flex，但是为了探究各种hack方法，特将发现总结成文。

#### flex布局

```css
.parent {
    display: flex;
    justify-content: center;
    align-items: center;
}
```

#### 行内元素

```css
.parent {
    height: 200px;
    line-height: 200px;         /* 行高与高度相等 */
    text-align: center;
}
.parent .child {
    display: inline-block;
    line-height: 1;             /* 修整行高 */
}
```

#### 块级元素

```css
.parent {

}

/* 只垂直居中 */
.parent:after {
    display: inline-block;
    content: '';
    height: 100%;
    vertical-align: middle;
}

.child {
    /* 只水平居中 */
    width: 100px;               /* 必须设置宽度 */
    margin: 0 auto;
    
    /* 只垂直居中 */
    vertical-align: middle;
    display: inline-block;
}
```

#### 绝对定位 + 负margin

这种方式无法自适应宽高。

```css
.parent {
    position: relative;
}
.parent .child {
    position: absolute;
    top: 50%;
    bottom: 50%;
    width: 100px;
    height: 100px;
    margin-left: -50px;         /* 宽度一半 */
    margin-top: -50px;          /* 高度一半 */
}
```

#### 绝对布局 + transform

```css
.parent {
    position: relative;
}

.parent .child {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);   /* 自适应宽高 */
}
```

#### 绝对布局 + auto margin

这种方式无法自适应宽高。

```css
.parent {
    position: relative;
}

.parent .child {
    position: absolute;
    width: 100px;
    height: 100px;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
}
```

#### 缩放

```css
.parent {

}
.parent .child {
    height: 100%;
    width: 100%;
    transform: scale(0.5, 0.5);
}
```
