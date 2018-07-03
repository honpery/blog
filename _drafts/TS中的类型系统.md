---
title: TS中的类型系统
categories: 前端
tags: ts
date: 2018-06-30 20:07:43
---

本文整理了ts中类型系统的用法

<!-- more -->

## 前言

TypeScript核心功能基本就是两个，类型系统和ES6+编译器。

ts最近基本版本，对类型系统有了更多的支持。借此机会，整理ts中的类型系统用法。

## 基础类型

js中的基本类型为：Number、String、Boolean、null、undefined、Symbol，ts中均有对应。

### `number`

数字，与包装类型`Number`等价，两者推导类型均为`number`，不推荐使用包装类型。

```ts
let num1: number = Number(1);       // 不报错
let num2: Number = 1;               // 不报错
let num3 = Number(1);               // number
```

### `string`

字符串，与包装类型`String`等价，两者推导类型均为`string`，不推荐使用包装类型。

```ts
let str1: string = String('str1');  // 不报错
let str2: String = 'str2';          // 不报错
let str3 = String('11');            // string
```

### `boolean`

布尔值。与包装类型`Boolean`等价，两者推导类型均为`boolean`，不推荐使用包装类型。

```ts
let b1: boolean = Boolean(true);    // 不报错
let b2: Boolean = true;             // 不报错
let b3 = Boolean(true);             // boolean
```

### `symbol`

### `undefined`

### `null`

### `void`

### `any`

### `never`

### `Array<T>`与`T[]`

### `enum`

### `object`

## 接口

## 类

## 函数

## 小结

## 参考资料

- [TypeScript官方文档](http://www.typescriptlang.org/)