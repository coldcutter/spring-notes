# 18 Messaging with WebSocket and STOMP

## 18.1 Working with Spring’s low-level WebSocket API

![](/assets/QQ20161007-1@2x.png)

![](/assets/QQ20161007-2@2x.png)

更方便的做法是继承 AbstractWebSocketHandler ：

![](/assets/QQ20161007-3@2x.png)

配置：

![](/assets/QQ20161007-4@2x.png)

前端js：

![](/assets/QQ20161007-5@2x.png)

## 18.2 Coping with a lack of WebSocket support

遗憾的是，并不是所有浏览器都支持WebSocket，here’s a brief list of the minimum versions of several popular browsers that support WebSocket:

* Internet Explorer: 10.0
* Firefox: 4.0 \(partial\), 6.0 \(full\)
* Chrome: 4.0 \(partial\), 13.0 \(full\)
* Safari: 5.0 \(partial\), 6.0 \(full\)
* Opera: 11.0 \(partial\), 12.10 \(full\)
* iOS Safari: 4.2 \(partial\), 6.0 \(full\)
* Android Browser: 4.4

也不是所有应用服务器都支持WebSocket，就算客户端和服务器端都支持，firewall proxies generally block anything but HTTP traffic. They’re not capable or not configured \(yet\) to allow WebSocket communication.

SockJS是一个WebSocket模拟器，尽可能在表面接近 WebSocket API ，底层则更智能：SockJS总是首选 WebSocket ，如果 WebSocket 不可用，it will determine the best available option from the following:

* XHR streaming
* XDR streaming
* iFrame event source
* iFrame HTML file
* XHR polling
* XDR polling
* iFrame XHR polling
* JSONP polling

你不用关心上面这些细节，SockJS会帮你处理。

想要启用SockJS，你需要修改 registerWebSocketHandlers\(\) 方法：

```
@Override
public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
  registry.addHandler(marcoHandler(), "/marco").withSockJS();
}
```

在客户端，你需要引入 sockjs ：

```
<script src="http://cdn.sockjs.org/sockjs-0.3.min.js"></script>
```

然后改一点点18.4的代码：

```
var url = 'marco';
var sock = new SockJS(url);
```

SockJS deals in URLs with the http:\/\/ or https:\/\/ scheme instead of ws:\/\/ and wss:\/\/，你可以使用相对路径。

## 18.3 Working with STOMP messaging



