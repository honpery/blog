---
title: js中的正则
categories: 前端
tags: js
date: 2018-07-08 20:36:02
---

<p></p>

<!-- more -->

## 前言

正则是字符串处理中最为简练与便捷的方式。本文全面总结了其在js中的应用。

正则由两部分组成，模式(pattern)和修饰符(flags)。模式是要命中的规则，模式包含一套特殊字符，用于指代特定规则，修饰符表示模式的附加规则。

## 创建

js中的正则创建有两种创建方式，`正则表达式字面量`和`RegExp构造函数`。

#### 正则表达式字面量

正则表达式字面量的形式为，`/pattern/flags`，两个单斜杠，中间为模式，最后为可选的修饰符。

```js
const r1 = /abc/;
const r2 = /abc/ig;
```

#### RegExp构造函数

RegExp构造函数接受两个参数，第一个为模式，可以是字符串或正则，第二个为可选的修饰符。

当第一个参数为正则，第二个参数存在时，存在兼容性问题。es5中，会报`TypeError`，es6中。修饰符会被第二个参数覆盖掉。

```js
new RegExp('abc');               // /abc/
new RegExp(/aaa/);               // /aaa/
new RegExp('abc', 'ig');         // /abc/ig

// es5
new RegExp(/abc/g, 'i');         // TypeError
// es6
new RegExp(/abc/g, 'i');         // /abc/i
```

#### 对比

- 在正则表达式不变的情况下，前者性能更好。后者多用于运行时动态创建正则表达式。
- 两种方式创建的返回值均为字面量，`typeof`的值均为`object`。

## 特殊字符

特殊字符可以分为：字符类别、字符集合、便捷、分组与分享引用、数量词和断言几大类。

#### 字符类别

`.`匹配单个任意字符。
`\d`匹配单个数字，等价于`[0-9]`。
`\D`匹配单个非数字，等价于`[^0-9]`。
`\w`匹配基本拉丁字母表中的字母数字、字符、下划线，等价于`[0-9a-zA-Z]`。
`\W`匹配不在基本拉丁字母表中的字符，等价于`[^0-9a-zA-Z]`。
`\s`匹配一个空白符，包括空格、制表符、换页符、换行符和其他 Unicode 空格。
`\S`匹配非空白符。

```js

```

#### 字符集合

`[abc]`匹配集合中的任意字符，可以使用连字符`-`指定范围。
`[^abc]`匹配不在集合中的任意字符，同样可指定范围。

```js
/[abc]/.test('b');          // true
/[^abc]/.test('d');         // true
```

#### 边界

`^`匹配输入开始。如果存在多行修饰符，也匹配换行符后的开始。
`$`匹配输入结尾。如果存在多行修饰符，也匹配换行符前的结尾。
`\b`
`\B`

```js
/^abc/.test('abcd');        // true
/abc$/.test('ababc');       // true
```

#### 分组和反向引用

#### 数量词

## 修饰符

### g 全局搜索

### i 不区分大小写

### m 多行搜索

### y 粘性搜索

### u unicode

### s

## js中正则的相关方法

### RegExp.prototype.test()

`test`方法用于判断是否匹配。

### RegExp.prototype.exec()

`exec`方法用于执行匹配。如果匹配到则返回匹配子串数组，否则返回`null`。

如果正则包含`组匹配`，结果数组第一个元素为整个匹配的结果，后面的为每个组匹配到的结果。

```js
/abc/.exec('abcd');         // ['abc']
```

### String.prototype.match()

### String.prototype.replace()

### String.prototype.search()

### String.prototype.split()


## 小结

## 参考资料

- [MDN - 正则表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions)
- [阮一峰javascript教程 - 正则表达式](http://javascript.ruanyifeng.com/stdlib/regexp.html)
- (阮一峰es6入门 - 正则的扩展)(http://es6.ruanyifeng.com/#docs/regex)