# 3 Advanced wiring

## 3.1 环境和profiles

不同的环境需要配置不同的beans，比如dev，qa，prod环境下的DataSource是不一样的。可以使用 @Profile\("prod"\) 这样的注解。

在Spring 3.1中，@Profile注解只能在类级别使用（Java 配置类），从Spring 3.2开始，你可以在方法级别使用@Profile注解，与@Bean注解一起。

Spring用两个属性 spring.profiles.active 和 spring.profiles.default 决定哪些profiles是激活的，首先考虑spring.profiles.active设置的值，然后是spring.profiles.default，如果两个都没设置，那就没有激活的profiles，有几个地方来设置：

* As initialization parameters on DispatcherServlet
* As context parameters of a web application
* As JNDI entries
* As environment variables
* As JVM system properties
* Using the @ActiveProfiles annotation on an integration test class

## 3.2 Conditional beans

从Spring 4.0开始， 引入的 @Conditional 注解能够更灵活地选择性配置beans，比如当特定的类库在classpath下、或特定的别的bean被声明、或特定的环境变量被设置的情况下才创建bean。

```
@Bean
@Conditional(MagicExistsCondition.class)
public MagicBean magicBean() {
  return new MagicBean();
}
```

MagicExistsCondition是实现了Condition接口的类：

```
public interface Condition {
  boolean matches(ConditionContext ctxt, AnnotatedTypeMetadata metadata);
}
```

matches如果返回true，则创建对应的bean，ConditionContext和AnnotatedTypeMetadata可以拿到很多帮助我们判断的信息。

## 3.3 自动装配的歧义

在自动注入的时候，如果有多个bean符合要求，Spring就会抛出 NoUniqueBeanDefinitionException 异常。

你可以在bean定义的时候加上 @Primary 注解，这样当有多个bean时，Spring会选择有@Primary注解的那个bean，但是当有多个bean标注了@Primary时，歧义还是有。

另一种更好的方法是使用 @Qualifier 注解，在 @Autowired 的同时加上 @Qualifier\("iceCream"\) 注解，这样就选择了bean ID是iceCream的那个bean。

更精确地讲， @Qualifier\("iceCream"\) 指的是拥有 "iceCream" 限定符的bean，所有的beans会被赋予一个默认与bean ID相同的限定符，这样带来的一个问题就是当类名修改的时候，限定符就会改变，导致找不到bean。解决办法是你可以在bean定义的时候，加上 @Qualifier\("cold"\) 注解来设置限定符，cold用来说明该bean的特性，而不是依赖于bean ID。

## 3.4 beans作用域

默认情况下，所有spring application context中的beans都是单实例的，方便重用。Spring提供以下几种作用域：

* Singleton—One instance of the bean is created for the entire application.
* Prototype—One instance of the bean is created every time the bean is injected into or retrieved from the Spring application context.
* Session—In a web application, one instance of the bean is created for each session.
* Request—In a web application, one instance of the bean is created for each request.

可以使用 @Scope 注解：@Scope\(ConfigurableBeanFactory.SCOPE\_PROTOTYPE\)，指定了scope为prototype，而session和request作用域有所不同：

```
@Component
@Scope(
    value = WebApplicationContext.SCOPE_SESSION,
    proxyMode = ScopedProxyMode.INTERFACES)
public ShoppingCart cart() { ... }
```

注意到proxyMode属性，这是用来解决把session或request域的bean注入到singleton域bean的问题，因为单例bean需要在context加载的时候创建，但是session或request域的bean此时还没有创建，所以Spring会注入一个代理bean，暴露相同的方法。ScopedProxyMode.INTERFACES表示代理bean应该实现ShoppingCart接口，代理请求到相应的实现bean。不过如果ShoppingCart不是一个接口，而是一个具体类，则必须使用CGLib生成一个基于类的代理，要把proxyMode设为ScopedProxyMode.TARGET\_CLASS。

![](/assets/QQ20160904-1.png)

## 3.5 运行时值注入

我们需要在运行时注入值，而不是硬编码到配置类中。
可以使用 @PropertySource 注解和 Environment 接口：

```
@Configuration
@PropertySource("classpath:/com/soundsystem/app.properties")
public class ExpressiveConfig {

  @Autowired
  Environment env;

  @Bean
  public BlankDisc disc() {
    return new BlankDisc(env.getProperty("disc.title"), env.getProperty("disc.artist"));
  }
}
```

还可以使用属性占位符\(property placeholders\)：

```
public BlankDisc(
    @Value("${disc.title}") String title,
    @Value("${disc.artist}") String artist) {
  this.title = title;
  this.artist = artist;
}
```

要使用占位符值，必须配置 PropertyPlaceholderConfigurer 或者  PropertySourcesPlaceholderConfigurer bean，从Spring 3.1开始，推荐 PropertySourcesPlaceholderConfigurer ：

```
@Bean
public static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
  return new PropertySourcesPlaceholderConfigurer();
}
```

**Spring Expression Language（SpEL）**

SpEL功能强大，包括：

**引用beans，属性和方法**：\#{sgtPeppers}、\#{sgtPeppers.artist}、\#{artistSelector.selectArtist\(\)}、\#{artistSelector.selectArtist\(\)?.toUpperCase\(\)}，?.操作符保证null-safe，如果为null，就不调用，返回null。

**在表达式中引用类型**：T\(java.lang.Math\).PI、T\(java.lang.Math\).random\(\)

**正则表达式匹配**：\#{admin.email matches '\[a-zA-Z0-9.\_%+-\]+@\[a-zA-Z0-9.-\]+\.com'}

**集合和数组操作**：\#{jukebox.songs\[4\].title}，\#{jukebox.songs.?\[artist eq 'Aerosmith'\]}（筛选所有符合条件的项组成list）， \#{jukebox.songs.^\[artist eq 'Aerosmith'\]} （返回第一个符合条件的项， .$\[\]是返回最后一个），\#{jukebox.songs.!\[title\]}（返回所有项的title字段组成list）。

