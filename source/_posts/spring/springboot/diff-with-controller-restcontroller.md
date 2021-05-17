---
title: Controller和RestController的区别
date: 2021-05-11 14:26:40
categories:
  - spring
  - springboot
tags:
  - springboot
---

### 共同点

都是用来表示Spring某个类的是否可以接收HTTP请求

### 不同点

#### 返回结果

在SpringMVC中，经常会使用注解 的方式来定义一个控制器。
最常用的控制器注解`@Controller`，可以在控制器类中写各种业务方法，然后返回数据。
一般数据的返回分成两大种，页面 和 json 格式数据

##### 页面

1. 直接返回视图（页面）名称

```java
@RequestMapping("/index")
String index(){
	return "index";
}
```

2. ModelAndView对象

```java
@RequestMapping("/user")
public ModelAndView user(){
    Map<String,Object>values= new HashMap<>();
    values.put("id",1L);
    values.put("name","test");
    return new ModelAndView("user", values);
}
```

返回页面和相应数据。一般配合 Thymeleaf 、 Velocity、FreeMarke使用。spring boot 官方推荐使用 Thymeleaf 。该方式类似早期的jsp页面，将数据渲染后的页面返回前台

##### json

现在一般应用都是前后分离，所以返回json数据的方式使用更频繁。

我们有两种方式将返回的结果转化为json：

1. 使用 `@Controller`（修饰类） 和`@ResponseBody`（修饰方法） 两个注解，当然`@ResponseBody`还可以直接修饰类。
2. 如果某个控制器类设计初衷就是返回json数据，那么该类可以使用简化方式 `@RestCotroller`

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller  //控制器注解
@ResponseBody   //返回数据会被解析成json
public @interface RestController {
    @AliasFor(
        annotation = Controller.class
    )
    String value() default "";
}
```

从上面的源码中可以清晰的看到`@RestController = @Controller + @ResponseBody`

##### 如何抉择

使用`@Controller`修饰类，可以根据需要返回各种我们所需的数据（json,ModelAndView,静态页面），而使用`RestController`修饰类，最后返回结果都会被解析成json字符串，适合所有方法返回值都是json数据

参考：https://blog.csdn.net/yamadeee/article/details/80313068