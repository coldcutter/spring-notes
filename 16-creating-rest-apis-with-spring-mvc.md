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

Message conversion更加直接，直接把控制器返回的数据转换成资源展示形式，没有model和view。 DispatcherServlet 通过检查 request’s Accept 请求头，使用对应的MessageConverter。

![](/assets/QQ20160915-1.png)

![](/assets/QQ20160915-2.png)

**RETURNING RESOURCE STATE IN THE RESPONSE BODY**

绕过常规的model\/view流程最简单的方法是给控制器方法加上 @ResponseBody 注解。

@RequestMapping\(method=RequestMethod.GET, produces="application\/json"\)

this method will only handle requests whose Accept header includes application\/json。

**RECEIVING RESOURCE STATE IN THE REQUEST BODY**

类似的，@RequestBody注解可以把客户端传过来的资源表示转换成Java对象。

Spring 通过 Content-Type 请求头找到合适的MessageConverter来转换reqeust body

@RequestMapping\( method=RequestMethod.POST, consumes="application\/json"\)

This tells Spring that this method will only handle POST requests to \/spittles if the request’s Content-Type header is application\/json.

**DEFAULTING CONTROLLERS FOR MESSAGE CONVERSION**

Spring 4.0 引入了 @RestController 注解，If you annotate your controller class with @RestController instead of @Controller, Spring applies message conversion to all handler methods in the controller.

## 16.3 Serving more than resources

### 16.3.1 Communicating errors to the client

Spring offers a few options for dealing with such scenarios:

* Status codes can be specified with the @ResponseStatus annotation.
* Controller methods can return a ResponseEntity that carries more metadata concerning the response.
* An exception handler can deal with the error cases, leaving the handler methods to focus on the happy path.

**WORKING WITH RESPONSEENTITY**

ResponseEntity is an object that carries metadata \(such as headers and the status code\) about a response in addition to the object to be converted to a resource representation. There’s no need to annotate the method with @ResponseBody if it returns ResponseEntity.

**HANDLING ERRORS**

```
@ExceptionHandler(SpittleNotFoundException.class)
public ResponseEntity<Error> spittleNotFound(SpittleNotFoundException e) {
  long spittleId = e.getSpittleId();
  Error error = new Error(4, "Spittle [" + spittleId + "] not found");
  return new ResponseEntity<Error>(error, HttpStatus.NOT_FOUND);
}
```

```
public class SpittleNotFoundException extends RuntimeException {
  private long spittleId;
  public SpittleNotFoundException(long spittleId) {
    this.spittleId = spittleId;
  }

  public long getSpittleId() {
    return spittleId;
  }
}
```

You can clean things up a little more, though. Now that you know that spittleById\(\) will return a Spittle and that the HTTP status will always be 200 \(OK\), you no longer need to use ResponseEntity and can replace it with @ResponseBody:

```
@RequestMapping(value="/{id}", method=RequestMethod.GET)
public @ResponseBody Spittle spittleById(@PathVariable long id) {
  Spittle spittle = spittleRepository.findOne(id);
  if (spittle == null) {
    throw new SpittleNotFoundException(id);
  }
  return spittle;
}
```

Knowing that the error handler method always returns an Error and always responds with an HTTP status code of 404 \(Not Found\), you can apply a similar cleanup process to spittleNotFound\(\):

```
@ExceptionHandler(SpittleNotFoundException.class)
@ResponseStatus(HttpStatus.NOT_FOUND)
public @ResponseBody Error spittleNotFound(SpittleNotFoundException e) {
  long spittleId = e.getSpittleId();
  return new Error(4, "Spittle [" + spittleId + "] not found");
}
```

### 16.3.2 Setting headers in the response

When creating a new resource, it’s considered good form to communicate the resource’s URL to the client in the Location header of the response.

![](/assets/QQ20160915-3.png)

Rather than construct the URI manually, Spring offers some help in the form of UriComponentsBuilder.

![](/assets/QQ20160915-4.png)

## 16.4 Consuming REST resources

略。





