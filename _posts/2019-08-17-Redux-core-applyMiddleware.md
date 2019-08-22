---
layout:     post
title:      Redux 源码解析(五)  applyMiddleware.js
subtitle:   逐行解析Redux的API
date:       2019-08-17
author:     Aro
header-img: img/post-bg-redux.png
catalog: true
tags:
    - Redux
    - 前端
---

>目前所分析的 `Redux` 版本为 `4.0.4` 。中间件机制 `applyMiddleware` 在 `redux` 中是强大且便捷的，利用 `redux` 的中间件我们能够实现日志记录，异步调用等多种十分实用的功能。 `redux` 的中间件主要是通过 `applyMiddleware` 模块实现的。下面，我们就好好的看一下，这个模块到底有什么神奇的魔力。

## 分析

**我们需要知道一件事：中间件模块在redux的源码中是怎么被被调用的？**

中间件的本质是作为 `enhancer` 而存在的。所以，它是通过 `createStore` 方法传递到 `redux` 的内部中的。

```js
// line 48
if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
        throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
}
```

**从这段代码中，我们不难看出，中间件模块是一个高阶函数。其函数签名可以表述如下：**

>const applyMiddleware = (各个中间件列表) => (createStore(创建store)) => (reducer(reducer集合), preloadedState(初始state)) => {}

**而在redux中我们应该怎么使用它呢？**

```js
createStore(reducers, applyMiddleware(thunk))
```

到这里，我们知道了redux是怎么使用中间件的。下面，我们就要详细的解释：redux的源码内部是怎么处理中间件的。为了帮助大家更好的理解，先给大家扯点东西。redux的中间件都是遵循一定规范的。不管是官方的中间件还是我们以后需要自己写的中间件，其函数签名都是一定的。也就是说，中间件的基本的格式都是一样的，接收的参数也是redux注入进去的。下面就是redux中间件的基本格式：

>const reduxMiddleware = ({dispatch, getState}[简化的store]) => (next[上一个中间件的dispatch方法]) => (action[实际派发的action对象]) => {}

```js
function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```


## applyMiddleware.js 源码

```js
import compose from './compose'

/**
 * Creates a store enhancer that applies middleware to the dispatch method
 * of the Redux store. This is handy for a variety of tasks, such as expressing
 * asynchronous actions in a concise manner, or logging every action payload.
 *
 * See `redux-thunk` package as an example of the Redux middleware.
 *
 * Because middleware is potentially asynchronous, this should be the first
 * store enhancer in the composition chain.
 *
 * Note that each middleware will be given the `dispatch` and `getState` functions
 * as named arguments.
 *
 * @param {...Function} middlewares The middleware chain to be applied.
 * @returns {Function} A store enhancer applying the middleware.
 */
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```


## 参考链接

- <a href="https://github.com/reduxjs/redux" target="_blank">https://github.com/reduxjs/redux</a>
- <a href="https://segmentfault.com/a/1190000011883546" target="_blank">https://segmentfault.com/a/1190000011883546</a>