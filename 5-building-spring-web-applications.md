# 5 Building Spring web applications

## 5.1 Getting started with Spring MVC

![](/assets/QQ20160910-1.png)

1. Request首先到达DispatcherServlet，它是一个前端控制器，代理接收所有请求。
2. DispatcherServlet咨询handler mappings，通过URL计算出应该请求那个Spring MVC 控制器。
3. Controller处理请求。（事实上，一个设计良好的控制器自己很少或者不处理请求，而是委派给一个或多个service对象）；
4. Controller处理完请求，把模型数据和逻辑视图名称一起，发回给DispatcherServlet。
5. DispatcherServlet咨询视图解析器，把逻辑视图名映射到具体的视图实现。
6. 视图渲染模型数据，返回Response对象

**配置DispatcherServlet**

以往，DispatcherServlet被配置在web.xml，打包到war包里。不过，多亏了**Servlet 3**和Spring 3.1，这不再是唯一的选择，我们使用Java来配置 DispatcherServlet

![](/assets/QQ20160910-2.png)

任何继承了 AbstractAnnotationConfigDispatcherServletInitializer 的类，会自动被用来 configure DispatcherServlet and the Spring application context in the application’s servlet context。

在 **Servlet 3.0** 环境下，容器会寻找实现了 javax.servlet .ServletContainerInitializer 接口的类，这些类被用来配置 servlet container 。 SpringServletContainerInitializer 实现了该接口，来寻找实现了 WebApplicationInitializer 接口的类，并把配置任务交给它们。Spring 3.2 引入了基本实现类 AbstractAnnotationConfigDispatcherServletInitializer。

**两个Application Contexts**

DispatcherServlet启动时，它创建一个 Spring application context ，并开始加载配置中定义好的beans，getServletConfigClasses\(\)方法返回的正是需要加载的beans的配置类。

但在Spring web应用中，通常有另一个 application context ，由 ContextLoaderListener 创建。 DispatcherServlet 负责加载web组件，比如 controllers, view resolvers, and handler mappings ，而 ContextLoaderListener 负责加载其他beans，通常是驱动应用后端的中间层和数据层组件，由 getRootConfigClasses\(\) 方法返回的配置类来配置。

注意：这种配置DispatcherServlet的方法只有在部署到支持Servlet 3.0的服务器时才能正常工作，如 Apache Tomcat 7 及以上。

**启用Spring MVC**

下面是最简单的一个Spring MVC 配置类：

```
package spittr.config;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@EnableWebMvc
public class WebConfig {
}
```

这个类可以启用Spring MVC，不过还缺点东西：

* 没有配置视图解析器，Spring会默认使用BeanNameViewResolver，查找ID匹配view name且实现了View接口的bean来解析视图。
* 未启用Component-scanning，因此Spring找不到任何控制器，除非在配置中显式声明它们。
* DispatcherServlet处理所有请求，包括静态资源（这通常不是你想要的）

下面是一个最小可用配置类：
![](/assets/QQ20160910-3.png)

![](/assets/QQ20160910-4.png)

## 5.2 Writing a simple controller





