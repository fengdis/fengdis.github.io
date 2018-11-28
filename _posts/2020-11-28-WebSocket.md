---
layout:     post
title:      WebSocket
subtitle:   WebSocket
date:       2018-11-28
author:     fengdi
header-img: img/post-bg-code8.jpeg
catalog: true
tags:
    - WebSocket
    - 协议
    - 消息通讯
---

### 为什么需要 WebSocket？

- 初次接触 WebSocket 的人，都会问同样的问题：我们已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？
- **因为 HTTP 协议有一个缺陷：通信只能由客户端发起**，例如，我们想了解今天的天气，只能是客户端向服务器发出请求，服务器返回查询结果，HTTP 协议做不到服务器主动向客户端推送信息
- **这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦**
- 我们只能使用"轮询"：每隔一段时候，就发出一个询问，了解服务器有没有新的信息，**最典型的场景就是聊天室**。轮询的效率低，非常浪费资源（**因为必须不停连接，或者 HTTP 连接始终打开**）
- 因此，工程师们一直在思考，有没有更好的方法。而 WebSocket 就是这样发明的

### WebSocket 简介

- **WebSocket 最大特点就是服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是双向通信**

![](//upload-images.jianshu.io/upload_images/9434708-ee0a4dd529b29ce3..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/628/format/webp)

WebSocket

- 其特点包括：
  - 建立再 TCP 协议之上，服务器端的实现比较容易
  - 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，**并且握手阶段采用 HTTP 协议**，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器
  - **数据格式比较轻量，性能开销小，通信高效**
  - **可以发送文本、二进制数据**
  - 没有同源限制，客户端可以与任意服务器通信
  - 协议标识符是 ws（如果加密，则为 wss），服务器网址就是 URL

  ```
  ws://example.com:80/some/path
  ```

### 客户端的示例

- 下面是一个网页脚本的例子

```
var ws = new WebSocket("wss://echo.websocket.org");

ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};      

```

### 客户端的 API

- WebSocket 对象作为一个构造函数，用于新建 WebSocket 实例

```
var ws = new WebSocket('ws://localhost:8080');
```

- webSocket.readyState 属性返回实例对象的当前状态，共有四种：
  - CONNECTING：值为0，表示正在连接
  - OPEN：值为1，表示连接成功，可以通信了
  - CLOSING：值为2，表示连接正在关闭
  - CLOSED：值为3，表示连接已经关闭，或者打开连接失败

```
switch (ws.readyState) {
  case WebSocket.CONNECTING:
    // do something
    break;
  case WebSocket.OPEN:
    // do something
    break;
  case WebSocket.CLOSING:
    // do something
    break;
  case WebSocket.CLOSED:
    // do something
    break;
  default:
    // this never happens
    break;
}
```

- 实例对象的 onopen 属性，用于指定连接成功后的回调函数

```
ws.onopen = function () {
  ws.send('Hello Server!');
}
// 如果要指定多个回调函数，可以使用 addEventListener 方法
ws.addEventListener('open', function (event) {
  ws.send('Hello Server!');
});
```

- 实例对象的 onclose 属性，用于指定连接关闭后的回调函数

```
ws.onclose = function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
};
// 如果要指定多个回调函数，可以使用 addEventListener 方法
ws.addEventListener("close", function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
});
```

- 实例对象的 onmessage 属性，用于指定收到服务器数据后的回调函数

```
ws.onmessage = function(event) {
  var data = event.data;
  // 处理数据
};
// 如果要指定多个回调函数，可以使用 addEventListener 方法
ws.addEventListener("message", function(event) {
  var data = event.data;
  // 处理数据
});
```

- 注意，服务器数据可能是文本，也可能是二进制数据（blob对象或Arraybuffer对象）

```
ws.onmessage = function(event){
  if(typeof event.data === String) {
    console.log("Received data string");
  }

  if(event.data instanceof ArrayBuffer){
    var buffer = event.data;
    console.log("Received arraybuffer");
  }
}
```

- 除了动态判断收到的数据类型，也可以使用 binaryType 属性，显式指定收到的二进制数据类型

```
// 收到的是 blob 数据
ws.binaryType = "blob";
ws.onmessage = function(e) {
  console.log(e.data.size);
};

// 收到的是 ArrayBuffer 数据
ws.binaryType = "arraybuffer";
ws.onmessage = function(e) {
  console.log(e.data.byteLength);
};
```

- 实例对象的 send() 方法用于向服务器发送数

  - 发送文本

  ```
  ws.send('your message');
  ```

  - 发送 Blob 对象

  ```
  var file = document
  .querySelector('input[type="file"]')
  .files[0];
  ws.send(file);
  ```

  - 发送 ArrayBuffer 对象

  ```
  var img = canvas_context.getImageData(0, 0, 400, 320);
  var binary = new Uint8Array(img.data.length);
  for (var i = 0; i < img.data.length; i++) {
    binary[i] = img.data[i];
  }
  ws.send(binary.buffer);
  ```

- 实例对象的 bufferedAmount 属性，表示还有多少字节的二进制数据没有发送出去。它可以用来判断发送是否结束

```
var data = new ArrayBuffer(10000000);
socket.send(data);

if (socket.bufferedAmount === 0) {
  // 发送完毕
} else {
  // 发送还没结束
}
```

- 实例对象的 onerror 属性，用于指定报错时的回调函数

```
socket.onerror = function(event) {
  // handle error event
};

socket.addEventListener("error", function(event) {
  // handle error event
});
```