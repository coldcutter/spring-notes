# 6 Rendering web views

## 6.1 Understanding view resolution

```
public interface ViewResolver {
  View resolveViewName(String viewName, Locale locale) throws Exception;
}

public interface View {
  String getContentType();
  void render(Map<String, ?> model,
              HttpServletRequest request,
              HttpServletResponse response) throws Exception;
}
```

![](/assets/QQ20160910-10.png)

## 6.2 Creating JSP views

略。

## 6.3 Defining a layout with Apache Tiles views

略。

## 6.4 Working with Thymeleaf

![](/assets/QQ20160910-11.png)

表单绑定：

![](/assets/QQ20160910-12.png)

By using th:field, you get both a value attribute set to the value of firstName and also a name attribute set to firstName.









