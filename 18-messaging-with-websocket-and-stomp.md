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

Working directly with WebSocket \(or SockJS\) is a lot like developing a web application using only TCP sockets. Fortunately, you don’t have to work with raw WebSocket connections. Just as HTTP layers a request-response model on top of TCP sockets, STOMP layers a frame-based wire format to define messaging semantics on top of WebSocket.

STOMP（Simple Text Oriented Messaging Protocol）消息帧由一个命令，一个或多个头，以及一个消息体组成：

```
SEND
destination:/app/marco
content-length:20

{\"message\":\"Marco!\"}
```

Spring provides for STOMP-based messaging with a programming model based on Spring MVC. As you’ll see, handling STOMP messages in a Spring MVC controller isn’t much different from handling HTTP requests.

### 18.3.1 Enabling STOMP messaging

![](/assets/QQ20161007-6@2x.png)

registerStompEndpoints\(\) 注册了 \/marcopolo 作为 STOMP endpoint. It’s the endpoint that a client would connect to before subscribing to or publishing to a destination path.

configureMessageBroker\(\) 配置 message broker，该方法可选，如果不覆盖，会有一个简单内存消息代理用来处理\/topic打头的消息。

![](/assets/QQ20161007-7@2x.png)

当消息到来的时候，the destination prefix 决定了消息如何被处理，\/app打头的会被 @MessageMapping 注解的控制器方法处理，请求broker的消息，包括 @MessageMapping 方法返回值产生的消息，会被路由到消息代理，最终发送给订阅那些 destinations 的客户端。

**ENABLING A STOMP BROKER RELAY**

简单消息代理有限制，只支持STOMP命令的子集，由于它基于内存，因此不适合集群消息处理。对于生产环境，需要使用真正的消息代理，如 RabbitMQ 或 ActiveMQ ，当建立完消息代理之后，修改 configureMessageBroker 方法

```
@Override
public void configureMessageBroker(MessageBrokerRegistry registry) {
  registry.enableStompBrokerRelay("/topic", "/queue");
  registry.setApplicationDestinationPrefixes("/app");
}
```

Depending on which STOMP broker you choose, you may be limited in your choices for the destination prefix. RabbitMQ, for instance, only allows destinations of type \/temp-queue, \/exchange, \/topic, \/queue, \/amq\/queue, and \/reply-queue\/.

![](/assets/QQ20161007-8@2x.png)

By default, the STOMP broker relay assumes that the broker is listening on port 61613 of localhost and that the client username and password are both “guest”，你可以配置：

```
@Override
public void configureMessageBroker(MessageBrokerRegistry registry) {
  registry.enableStompBrokerRelay("/topic", "/queue")
          .setRelayHost("rabbit.someotherserver")
          .setRelayPort(62623)
          .setClientLogin("marcopolo")
          .setClientPasscode("letmein01");
  registry.setApplicationDestinationPrefixes("/app", "/foo");
}
```

### 18.3.2 Handling STOMP messages from the client

![](/assets/QQ20161007-9@2x.png)

Because handleShout\(\) accepts a Shout parameter, the payload of the STOMP message will be converted into a Shout using one of Spring’s message converters.

![](/assets/QQ20161007-10@2x.png)

**PROCESSING SUBSCRIPTIONS**

Any method that’s annotated with @SubscribeMapping will be invoked, much like @MessagingMapping methods, when a STOMP subscription message arrives.

The primary use case for @SubscribeMapping is to implement a request-reply pattern. In the request-reply pattern, the client subscribes to a destination expecting a one-time response at that destination.

```
@SubscribeMapping({"/marco"})
public Shout handleSubscription() {
  Shout outgoing = new Shout();
  outgoing.setMessage("Polo!");
  return outgoing;
}
```

The Shout object is then converted into a message and sent back to the client at the same destination to which the client subscribed.

订阅与HTTP GET的区别，GET是同步请求，而订阅是异步的。

**WRITING THE JAVASCRIPT CLIENT**

![](/assets/QQ20161007-11@2x.png)

### 18.3.3 Sending messages to the client

**SENDING A MESSAGE AFTER HANDLING A MESSAGE**

```
@MessageMapping("/marco")
public Shout handleShout(Shout incoming) {
  logger.info("Received message: " + incoming.getMessage());
  Shout outgoing = new Shout();
  outgoing.setMessage("Polo!");
  return outgoing;
}
```

When an @MessageMapping-annotated method has a return value, the returned object will be converted \(via a message converter\) and placed into the payload of a STOMP frame and published to the broker.

By default, the frame will be published to the same destination that triggered the handler method, but with \/topic as the prefix, But you can override the destination by annotating the method with @SendTo:

```
@MessageMapping("/marco")
@SendTo("/topic/shout")
```

In a similar way, an @SubscribeMapping-annotated method can send a message in reply to a subscription. What’s different with @SubscribeMapping is that the Shout message is sent directly to the client without going through the broker. If you annotate the method with @SendTo, the message will be sent to the destination specified, going through the broker.

**SENDING A MESSAGE FROM ANYWHERE**

![](/assets/QQ20161007-12@2x.png)

The Handlebars template is defined in a separate &lt;script&gt; tag as follows:

```
<script id="spittle-template" type="text/x-handlebars-template">
  <li id="preexist">
    <div class="spittleMessage">{{message}}</div>
    <div>
      <span class="spittleTime">{{time}}</span>
      <span class="spittleLocation">({{latitude}}, {{longitude}})</span>
    </div>
  </li>
</script>
```

![](/assets/QQ20161007-13@2x.png)

## 18.4 Working with user-targeted messages

There are three ways to take advantage of an authenticated user when messaging with Spring and STOMP:

* The@MessageMappingand@SubscribeMappingmethodscanreceiveaPrincipal for the authenticated user.
* Valuesreturnedfromthe@MessageMapping,@SubscribeMapping,and@MessageException methods can be sent as messages to the authenticated user.
* The SimpMessagingTemplate can send messages to a specific user.

### 18.4.1 Working with user messages in a controller

```
@MessageMapping("/spittle")
@SendToUser("/queue/notifications")
public Notification handleSpittle(Principal principal, SpittleForm form) {
  Spittle spittle = new Spittle(principal.getName(), form.getText(), new Date());
  spittleRepo.save(spittle);
  return new Notification("Saved Spittle");
}
```

Consider this line of JavaScript that subscribes to a user-specific destination:

```
stomp.subscribe("/user/queue/notifications", handleNotifications);
```

Notice that the destination is prefixed with \/user. Internally, destinations that are prefixed with \/user are handled in a special way.

![](/assets/QQ20161007-14@2x.png)

UserDestinationMessageHandler’s primary job is to reroute user messages to a destination that’s unique to the user. In the case of a subscription, it derives the target destination by removing the \/user prefix and adding a suffix that’s based on the user’s session. For instance, a subscription to \/user\/queue\/notifications may end up being rerouted to a destination named \/queue\/notifications-user6hr83v6t. 

### 18.4.2 Sending messages to a specific user

![](/assets/QQ20161007-15@2x.png)

## 18.5 Handling message exceptions 

![](/assets/QQ20161007-16@2x.png)







