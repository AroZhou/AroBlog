---
layout:     post
title:      JS原生WebSocket
subtitle:   对WebSocket的JS原生实现与封装
date:       2019-08-09
author:     Aro
header-img: img/post-bg-websocket.jpg
catalog: true
tags:
    - JS
    - 前端
    - WebSocket
---

# 前言

>由于<a href="https://www.zzex.me" target="_blank">公司</a>业务的需求 要将原本交易中心的部分由Http请求改为用WebSocket推送，第一次接触，就从原生JS开始，手摸手一步一步实现一个WebSocket。项目的前后端都可能有设计不到位的地方，本文只希望能够触类旁通，对于不同的业务场景仍需要进行一些修改。

## 先贴上主要业务代码

>在业务场景下，需要切换不同的交易对 ticker，实现数据的推送，首先是`websocket.js`文件

```js
const WSS_URL = 'wss://zzexwebsocket.info/ws/v1';
let Socket = null;
let timerPingInterval = null;

/**建立连接 */
function createSocket() {
  Socket = new WebSocket(WSS_URL);
  Socket.onopen = onopenWS;
  Socket.onmessage = onmessageWS;
  Socket.onerror = onerrorWS;
  Socket.onclose = oncloseWS;
}

// 发送心跳
function onopenWS() {
  sendPing()
}

// 连接失败重连
function onerrorWS() {
  console.log('onerrorWS')
  clearInterval(timerPingInterval)
  Socket.close()
  createSocket()
}

// 处理数据
function onmessageWS(e) {
  window.dispatchEvent(new CustomEvent('onmessageWS', {
    detail: {
      data: e
    }
  }))
}

// 发送数据
function sendWSPush(obj) {
  if (Socket !== null && Socket.readyState === 3) {
    Socket.close()
    createSocket()
  } else if (Socket.readyState === 1) {
    Socket.send(JSON.stringify(obj))
  } else if (Socket.readyState === 0) {
    setTimeout(() => {
      Socket.send(JSON.stringify(obj))
    }, 3000)
  }
}

// 关闭
function oncloseWS() {
  clearInterval(timerPingInterval)
  console.log('websocket已断开')
}

// 发送心跳
function sendPing() {
  Socket.send('ping')
  timerPingInterval = setInterval(() => {
    Socket.send('ping')
  }, 3000)
}

```

>接下来引入`websocket.js`，处理数据

```js
let ticker = 'zzexzc'
let depthAction = {
    "channel": `depth.${ticker}`
}
let getData = function (e) {
    console.log(JSON.parse(e.detail.data.data))
};

createSocket()
sendWSPush(depthAction)

//监听ws数据响应
window.addEventListener('onmessageWS', getData);

//根据需要，销毁事件监听
window.removeEventListener('onmessageWS', getData)
```


## 为什么需要WebSocket?

交易中心的数据原本是通过Http请求获得，通信只能由客户端发起。这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就会非常麻烦。我们只能用“轮询”：每隔一段时间，就发出一个询问，了解服务器有没有新的信息。

![20190809HTTP-LONG-POLLING](/img/HTTP-LONG-POLLING.png)

轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。


## 简介

WebSocket 协议在2008年诞生，2011年成为国际标准。<a href="https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket#%E6%B5%8F%E8%A7%88%E5%99%A8%E5%85%BC%E5%AE%B9%E6%80%A7" target="_blank">大部分浏览器都已经支持</a>
它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话。

![20190809Http&WebSocket](/img/Http&WebSocket.png)

还有很多特点：
- 建立在 TCP 协议之上，服务器端的实现比较容易。
- 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
- 数据格式比较轻量，性能开销小，通信高效。
- 可以发送文本，也可以发送二进制数据。
- 没有同源限制，客户端可以与任意服务器通信。
- 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。


## API

### 构造函数

`WebSocket(url[, protocols])` 返回一个 `WebSocket` 对象

- `url`
要连接的URL；这应该是WebSocket服务器将响应的URL。
- `protocols`
可选；单协议字符串或者包含协议字符串的数组。


### 常量(4)

Constant | Value
WebSocket.CONNECTING | 0
WebSocket.OPEN | 1
WebSocket.CLOSING | 2
WebSocket.CLOSED | 3

以上是WebSocket 构造函数的原型中存在的一些常量，可通过 WebSocket.readyState 对照上述常量判断 WebSocket 连接 当前所处的状态


### 属性(10)

下面只列举业务中使用的属性
- `WebSocket.onclose`
用于指定连接关闭后的回调函数
- `WebSocket.onerror`
用于指定连接失败后的回调函数
- `WebSocket.onmessage`
用于指定当从服务器接受到信息时的回调函数
- `WebSocket.onopen`
用于指定连接成功后的回调函数
- `WebSocket.readyState`
当前的链接状态


### 方法(2)

- `WebSocket.close([code[, reason]])`
关闭当前链接
    - code 可选，一个数字状态码，它解释了连接关闭的原因。如果没有传这个参数，默认使用1005。
    - reason 可选，一个人类可读的字符串，它解释了连接关闭的原因。这个UTF-8编码的字符串不能超过123个字节。
- `WebSocket.send(data)`
向服务器发送数据，必须是下列类型之一
    - USVString
    - ArrayBuffer 
    - Blob
    - ArrayBufferView


# 参考链接

- <a href="https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket" target="_blank">https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket</a>

