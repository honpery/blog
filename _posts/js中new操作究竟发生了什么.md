---
title: js中new操作究竟发生了什么
tags: js
categories: 前端
date: 2018-06-30 12:00:00
---

<p></p>
<!-- more -->

## 前言

本文通过以下实例探究js中的new过程究竟发生了什么。

```js
function Person (name) {
    this.name = name;
}

Person.prototype.getName = function () {
    return this.name;
}

const p = new Person('demo');
```


## 构造函数、原型对象、实例的关系

首先通过控制台，看看实例对象当前的结构。

```js
{
    name: 'demo',
    __proto__: {
        constructor: Person,
        getName: Function,
        __proto__: Object
    }
}
```

不难看出三者之间的关系如下图。

![](/images/prototype-relation.png)

构造函数 -> 原型对象：构造函数的prototype属性，指向原型对象。
原型对象 -> 构造函数：原型对象的constructor属性，指向其构造函数。
实例 -> 原型对象：实例的`__proto__`属性，指向其原型对象。

## this指针

this指针是动态的，需要运行时才能知道（箭头函数内部的this是静态的），其指向的4种情况非常重要：

- 函数调用：指向全局对象。
- 方法调用：指向该对象。
- 构造函数：指向对象实例。
- call、apply：指向第一个参数传入的对象。

可以看到，new的过程需要将this指针指向新的对象实例上。

## 代码展示new的过程

因为new是操作符，为方便演示原理，下面创建了一个`newObj`函数。

```js
function newObj (constructor, args) {
    const obj = {};                             // 创建一个空对象
    obj.__proto__ = constructor.prototype;      // 修改__proto__指向原型对象
    constructor.apply(obj, args);               // 修改this指针
    return obj;                                 // 返回新对象
}
```

## 小结

说到底，new包含四个过程，创建新对象，设置`__proto__`属性，绑定this，返回该对象。
