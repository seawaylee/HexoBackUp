---
title: Chapter06 渲染Web试图 
date: 2017-03-09 23:42:53
tags: [JavaWeb,Spring]
---

我们最常使用的JSP文件是使用InternalResourceViewResolver试图解析器来渲染试图。
在第5章中，我们用这个视图解析器进行了配置，为了得到试图的名字，会使用"/WEB-INF/views/"前缀和".jsp"后缀，从而确定来渲染模型的JSP文件的物理位置。
SpringMVC规定以了一个名为ViewResolver的接口，它大致如下所示:

<!--more-->

```java
public interface ViewResolver
{
    View resolveViewName(String viewName,Locale locale);
}
```
当给resolveViewName()方法传入一个视图名和Locale对象时，他会返回一个View实例。View是另外一个接口，如下所示：


```java
public interface View
{
    String getContentType();
    void render(Map<String,?> model, HttpServletRequest request, HttpServletResponse response) ;
}
```

View接口的任务就是接受模型以及Servlet的request和response对象，并将其输出结果渲染到response中。
这看起来非常简单，我们所需要做的就是编写ViewResolver和View的实现，将要渲染的内容放到response中，进而展现到用户的浏览器中。虽然我们可以编写者两个实现类，但是一般情况下，直接使用Spring提供的多个内置的实现就可以了。一般我们用InternalResourceViewResolver，其他类型请自行查阅。



