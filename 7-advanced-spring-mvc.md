# 7 Advanced Spring MVC

## 7.1 Alternate Spring MVC configuration

### 7.1.1 Customizing DispatcherServlet configuration

可以覆盖 AbstractAnnotationConfigDispatcherServletInitializer 的 customizeRegistration\(\) 方法来自定义配置，该方法在注册完DispatcherServlet之后被调用，比如你可以配置 Servlet 3.0 multipart support ：

```
@Override
protected void customizeRegistration(Dynamic registration) {
  registration.setMultipartConfig(
      new MultipartConfigElement("/tmp/spittr/uploads"));
}
```

### 7.1.2 Adding additional servlets and filters

![](/assets/QQ20160911-1.png)

![](/assets/QQ20160911-2.png)

To register one or more filters and map them to DispatcherServlet, all you need to do is override the getServletFilters\(\) method of AbstractAnnotationConfigDispatcherServletInitializer：

```
@Override
protected Filter[] getServletFilters() {
  return new Filter[] { new MyFilter() };
}
```

### 7.1.3 Declaring DispatcherServlet in web.xml

略。

