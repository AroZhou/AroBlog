---
layout:     post
title:      React Native中的 props 和 state
subtitle:   来一个关于 state 和 props 的总结
date:       2019-08-22
author:     Aro
header-img: img/post-bg-reactnative.png
catalog: true
tags:
    - React Native
    - 前端
---

>在任何应用中，数据都是必不可少的。我们需要直接的改变页面上一块的区域来使得视图的刷新，或者间接地改变其他地方的数据。React的数据是自顶向下单向流动的，即从父组件到子组件中，组件的数据存储在 `props` 和 `state` 中，这两个属性有啥子区别呢？


## Props（属性）

React的核心思想就是组件化思想，页面会被切分成一些独立的、可复用的组件。

大多数组件在创建时就可以使用各种参数来进行定制。用于定制的这些参数就称为 `props`（属性）。组件从概念上看就是一个函数，可以接受一个参数作为输入值，这个参数就是 `props`，所以可以把 `props` 理解为从外部传入组件内部的数据。由于React是单向数据流，所以 `props` 基本上也就是从服父级组件向子组件传递的数据。


#### 以基础组件为例

>以常见的基础组件 `Image` 为例，在创建一个图片时，可以传入一个名为 `source` 的 `prop` 来指定要显示的图片的地址，以及使用名为`style` 的 `prop` 来控制其尺寸。

```js
import React, { Component } from 'react';
import { Image } from 'react-native';

export default class Bananas extends Component {
  render() {
    let pic = {
      uri: 'xxx.png'
    };
    return (
      <Image source={pic} style={...} />
    );
  }
}
```


#### 以自定义组件为例

>自定义的组件也可以使用 `props`。通过在不同的场景使用不同的属性定制，可以尽量提高自定义组件的复用范畴。只需在 `render` 函数中引用 `this.props`，然后按需处理即可。下面是一个例子：

```js
import React, { Component } from 'react';
import { Text, View } from 'react-native';

class Greeting extends Component {
  render() {
    return (
      <View style={...}>
        <Text>Hello {this.props.name}!</Text>
      </View>
    );
  }
}

export default class LotsOfGreetings extends Component {
  render() {
    return (
      <View style={...}>
        <Greeting name='Rexxar' />
        <Greeting name='Jaina' />
        <Greeting name='Valeera' />
      </View>
    );
  }
}
```


## State（状态）

>我们使用两种数据来控制一个组件：`props` 和 `state`。 `props`是在父组件中指定，而且一经指定，在被指定的组件的生命周期中则不再改变。 对于需要改变的数据，我们需要使用 `state`。

一般来说，你需要在 `constructor` 中初始化 `state`，然后在需要修改时调用 `setState` 方法。


#### 举个例子

假如我们需要制作一段不停闪烁的文字。文字内容本身在组件创建时就已经指定好了，所以文字内容应该是一个prop。而文字的显示或隐藏的状态（快速的显隐切换就产生了闪烁的效果）则是随着时间变化的，因此这一状态应该写到state中。

```js
import React, { Component } from 'react';
import { Text, View } from 'react-native';

class Blink extends Component {
  state = { isShowingText: true };
  
  componentDidMount() {
    // 每1000毫秒对showText状态做一次取反操作
    setInterval(() => {
      this.setState(previousState => {
        return { isShowingText: !previousState.isShowingText };
      });
    }, 1000);
  }

  render() {
    // 根据当前showText的值决定是否显示text内容
    if (!this.state.isShowingText) {
      return null;
    }

    return (
      <Text>{this.props.text}</Text>
    );
  }
}

export default class BlinkApp extends Component {
  render() {
    return (
      <View>
        <Blink text='I love to blink' />
        <Blink text='Yes blinking is so great' />
        <Blink text='Why did they ever take this out of HTML' />
        <Blink text='Look at me look at me look at me' />
      </View>
    );
  }
}
```

**提示一些初学者应该牢记的要点:**

- 一切界面变化都是状态 `state 变化
- `state` 的修改必须通过 `setState()` 方法
    - `this.state.likes = 100` 这样的直接赋值修改无效！
    - `setState` 是一个 `merge` 合并操作，只修改指定属性，不影响其他属性
    - `setState` 是异步操作，修改不会马上生效


## 参考链接

- <a href="https://reactnative.cn/docs/props/" target="_blank">https://reactnative.cn/docs/props/</a>
- <a href="https://reactnative.cn/docs/state/" target="_blank">https://reactnative.cn/docs/state/</a>