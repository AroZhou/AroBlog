---
layout:     post
title:      CSS FlexBox布局
subtitle:   Flex布局的语法详解
date:       2019-08-11
author:     Aro
header-img: img/post-bg-cssflexbox.jpeg
catalog: true
tags:
    - CSS
    - 前端
---

>Flex布局在开发中用到甚多，以前每次用到都要去谷歌一遍，快速解决问题之后就不了了之。这样效率很低下，虽然谷歌看一遍就能解决眼前的问题，但是这次恰逢周末，想到那就不如把Flex布局整理一下记在心里。免得一次次的去查。


布局的传统解决方案，基于盒状模型，依赖 `display` 属性 + `position`属性 + `float`属性。它对于那些特殊布局非常不方便，比如，垂直居中就不容易实现。

2009年，W3C 提出了一种新的方案----Flex 布局，可以简便、完整、响应式地实现各种页面布局。目前，它已经得到了所有浏览器的支持，这意味着，现在就能很安全地使用这项功能。

## Flex 布局简介

Flex 是 Flexible Box 的缩写，意为"弹性布局"，用来为盒状模型提供最大的灵活性。

任何一个容器都可以指定为 Flex 布局。

```css
.box{
    display: flex;
}
```

行内元素也可以使用 Flex 布局。

```css
.box{
  display: inline-flex;
}
```

Webkit 内核的浏览器，必须加上`-webkit`前缀。

```css
.box{
  display: -webkit-flex; /* Safari */
  display: flex;
}
```
>注意，设为 Flex 布局以后，子元素的`float`、`clear`和`vertical-align`属性将失效。


## Flex 结构

采用 Flex 布局的元素，称为 Flex 容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为 Flex 项目（flex item），简称"项目"。

![flex](/img/20190711flex.png)

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做`main start`，结束位置叫做`main end`；交叉轴的开始位置叫做`cross start`，结束位置叫做`cross end`。

项目默认沿主轴排列。单个项目占据的主轴空间叫做`main size`，占据的交叉轴空间叫做`cross size`。


## Flex 容器的属性

以下六个属性设置在容器上。
- flex-direction
- flex-wrap
- flex-flow
- justify-content
- align-items
- align-content


