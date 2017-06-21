---
title: SpringInAction读书笔记(七) SpringMVC高级技术 
date: 2017-03-09 23:44:02
tags: [JavaWeb,Spring]
---

## 7.1 SpringMVC的环境搭建方式
**Java配置方式**

<!--more-->

全局配置


```java
public class SpitterWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer
{

    /**
     * Spring上下文配置
     * ContextLoaderListener
     * @Author NikoBelic
     * @Date 09/01/2017 20:40
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[]{RootConfig.class};
    }

    /**
     * SpringMVC上下文配置
     * DisparcherServlet
     * @Author NikoBelic
     * @Date 09/01/2017 20:41
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[]{WebConfig.class};
    }

    /**
     * SpringMVC请求拦截,将DispatcherServlet映射到/
     * @Author NikoBelic
     * @Date 09/01/2017 20:41
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 配置文件上传限制
     * @Author NikoBelic
     * @Date 09/01/2017 20:40
     */
    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setMultipartConfig(new MultipartConfigElement("/Users/lixiwei-mac/Desktop/tmp",20971520, 41943040, 0));
    }
}

```

Spring上下文配置（ContextLoaderListener）

```java
/**
 * @author NikoBelic
 * @create 09/01/2017 20:30
 */
//@Configuration
@ComponentScan(basePackages = "chapter07",excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value = EnableWebMvc.class)})
public class RootConfig
{
}
```
SpringMVC配置（DispatcherServlet）


```java
@Configuration
@EnableWebMvc
@ComponentScan("chapter07.controller")
public class WebConfig extends WebMvcConfigurerAdapter
{

    /**
     * 配置视图解析器
     * @Author NikoBelic
     * @Date 09/01/2017 20:37
     */
    @Bean
    public ViewResolver viewResolver()
    {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        resolver.setViewClass(JstlView.class);
        return resolver;
    }

    /**
     * 配置静态资源的处理
     * @Author NikoBelic
     * @Date 09/01/2017 20:37
     */
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    /**
     * 配置文件上传解析器
     * @Author NikoBelic
     * @Date 09/01/2017 20:38
     */
    @Bean
    public MultipartResolver multipartResolver() throws IOException {
        return new StandardServletMultipartResolver();
    }

}
```

**XML配置方式（略） 可以参考前几章**

## 7.2 处理Multipart文件上传
web相关配置已经包含在7.1中

jsp页面

```html
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="sf" uri="http://www.springframework.org/tags/form" %>
<%@ page isELIgnored="false" %>
<%@ page session="false" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>RegisterForm</title>
</head>
<body>
<h1>Register</h1>
<%--这里Form表单没有设置action,因此他会将请求提交到与展现时相同的URL路径上--%>
<form method="post" enctype="multipart/form-data">
    Username:<input type="text" name="username"/><br/>
    Password:<input type="password" name="password"/><br/>
    FirstName:<input type="text" name="firstName"/><br/>
    LastName:<input type="text" name="lastName"/><br/>
    ProfilePic:<input type="file" name="profilePic" accept="image/jpeg,image/png,image/gif"/><br/>
    <input type="submit" value="Register"/>
</form>

使用Spring表单绑定标签
<%--<sf:form method="post" commandName="spitter">--%>
    <%--Username:<sf:input path="username"/><br/>--%>
    <%--Password:<sf:password path="password"/><br/>--%>
    <%--<input type="submit" value="Register">--%>
<%--</sf:form>--%>
</body>
</html>
```

控制器


```java
@Controller
@RequestMapping(value = "weibo")
public class WeiboController
{
    /**
     * 跳转到注册界面
     * @Author NikoBelic
     * @Date 09/01/2017 22:35
     */
    @RequestMapping(value = "/register",method = RequestMethod.GET)
    public String showRegister()
    {
        return "weibo/register";
    }

    /**
     * 处理注册流程
     * @Author NikoBelic
     * @Date 09/01/2017 22:36
     */
    @RequestMapping(value = "/register",method = RequestMethod.POST)
    public String processRegister(@RequestPart("profilePic")MultipartFile profilePic, @Valid Spitter spitter, Errors errors, RedirectAttributes model) throws IOException, DuplicateException
    {
        profilePic.transferTo(new File("/Users/lixiwei-mac/Desktop/data/" + profilePic.getOriginalFilename()));
        System.out.println(spitter);
        if (spitter.getUsername().equals("dup"))
            throw new DuplicateException();
        model.addAttribute("username",spitter.getUsername());
        model.addAttribute("password",spitter.getPassword());
        model.addFlashAttribute(spitter);
        return "redirect:/weibo/profile/{username}";
    }


    @RequestMapping(value = "/profile/{username}",method = RequestMethod.GET)
    public String profile(@PathVariable("username") String username,String password,Model model)
    {
        System.out.println(username);
        System.out.println(password);
        System.out.println(model.containsAttribute("spitter"));
        return "profile";
    }

    /**
     * 控制器异常处理
     * @Author NikoBelic
     * @Date 10/01/2017 08:51
     */
    //@ExceptionHandler(DuplicateException.class)
    //@ResponseBody
    //public String handleDuplicateException()
    //{
    //    return "error/duplicate";
    //}

}

```

customizeRegistration配置指定了临时文件的存储地址（传输完毕后会自动清除）

## 7.3 处理异常
使用@ExceptionHandler可以处理Controller抛出的异常，异常出现后可以跳转到更加友好的提示界面。@ControllerAdvice作为一个Controller的全局配置，对所有@Controller注解的@RequestMapping方法生效。


```java
@ControllerAdvice
public class AppWideExceptionHandler
{
    @ExceptionHandler(DuplicateException.class)
    public String duplicateSpittleHandler()
    {
        return "error/duplicate";
    }
}

```

## 7.4 跨重定向传递数据
redirect：将客户端的请求以get方式重定向到另一个controller，可以用${}路径和?参数的方式传递**简单**数据，但是无法传输Spitter这样的对象。重定向后模型中的数据将会丢失。
forward：将客户端的请求转发到另外一个页面，保留原有的模型数据，但是浏览器地址不会发生改变，因为浏览器不知道自己的请求被转发了。

如果需要使用redirect并想保留model中的数据，可以使用flash属性。使用RedirectAttributes作为model，它会将模型暂存到session中，重定向后可以使用Model取出对应的对象。我们也可以不借助Spring提供的这个方法，自己将对象存储到session中，转发后在从session中取出并清空session。



