---
title: 分解koa(1)：中间件
categories: 后端
tags: koa
date: 2018-07-20 17:17:08
---

<p></p>
<!-- more -->

## 前言

中间件是koa的核心特性。

简单的说，中间件就是函数，请求进来后，依次执行一系列中间件。

本文深入源码，探究koa的中间件的实现。

## koa中的中间件使用

调用use方法注册中间件，中间件会保存在队列中。

中间件即为一函数，包含上下文、next方法参数。

中间件符合洋葱模型，调用next方法暂停当前中间件执行，调用下一个中间件，按此依次执行，直到一个返回，会反向恢复中间件原有行为。

洋葱模型很像函数调用栈。

```js
const app = require('koa')();

app.use((ctx, next) => {
    console.log('1 start');
    next();
    console.log('1 end');
});

app.use((ctx, next) => {
    console.log('2 start');
    next();
    console.log('2 end');
});

app.listen(8080);

// 1 start -> 2 start -> 2 end -> 1 start
```

当然，这里的next函数返回的是Promise对象，可以使用`async`的方式执行。

```js
app.use(async (ctx, next) => {
    console.log('begin');
    await next();
    console.log('end');
});
```

## 剖析中间件原理

### 样例

反向思考，将原有中间件剥离出来，作为测试用例。p1/p2为中间件，cb为又返回结果的中间件。

```js
function p1 () {
    console.log('p1 start.');
    console.log('p1 end.');
}

function p2 () {
    console.log('p2 start.');
    console.log('p2 end');
}

function cb () {
    console.log('cb');
}

// p1 start. -> p2 start. -> cb -> p2 end. -> p1 end.
```

### 调用栈

上文提到，洋葱模型很像调用栈，可以借此为出发点。

```js
function p1 () {
    console.log('p1 start.');
    p2();
    console.log('p1 end.');
}

function p2 () {
    console.log('p2 start.');
    cb();
    console.log('p2 end');
}

function cb () {
    console.log('cb');
}

p1();
```

可以看出，这种硬编码耦合性非常高，如果想换个顺序基本gg了。

### 高阶函数

思考上面的栗子，可否将调用的函数传进去，这就是高阶函数的思路。

```js
function p1 (next) {
    console.log('p1 start.');
    next();
    console.log('p1 end.');
}

function p2 (next) {
    console.log('p2 start.');
    next();
    console.log('p2 end');
}

function cb () {
    console.log('cb');
}

p1(p2);
```

可以看出，问题是解决了，但是调用方式不优雅，嵌套层次太多。

### 组合

前文提到，中间件有单独的队列，可以考虑将中间件以数组形式传参。

可以考虑封装`compose`函数，在其内部递归调用中间件队列。

```js
function p1(next) {
    console.log('p1 start.');
    next();
    console.log('p1 end.');
}

function p2(next) {
    console.log('p2 start.');
    next();
    console.log('p2 end');
}

function cb() {
    console.log('cb');
}

function compose(arr) {
    function dispatch(i) {
        const fn = arr[i];
        if (!fn) return;
        return fn(dispatch.bind(null, ++i));
    }

    return dispatch(0);
}

compose([p1, p2, cb]);
```

此时需求基本达到，按照官方的写法进一步修改。

### 上下文、回调

按照官方的写法，改写测试用例，支持上下文参数。

```js
// test usage
function p1(ctx, next) {
    console.log('p1 start.');
    next();
    console.log('p1 end.');
}

function p2(ctx, next) {
    console.log('p2 start.');
    next();
    console.log('p2 end');
}

function cb() {
    console.log('cb');
}
```

下面改写compose方法，支持上下文，需要在调用的传入上下文参数，所以考虑返回一个闭包。

闭包的参数也可以加一个最终的回调，在其中做返回处理。

```js
function compose(arr) {
    return function(ctx, next) {
        function dispatch(i) {
            let fn = arr[i];
            if (i === arr.length) fn = next;
            if (!fn) return;
            return fn(ctx, dispatch.bind(null, ++i));
        }

        return dispatch(0);
    }
}

const run = compose([p1, p2])(ctx, cb);
```

### 多次调用next

当多次调用next时，会出先跳跃的问题，需要单独管理索引。

```js
function compose(arr) {
    return function (ctx, next) {
        let index = -1;
        function dispatch(i) {
            if (i < index) throw new Error('mluti call next()');
            index = i;

            let fn = arr[i];
            if (i === arr.length) fn = next;
            if (!fn) return;

            return fn(ctx, dispatch.bind(null, i + 1));
        }

        return dispatch(0);
    }
}


```


上面的方法都是同步的情况，~~~异步场景靠回调~~~。

### 异步

考虑Promise、async、await处理异步的情况，需要对compose进行修改。

```js
function compose(arr) {
    return function (ctx, next) {
        let index = -1;
        function dispatch(i) {
            if (i < index) Promise.reject(new Error('mluti call next()'));
            index = i;

            let fn = arr[i];
            if (i === arr.length) fn = next;
            if (!fn) return Promise.resolve();

            try {
                return Promise.resolve(fn(ctx, dispatch.bind(null, i + 1)));
            }catch (err) {
                return Promise.reject(err);
            }

        }

        return dispatch(0);
    }
}
```

上面的`compose`即为koa中的依赖库`koa-compose`的核心思路。

### 参数检测

中间件队列需是数组，每个元素为函数。

```js
function compose(middleware) {
    if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
    for (const fn of middleware) {
        if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
    }

    return function(context, next) {
        let index = -1
        return dispatch(0)

        function dispatch(i) {
            if (i <= index) return Promise.reject(new Error('next() called multiple times'))
            index = i
            let fn = middleware[i]
            if (i === middleware.length) fn = next
            if (!fn) return Promise.resolve()
            try {
                return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
            } catch (err) {
                return Promise.reject(err)
            }
        }
    }
}
```

此处即为`koa-compose`的全部源码。

## 小结

本文分析了koa中的中间件原理，完全依赖于koa-compose。