# 16 Creating REST APIs with Spring MVC

## 16.1 Getting REST

### 16.1.1 The fundamentals of REST

RPC是面向服务的，关注actions和verbs，而REST是面向资源的，强调组成应用的事物和名词。

_Representational State Transfer _\(REST\)：

* Representational——资源可以被表现为任何形式，包括XML，JSON，甚至HTML，最适合资源消费者的形式就可以了。
* State——REST更多关注资源的状态，而不是对资源采取的动作。
* Transfer——REST涉及资源的传递，从一个应用传到另一个。

Put more succinctly, REST is about transferring the state of resources—in a representational form that is most appropriate for the client or server—from a server to a client \(or vice versa\).

资源通过URLs定位，通过HTTP方法定义动作：

* Create—POST
* Read—GET
* Update—PUTorPATCH
* Delete—DELETE

上面只是常见的映射，不是严格要求，比如有拿PUT创建新资源和POST更新资源的。

### 16.1.2 How Spring supports REST

略。

## 16.2 Creating your first REST endpoint

Spring offers two options to transform a resource’s Java representation into the representation that’s shipped to the client:

* Content negotiation—A view is selected that can render the model into a representation to be served to the client.
* Message conversion—A message converter transforms an object returned from the controller into a representation to be served to the client.

### 16.2.1 Negotiating resource representation

```
@Bean
public ViewResolver cnViewResolver() {
  return new ContentNegotiatingViewResolver();
}
```

NegotiatingViewResolver的工作主要有两步：

1. Determine the requested media type\(s\).
2. Find the best view for the requested media type\(s\).

**DETERMINING THE REQUESTED MEDIA TYPES**

如果URL有一个文件后缀，则根据后缀确定（如.json对应 application\/json ），如果没法确定，再看Accept请求头，再不行，就使用 default content type 。

**INFLUENCING HOW MEDIA TYPES ARE CHOSEN**

可以使用 ContentNegotiationManager 来：

* Specify a default content type to fall back to if a content type can’t be derived from the request.
* Specify a content type via a request parameter.
* Ignore the request’s Accept header.
* Map request extensions to specific media types.
* Use the Java Activation Framework \(JAF\) as a fallback option for looking up
  media types from extensions.

有3种方式配置ContentNegotiationManager：

* Directly declare a bean whose type is ContentNegotiationManager.
* Create the bean indirectly via ContentNegotiationManagerFactoryBean.
* Override the configureContentNegotiation\(\) method of WebMvcConfigurerAdapter.

```
<bean id="contentNegotiationManager"
  class="org.springframework.http.ContentNegotiationManagerFactoryBean"
  p:defaultContentType="application/json">
```

```
@Override
public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
  configurer.defaultContentType(MediaType.APPLICATION_JSON);
}

@Bean

public ViewResolver cnViewResolver(ContentNegotiationManager cnm) {
  ContentNegotiatingViewResolver cnvr = new ContentNegotiatingViewResolver();
  cnvr.setContentNegotiationManager(cnm);
  return cnvr;
}
```

**THE BENEFITS AND LIMITATIONS OF CONTENTNEGOTIATINGVIEWRESOLVER**

The key benefit of using ContentNegotiatingViewResolver is that it layers REST resource representation on top of the Spring MVC with no change in controller code. The same controller method that serves human-facing HTML content can also serve JSON or XML to a non-human client.

ContentNegotiatingViewResolver also has a serious limitation. As a ViewResolver implementation, it only has an opportunity to determine how a resource is rendered to a client. It has no say in what representations a controller can consume from the client. If the client is sending JSON or XML, then ContentNegotiatingViewResolver isn’t much help.

The View chosen renders the model—not the resource—to the client.

![](/assets/QQ20160913-1.png)

由于种种限制，不建议使用 ContentNegotiatingViewResolver 。

### 16.2.2 Working with HTTP message converters 





