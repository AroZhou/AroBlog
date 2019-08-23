---
layout:     post
title:      ReactNative 动画详解
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

React Native 提供了两个互补的动画系统：用于创建精细的交互控制的动画Animated和用于全局的布局动画LayoutAnimation。

## Animated

> `Animated` 使得开发者可以非常容易地实现各种各样的动画和交互方式，并且具备极高的性能。 `Animated` 旨在以声明的形式来定义动画的输入与输出，在其中建立一个可配置的变化函数，然后使用简单的 `start` / `stop` 方法来控制动画按顺序执行。  `Animated` 仅封装了6个可以动画化的组件：`View`、 `Text`、 `Image`、 `ScrollView`、 `FlatList`和`SectionList` ，不过你也可以使用 `Animated.createAnimatedComponent()` 来封装你自己的组件。

