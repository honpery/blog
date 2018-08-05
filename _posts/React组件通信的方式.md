---
title: React组件通信的方式
categories: 前端
tags: React
date: 2018-06-30 17:07:12
---

<p></p>
<!-- more -->

## 概述

React中组件会形成树的结构，所以如何通信就成了必知问题。

## 父 -> 子：属性传递

这是官方提供的核心功能，父组件通过属性传递数据，子组件通过`props`属性接受。

```jsx
function Parent () {
    return <Child name="demo" />;
}

function Child (props) {
    return <p>{props.name}</p>;
}
```

```jsx
function Parent () {
    return <Middle name="demo" />;
}

function Middle (props) {
    return <Child {...props} />;
}

function Child (props) {
    return <p>{props.name}</p>;
}
```

## 子 -> 父：回调

这种情况也是通过props，只不过传递的是回调函数，当子组件需要传数据给父组件时，调用该回调。

这里的常见问题是，类组件的this指针问题，需要通过`bind`方法或者箭头函数的特性进行绑定。

（无状态组件无this指针，createElement创建的组件有自绑定）

```jsx
class Parent extends React.Component {
    name = 'parent';

    getName (name) {
        console.log(`${this.name} component`);
    }

    render () {
        <Child onClick={this.getName.bind(this)} />;
    }
}

function Child (props) {
    return <button onClick={props.onClick}>click me</button>;
}
```

## 跨级组件：Context

可以使用React 16.3提供的新版Context API来传递数据。

```jsx
const {Provider, Consumer} = createContext({
    name: 'default',
});

function A () {
    return (
        <Provider value={{name: 'demo'}}>
            <B />
        </Provider>
    )
}

function B () {
    return (
        <Consumer>
            {ctx => <p>{ctx.name}</p>}
        </Consumer>
    )
}
```

## 兄弟组件：提升父组件

可以通过同一父组件进行传递。

```jsx
class Parent extends React.Component {
    constructor (props) {
        super(props);
        this.state = { name: '' };
    }
    render () {
        return (
            <div>
                <Child1 onClick={name => this.setState({name})} />
                <Child2 name={this.state.name} />
            </div>
        );
    }

}

function Child1 (props) {
    return <button onClick={() => props.onClick('child1')}>click me</button>;
}

function Child2 (props) {
    return <p>{props.name}</p>
}
```

这种方式有几个缺点：

- 父组件状态变动，会导致所有子组件走对应的生命周期。
- 若只为通信而创建父组件，此组件有些多余，到了层次复杂。
- 层次较深时层层传递很蛋疼。

也可以通过Context API实现，方法同跨级组件。

## 万能大法：观察者模式

可以看到上面的方法，是把通信数据保存在了父组件的状态中。

换个思路，参考事件机制，通过某个对象进行中转，发布者变更数据后，该对象内部调用订阅者的回调。

以下为事件代理对象，其中on方法用于订阅，off用于取消订阅，emit用于触发事件。

```js
const EventProxy = {
    _queues: {},
    on(name, cb) {
        const queue = this._queues[name] = this._queues[name] || [];
        queue.push(cb);
    },

    off(name, cb) {
        const queue = this._queues[name];
        if (!queue || !queue.length) return;
        queue.forEach(_cb => _cb !== cb);
    },

    emit(name, data) {
        const queue = this._queues[name];
        if (!queue || !queue.length) return;
        queue.forEach(cb => cb(data));
    }
};

function Child1() {
    return <button onClick={() => EventProxy.emit('click', 'demo')}>click</button>
}

class Child2 extends React.Component {
    constructor (props) {
        super(props);
        this.state = {name: ''};
    }

    _onClick (name) {
        this.setState({name});
    }

    componentDidMount () {
        EventProxy.on('click', this._onClick.bind(this));
    }

    componentWillUnmount () {
        EventProxy.off('click', this._onClick.bind(this));
    }

    render () {
        return <p>{this.state.name}</p>;
    }
}
```

这种方法看上去很刺激，但是隐患无穷。

React提倡的单向数据流，更多的是为了调试与维护的时候，更好最终数据的状态与流向。

观察者模式会造成，发布者的位置不易追踪，造成bug。

## 小结

可以看到，更多的还是利用props的单向传递进行通信，观察者模式尽量少用，这是反范式的。

## 参考资料

- [React 组件间通讯](http://taobaofed.org/blog/2016/11/17/react-components-communication/)