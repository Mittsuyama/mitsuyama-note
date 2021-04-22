# React 官方文档

官方文档链接：[Hello World - React](https://www.reactjscn.com/docs/hello-world.html)

## 基础部分

### 元素渲染

React 为了防止 XSS 攻击，会转义内容。想要渲染 HTML 代码，需要用使用如下方式：

```jsx
<div dangerouslySetInnerHTML={ __html: HTML_CODE } />
```

JSX 代表 Object，Babel 转换器会调用 `React.createElement()` 来转换成一个类似如下的对象（真实情况更复杂）：

```jsx
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world'
  }
};
```

即使每次更新都调用一遍 `Reder()` ，渲染的时候也只会渲染改变了的部分。希望开发者将页面视为一个个特定的固定的内容，就像**帧动画的每一帧**，而不是一直处于变化之中。

组件名必须是大写，使用 `<Welcome />` 表示一个组件，而 `<welcome />` 会被认为是一个原生 `HTML` 标签

### State & 生命周期

关于 `setState` 的同步异步问题：`this.props` 和 `this.state` **可能**是异步更新的，开发者不应该依靠它们的值来计算下一个状态。而应该用函数的形式：

```jsx
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));

// 而不是 { counter: this.state.counter + 1 }
```

什么时候是异步的？

1. React 的时间函数
2. React 的生命周期函数

同步：其他的异步函数，比如：定时器，原生事件监听，promise 等。

### 关于合并

React 会合并相邻的 `setState({})` ，来提高性能，即：

```jsx
// 初始 counter 为 0

// 此时的 this.state.counter = 0 
this.setState({ counter: this.state.counter + 1 }); 

// 此时的 this.state.counter 同样为 0 
this.setState({ counter: this.state.counter + 1 });
```

但是 React 不会合并 `setState()` 接受函数的情况，接受函数时，React 会保证每次传入的值都是最新的，但是页面刷新会合并（即更新多次数据，更新一次页面）

### 事件处理

React 事件绑定属性的命名采用**驼峰式写法**，而不是小写，比如 `onClick` 。React 中不能使用 `return false;` 来阻止默认行为，必需**显示地**使用`event.preventDefault()` 。

查看同步事件参考指南：[SyntheticEvent - React](https://www.reactjscn.com/docs/events.html)

React 拥有和浏览器原事件一样的接口，包括 `stopProgagation()` 和 `preventDefault()` 。

**事件池**：`SyntheticEvent` 是共享的。那就意味着在调用事件回调之后，`SyntheticEvent` 对象将会被重用，并且所有属性会被置空。这是出于性能因素考虑的。 因此，无法以异步方式访问事件。即：

```jsx
function onClick(event) {
  console.log(event); // => nullified object.
  console.log(event.type); // => "click"
  const eventType = event.type; // => "click"

  setTimeout(function() {
    console.log(event.type); // => null
    console.log(eventType); // => "click"
  }, 0);
}
```

### 状态提升

使用 react 经常会遇到几个组件需要共用状态数据的情况。这种情况下，我们最好将这部分共享的状态**提升至他们最近的父组件**当中进行管理。