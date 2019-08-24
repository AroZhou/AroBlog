---
layout:     post
title:      ReactNative Animated 动画解析
subtitle:   由一个 ReactNative 项目引发的思考
date:       2019-08-20
author:     Aro
header-img: img/post-bg-reactnative-animated.png
catalog: true
tags:
    - React Native
    - 前端动画
    - 前端
---

流畅、有意义的动画对于移动应用用户体验来说是非常重要的。现实生活中的物体在开始移动和停下来的时候都具有一定的惯性，我们在界面中也可以使用动画来实现契合物理规律的交互。

在 [我的项目](https://gitee.com/AroZhou/px_rn) 中主要是用Animated来实现一个呼吸效果的 `占位图(placeholder)` 。`Animated` 库旨在使动画变得流畅，强大并易于构建和维护。`Animated` 侧重于输入和输出之间的声明性关系，以及两者之间的可配置变换，此外还提供了简单的 `start` / `stop` 方法来控制基于时间的动画执行。

创建动画最简单的工作流程是创建一个 Animated.Value ，将它连接到动画组件的一个或多个样式属性，然后使用Animated.timing()通过动画效果展示数据的变化：

```js
Animated.timing(
  // timing方法使动画值随时间变化
  this.state.fadeAnim, // 要变化的动画值
  {
    toValue: 1, // 最终的动画值
  },
).start(); // 开始执行动画
```


## 概览

`Animated` 提供了两种类型的值：
- `Animated.Value()` 用于单个值
- `Animated.ValueXY()` 用于矢量值

#### 配置动画

`Animated` 提供了三种动画类型。每种动画类型都提供了特定的函数曲线，用于控制动画值从初始值变化到最终值的变化过程：
- `Animated.decay()` 以指定的初始速度开始变化，然后变化速度越来越慢直至停下。
- `Animated.spring()` 提供了一个简单的弹簧物理模型。
- `Animated.timing()` 使用 `easing` 函数让数值随时间动起来。

大多数情况下你应该使用 `timing()`。默认情况下，它使用对称的 `easeInOut` 曲线，将对象逐渐加速到全速，然后通过逐渐减速停止结束。

#### 使用动画

通过在动画上调用 `start()` 来启动动画。 `start()` 需要一个 完成 回调函数，当动画完成时将会调用它。如果动画运行正常，则将通过 `{finished：true}` 触发回调。如果动画是因为调用了 `stop()` 而结束（例如，因为它被手势或其他动画中断），则它会收到 `{finished：false}`。

#### 启用原生动画驱动

使用原生动画，我们会在开始动画之前将所有关于动画的内容发送到原生代码，从而使用原生代码在 UI 线程上执行动画，而不是通过对每一帧的桥接去执行动画。一旦动画开始，JS 线程就可以在不影响动画效果的情况下阻塞（去执行其他任务）掉了。

您可以通过在动画配置中指定 `useNativeDriver：true` 来使用原生动画驱动。你可以在动画文档 中看到更详细的解释。

#### 自定义动画组件

组件必须经过特殊处理才能用于动画。所谓的特殊处理主要是指把动画值绑定到属性上，并且在一帧帧执行动画时避免 react 重新渲染和重新调和的开销。此外还得在组件卸载时做一些清理工作，使得这些组件在使用时是安全的。

- `createAnimatedComponent()` 方法正是用来处理组件，使其可以用于动画。

`Animated` 中默认导出了以下这些可以直接使用的动画组件，当然它们都是通过使用上面这个方法进行了封装：

- `Animated.Image`
- `Animated.ScrollView`
- `Animated.Text`
- `Animated.View`

#### 组合动画

动画还可以使用组合函数以复杂的方式进行组合：

- `Animated.delay()` 在给定延迟后开始动画。
- `Animated.parallel()` 同时启动多个动画。
- `Animated.sequence()` 按顺序启动动画，等待每一个动画完成后再开始下一个动画。
- `Animated.stagger()` 按照给定的延时间隔，顺序并行的启动动画。

默认情况下，如果一个动画停止或中断，则组合中的所有其他动画也会停止。

#### 合成动画值

你可以使用加减乘除以及取余等运算来把两个动画值合成为一个新的动画值：

- `Animated.add()`
- `Animated.subtract()`
- `Animated.divide()`
- `Animated.modulo()`
- `Animated.multiply()`

#### 插值

`interpolate()` 函数允许输入范围映射到不同的输出范围。默认情况下，它将推断超出给定范围的曲线，但也可以限制输出值。它默认使用线性插值，但也支持缓动功能。

- interpolate()


## 实现 `AroPlaceholder` 组件

`AroPlaceholder.js`

```js
{% raw %}
import React, {Component} from 'react';
import {Animated, Easing} from "react-native";

export default class FadeInView extends Component {

  constructor(props) {
    super(props);
    this.breathValue = new Animated.Value(0)
  }

  componentDidMount() {
    this.breath();
  }

  render() {

    const breath = this.breathValue.interpolate({
      inputRange: [0, 1, 2],
      outputRange: [.6, 1, .6]
    });

    return (
      <Animated.View                 // 使用专门的可动画化的View组件
        style={{
          ...this.props.style,
          opacity: breath,
        }}
      >
        {this.props.children}
      </Animated.View>
    );

  }

  breath() {
    this.breathValue.setValue(0);
    Animated.timing(                  // 随时间变化而执行动画
      this.breathValue,            // 动画中的变量值
      {
        toValue: 2,
        duration: 2000,              // 让动画持续一段时间
        easing: Easing.linear,
        useNativeDriver: true
      }
    ).start((isover) => {
      if(isover.finished){
        this.breath()
      }
    });                        // 开始执行动画
  }

}
{% endraw %}
```

`index.js`

```js
import React, {Component} from "react";
import {StyleSheet} from 'react-native';

import FadeInView from "./AroPlaceholder"

export default class AroPlaceholder extends Component {

  render(){
    return (
      <FadeInView style={styles.container} />
    )
  }

}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#9A9A9A',
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius: 4,
    marginTop: 4,
    marginBottom: 4,
    marginLeft: 15,
    marginRight: 15
  }
});
```


## 参考链接

- <a href="https://reactnative.cn/docs/animated/" target="_blank">https://reactnative.cn/docs/animated/</a>