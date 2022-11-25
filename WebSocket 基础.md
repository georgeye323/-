内部参考代码——家乐福项目。

---

参考资料：

1. [WebSocket 教程 - 阮一峰](http://www.ruanyifeng.com/blog/2017/05/websocket.html)
2. [The WebSocket Protocol](http://websocket.org/aboutwebsocket.html)
3. [WebSocket 是什么原理？为什么可以实现持久连接？ - 知乎](https://www.zhihu.com/question/20215561)
4. [WebSocket - MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
5. [Nginx代理webSocket时60s自动断开, 怎么保持长连接 - 晴识明月](https://blog.csdn.net/cm786526/article/details/79939687)
6. [python 利用websocket实现服务器向客户端推送消息 - 晴识明月](https://blog.csdn.net/cm786526/article/details/79900948)

---

## 简介
### 什么是 WebSocket
WebSocket protocol 是 HTML5 定义的一种新的标准协议（RFC6455），它实现了浏览器与服务器的全双工通信（full-duplex）。

### 为什么需要 WebSocket
传统的 HTTP+HTML 方案只适用于客户端主动发起请求的场景，而无法满足服务器端发起的通信要求。而 `Ajax` 和 `Long poll` 等基于传统HTTP的动态客户端技术使用轮询，耗费了大量的网络带宽和计算资源。
WebSocket 相对于普通的 Socket 通信，在应用层定义了基本的交互流程，使得 Tornado 等服务器框架和JavaScript 客户端可以构建出标准的 WebSocket 模块。

### WebSocket 的特点
最大的特点，当然是服务器端可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话。

其他特点包括：

- 建立在 TCP 协议之上，服务器端的实现比较容易。
- 与 HTTP 协议有良好的兼容性。默认端口也是 80 和 443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
- 数据格式比较轻量，性能开销小，通信高效。
- 可以发送文本，也可以发送二进制数据。
- 没有同源限制，客户端可以与任意服务器通信。
- 协议标识符是 `ws`（如果加密，则为 `wss`），服务器网址就是 URL。

## WebSocket 握手原理
WebSocket 的通信原理是在客户端与服务器之间建立 TCP 持久链接，从而使得当服务器有消息需要推送给客户端时能够进行即时通信。

虽然 WebSocket 不是 HTTP，但在握手阶段，仍然使用了 HTTP 协议进行传输。

### 客户端握手请求
客户端通过发送含有特殊字段的 HTTP Request 告诉服务器需要建立一个 WebSocket 连接，特殊字段如下
```
GET ws://echo.websocket.org/?encoding=text HTTP/1.1
Origin: http://websocket.org
Connection: Upgrade
Sec-WebSocket-Key: uRovscZjNol/umbTt5uKmw==
Upgrade: websocket
Sec-WebSocket-Version: 13
```

含义是：

- 客户端希望建立一个 WebSocket 链接，`ws` 对应 `http`，`wss` 对应 `https`；
- 客户端使用的 WebSocket 版本是 13，密钥是 `uRovscZjNo1/umbTt5uKmw==`（**不用于加密，用于标识该连接**）。
- `Origin` 由浏览器添加，**WebSocket协议本身不要求同源策略（Same-origin Policy）**，但服务器可以根据 `Origin` 拒绝 WebSocket 请求；

还有可能存在一个 `Sec-WebSocket-Protocol: chat` 字段，由用户定义，用来区分同 URL 下，不同的服务所需要的协议。

### 服务端握手响应
服务端如果同意 WebSocket 链接则返回类似的 Response，特殊字段如下
```
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: WebSocket
Access-Control-Allow-Origin: http://websocket.org
Access-Control-Allow-Credentials: true
Sec-WebSocket-Accept: rLHCkw/SKsO9GAH/ZSFhBATDKrU=
Access-Control-Allow-Headers: content-type
```

- 服务器返回 `HTTP 101`，服务器已经将本连接转换为 WebSocket 链接；
- `Sec-WebSocket-Accept` 经过服务器确认并密过后的 `Sec-WebSocket-Key`，用于客户端确认服务器身份。
- 同源策略的话看服务器的实现；

至此，在客户端和服务端之间已经建立了一个 TCP 持久长链接，双方已经可以随时向对方发送消息。

## HTML5 客户端实现
客户端围绕着 `WebSocket` 对象展开。

在 JavaScript 中可以通过如下代码初始化 `WebSocket` 对象，在代码中需要给 WebSocket 构造函数传入服务器的 URL 地址，URL 是 `ws` 或 `wss` 开头的 URL。
```javascript
var Socket = new WebSocket(url);
```

### 回调函数属性
可以为该对象的如下事件制定处理函数以响应它们。

- `WebSocket.onopen` 此事件发生在 WebSocket 链接建立时；
- `WebSocket.onmessage` 此事件发生在收到了来自服务器的消息时；
- `WebSocket.onerror` 此事件发生在通信过程中有任何错误时；
- `WebSocket.onclose` 此事件发生在与服务器的链接关闭时。

### 普通属性

- `WebSocket.readyState`返回实例对象的当前状态，共四种
   - `WebSocket.CONNECTING` 值为0，表示正在连接。
   - `WebSocket.OPEN` 值为1，表示连接成功，可以通信了。
   - `WebSocket.CLOSING`值为2，表示连接正在关闭。
   - `WebSocket.CLOSED` 值为3，表示连接已经关闭，或者打开连接失败。
- `webSocket.bufferedAmount` 表示还有多少字节的二进制数据没有发送出去，可以用来判断发送是否结束。

### 主动操作方法
除事件外，还可以通过 WebSocket 对象的两个方法进行主动操作：

- `WebSocket.send(data)` 向服务器发送消息；
- `WebSocket.close()` 主动关闭现有连接。

## Tornado 服务端实现
Tornado定义了`tornado.websocket.WebSocketHandler`类用于处理 WebSocket 链接的请求。

### 消息处理方法

- `open()`：在新链接建立时调用此方法。在本方法中，可以像在 `get()`、`post()` 中一样使用 `get_argument()` 函数获取客户端提交的参数，以及用 `get_secure_cookie()`、`set_secure_cookie()` 等方法操作 cookie。
- `on_message(message)`：收到来自客户端消息时调用；
- `on_close()`：在 WebSocket 连接关闭时调用。

### 主动操作方法

- `write_message(message, binary=False)`：用于写消息；
- `close(code=None, reason=None)`：主动关闭 WebSocket 连接。

### 其他方法

- `check_origin(origin)` 重新实现自定义的同源检查策略。`origin` 参数是从 header 中获得的 `Origin`，如果客户端没有发送 `Origin` header，该方法不会被调用。返回 `True` 表示接受请求，返回 `False` 表示拒绝请求。Tornado 4.0 以上版本，默认情况下，只允许header中 `Origin` 和 `Host` 域名一致的请求。

## Nginx 配置
主要是对 `Upgrade` 和 `Connection` header 的设置。
```nginx
location ~ /api/kernels/ {
    proxy_pass          http://localhost:8888;
    proxy_set_header    Host       $host;
    proxy_http_version  1.1;
    proxy_set_header    Upgrade    "websocket";
    proxy_set_header    Connection "Upgrade";
    proxy_read_timeout  86400;
}
```

## 保持连接
大概有以下几种方式：

1. 修改 Nginx 连接超时时间 `proxy_read_timeout`；
2. 客户端定时发送心跳包维持连接；
3. 客户端异常断开时自动重连。
