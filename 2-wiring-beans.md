# 2 Wiring beans

## 2.1 Spring配置选项

Spring提供三种配置beans的方法，XML，Java和自动装配，可以混用，一般推荐自动装配，然后是JavaConfig，最后才是XML。

## 2.2 自动装配

主要是两方面： _Component scanning_ 和 _Autowiring_ 。

给类加上 @Component 注解，再加入一个拥有 @Configuration 和 @ComponentScan 的配置类，Spring就能扫描到所有标注了Component的类，使它成为一个bean。

每个bean都有一个唯一ID，@Component的bean对应的ID默认是对应的类名把第一个字母变成小写，当然也可以指定id： @Component\("lonelyHeartsClub"\)

@ComponentScan 的默认扫描路径是配置类所在的包以及下面的子包，可以设置 basePackages 和 basePackageClasses

通过 @Autowired 注入依赖的bean。如果找不到相应的bean，就会抛出异常，可以加上 @Autowired\(required = false\) 避免抛出异常，但是需要注意null的情况！

## 2.3 JavaConfig

在 @Configuration 配置类中，使用 @Bean 注解定义bean：

```
@Bean
public CompactDisc sgtPeppers() {
  return new SgtPeppers();
}
```

bean ID默认是方法名称。

注入可以直接调用方法：

```
@Bean
public CDPlayer cdPlayer() {
  return new CDPlayer(sgtPeppers());
}
```

Spring会拦截sgtPeppers\(\)调用，以确保返回该方法定义的bean，而不是再一次调用。默认情况下，Spring中所有的bean都是单例。

## 2.4 XML配置

略。

## 2.5 导入与混合配置

可以使用 @Import 引入 Java 配置类，使用 @ImportResource 引入 XML 配置文件。



