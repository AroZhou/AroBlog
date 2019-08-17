---
layout:     post
title:      Redux 源码解析(二)  compose.js
subtitle:   逐行解析Redux的API
date:       2019-08-14
author:     Aro
header-img: img/post-bg-redux.png
catalog: true
tags:
    - Redux
    - 前端
---

>目前所分析的`Redux`版本为`4.0.4`。`compose`这个模块的代码十分简练，但是实现的作用却是十分强大。`reduce`正是这个模块的核心。


## `reduce`是什么？

`reduce()` 方法对数组中的每个元素执行一个由您提供的`reducer`函数(升序执行)，将其结果汇总为单个返回值。注意，这个升序指的是按照字典序。

具体的解析在我之前一篇文章中提到过。


## `compose`的效果

明白了reduce是怎么回事之后，我们先来看一下compose有什么神奇的效果：

```js
import compose from '...';

// function f
const f = (arg) => `函数f(${arg})`
// function g
const g = (arg) => `函数g(${arg})`
// function h 最后一个函数可以接受多个参数
const h = (...arg) => `函数h(${arg.join('_')})`

const result = compose(f, g, h)(1,2,3)

console.log(typeof result) // function

console.log(result) //函数f(函数g(函数h(1_2_3)))
```


## compose.js 源码

```js
/**
 * Composes single-argument functions from right to left. The rightmost
 * function can take multiple arguments as it provides the signature for
 * the resulting composite function.
 *
 * @param {...Function} funcs The functions to compose.
 * @returns {Function} A function obtained by composing the argument functions
 * from right to left. For example, compose(f, g, h) is identical to doing
 * (...args) => f(g(h(...args))).
 */

export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

## 参考链接

<a href="https://github.com/reduxjs/redux" target="_blank">https://github.com/reduxjs/redux</a>