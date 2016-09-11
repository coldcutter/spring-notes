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

## 7.2 Processing multipart form data

### 7.2.1 Configuring a multipart resolver

DispatcherServlet没有实现 multipart 请求的解析逻辑，而是代理给 Spring’s MultipartResolver 接口，从Spring 3.1开始，Spring提供了两个开箱即用的实现：

* CommonsMultipartResolver—Resolves multipart requests using Jakarta Commons FileUpload
* StandardServletMultipartResolver—Relies on Servlet 3.0 support for multipart requests \(since Spring 3.1\)

一般来讲，你应该优先选择 StandardServletMultipartResolver ，因为Servlet容器本身支持，不需要额外的依赖。如果你部署到Servlet 3.0以前的容器或者使用Spring 3.1以前的版本，就只能使用 CommonsMultipartResolver 了。

**RESOLVING MULTIPART REQUESTS WITH SERVLET 3.0**

```
@Bean
public MultipartResolver multipartResolver() throws IOException {
  return new StandardServletMultipartResolver();
}
```

如果你想指定最大文件大小或者临时保存路径，你只能在servlet配置中指定，More specifically, you must configure multipart details as part of DispatcherServlet’s configuration in web.xml or in the servlet initializer class.（见7.1.1）

除了路径，你还可以指定：

* The maximum size \(in bytes\) of any file uploaded. By default there is no limit.
* The maximum size \(in bytes\) of the entire multipart request, regardless of how
  many parts or how big any of the parts are. By default there is no limit.
* The maximum size \(in bytes\) of a file that can be uploaded without being written to the temporary location. The default is 0, meaning that all uploaded files
  will be written to disk.

```
@Override
protected void customizeRegistration(Dynamic registration) {
  registration.setMultipartConfig(
      new MultipartConfigElement("/tmp/spittr/uploads",
          2097152, 4194304, 0));
}
```

**CONFIGURING A JAKARTA COMMONS FILEUPLOAD MULTIPART RESOLVER**

```
@Bean
public MultipartResolver multipartResolver() throws IOException {
  CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
  multipartResolver.setUploadTempDir(new FileSystemResource("/tmp/spittr/uploads"));
  multipartResolver.setMaxUploadSize(2097152);
  multipartResolver.setMaxInMemorySize(0);
  return multipartResolver;
}
```

### 7.2.2 Handling multipart requests

![](/assets/QQ20160911-3.png)

@RequestPart\("profilePicture"\) byte\[\] profilePicture

**RECEIVING A MULTIPARTFILE**

使用byte\[\]不太方便，使用MultipartFile：

![](/assets/QQ20160911-4.png)

**RECEIVING THE UPLOADED FILE AS A PART**

![](/assets/QQ20160911-5.png)

**注意：如果你使用Part来接收参数，你就不需要配置 StandardServletMultipartResolver bean了，当你使用 MultipartFile 的时候才需要它。**

