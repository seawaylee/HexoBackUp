---
title: Http请求中Content-Type讲解以及在Spring MVC中的应用
date: 2017-03-29 21:33:14
tags: [JavaWeb]
---

# 引言
>在Http请求中，我们每天都在使用Content-type来指定不同格式的请求信息，但是却很少有人去全面了解content-type中允许的值有多少，
这里将讲解Content-Type的可用值，以及在spring MVC中如何使用它们来映射请求信息。

<!--more-->

# 1 Content-Type 

>MediaType，即是Internet Media Type，互联网媒体类型；也叫做MIME类型，在Http协议消息头中，使用Content-Type来表示具体请求中的媒体类型信息。

```
类型格式：type/subtype(;parameter)? type  
主类型，任意的字符串，如text，如果是*号代表所有；   
subtype 子类型，任意的字符串，如html，如果是*号代表所有；   
parameter 可选，一些参数，如Accept请求头的q参数， Content-Type的 charset参数。
```

例如： Content-Type: text/html;charset:utf-8;

**常见的媒体格式类型如下：**

- text/html ： HTML格式
- text/plain ：纯文本格式      
- text/xml ：  XML格式
- image/gif ：gif图片格式    
- image/jpeg ：jpg图片格式 
- image/png：png图片格式


**以application开头的媒体格式类型：**

- application/xhtml+xml ：XHTML格式
- application/xml     ： XML数据格式
- application/atom+xml  ：Atom XML聚合格式    
- application/json    ： JSON数据格式
- application/pdf       ：pdf格式  
- application/msword  ： Word文档格式
- application/octet-stream ： 二进制流数据（如常见的文件下载）
- application/x-www-form-urlencoded ： <form encType=””>中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）

**另外一种常见的媒体格式是上传文件之时使用的：**

- multipart/form-data ： 需要在表单中进行文件上传时，就需要使用该格式


以上就是我们在日常的开发中，经常会用到的若干content-type的内容格式。


# 2 Spring MVC中关于关于Content-Type类型信息的使用

首先我们来看看RequestMapping中的Class定义：

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
      String[] value() default {};
      RequestMethod[] method() default {};
      String[] params() default {};
      String[] headers() default {};
      String[] consumes() default {};
      String[] produces() default {};
}
```

- value:  指定请求的实际地址， 比如 /action/info之类。
- method：  指定请求的method类型， GET、POST、PUT、DELETE等
- consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
- produces:    指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回
- params： 指定request中必须包含某些参数值是，才让该方法处理
- headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求

其中，consumes， produces使用content-typ信息进行过滤信息；headers中可以使用content-type进行过滤和判断。

# 3 使用示例

## 3.1 SpringMVC最基本配置文件

Web应用配置
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

DispatcherServlet配置
```java
@Configuration
@EnableWebMvc
@ComponentScan("mvc.controller")
public class WebConfig extends WebMvcConfigurerAdapter
{

}
```

Listener配置

```java
@Configuration
public class RootConfig
{
}
```

## 3.2 测试

```java
@Controller
@RequestMapping(value = "/mvc")
public class TestController
{

    /**
     * 测试启动
     *
     * @Author SeawayLee
     * @Date 2017/03/29 22:08
     */
    @ResponseBody
    @RequestMapping(value = "/")
    public Object helloMVC(HttpServletRequest request, HttpServletResponse response)
    {
        return "Let's Go";
    }

    /**
     * 测试Content-Type
     *
     * @Author SeawayLee
     * @Date 2017/03/29 22:24
     */
    @ResponseBody
    @RequestMapping(value = "/testContentType", consumes = "text/html;charset=utf-8", produces = "application/json;charset=utf-8", params = {"patientId"}, headers = {"ABC"})
    public Object testContentType(HttpServletRequest request, HttpServletResponse response, String patientId)
    {
        return patientId + "\n" + request.getContentType();
    }

    /**
     * 这种参数接收方式，实质上是UrlParam
     * UrlParam一定会导致中文乱码，除非客户端手动将中文URLEncode
     *
     * @Author SeawayLee
     * @Date 2017/03/29 22:49
     */
    @ResponseBody
    @RequestMapping(value = "/getJson1", method = RequestMethod.POST, produces = "application/json;charset=utf-8")
    public Object getJson1(HttpServletRequest request, HttpServletResponse response, String patientId, String patientName) throws UnsupportedEncodingException
    {
        System.out.println(patientName);
        return new Patient(patientId, patientName);
    }

    /**
     * @ReuqestBody 指明被注解的参数是请求的body，可以是一个json结构，只要双方字符集统一，不会出现中文乱码
     * @Author SeawayLee
     * @Date 2017/03/29 23:14
     */
    @ResponseBody
    @RequestMapping(value = "/getJson2", method = RequestMethod.POST, produces = "application/json")
    public Object getJson2(HttpServletRequest request, @RequestBody JSONObject patientJson) throws UnsupportedEncodingException
    {
        System.out.println(patientJson);
        return patientJson;
    }

    /**
     * 和上面一样
     * @Author SeawayLee
     * @Date 2017/03/29 23:16
     */
    @ResponseBody
    @RequestMapping(value = "/getJson3", method = RequestMethod.POST, produces = "application/json")
    public Object getJson3(HttpServletRequest request, @RequestBody Map<String, String> params) throws UnsupportedEncodingException
    {
        System.out.println(params);
        return params;
    }


    /**
     * 如果请求参数中既有json，又有独立的参数，客户端需要将数据以表单的形式提交
     * @Author SeawayLee
     * @Date 2017/03/29 23:36
     */
    @ResponseBody
    @RequestMapping(value = "/getJson4", method = RequestMethod.POST, produces = "application/json;charset=utf-8")
    public Object getJson4(HttpServletRequest request, @RequestParam(required = true) String patientJson,@RequestParam(required = false) String city) throws UnsupportedEncodingException
    {
        System.out.println(patientJson);
        System.out.println(city);
        return patientJson;
    }

    /**
     * 同上
     * @Author SeawayLee
     * @Date 2017/03/29 23:40
     */
    @ResponseBody
    @RequestMapping(value = "/getJson5", method = RequestMethod.POST, produces = "application/json;charset=utf-8")
    public Object getJson5(HttpServletRequest request) throws UnsupportedEncodingException
    {
        String patientJson = request.getParameter("patientJson");
        String city = request.getParameter("city");
        System.out.println(patientJson);
        System.out.println(city);
        return patientJson;
    }
    

    class Patient implements Serializable
    {
        private String patientId;

        private String patientName;

        public Patient(String patientId, String patientName) {
            this.patientId = patientId;
            this.patientName = patientName;
        }

        public String getPatientId() {
            return patientId;
        }

        public void setPatientId(String patientId) {
            this.patientId = patientId;
        }

        public String getPatientName() {
            return patientName;
        }

        public void setPatientName(String patientName) {
            this.patientName = patientName;
        }
    }
}

```



Content-Type测试结果

| RequestMapping配置 | 错误代码 | 原因分析 |
|---|---|---|
|consumes = "application/json;charset=utf-8"| 415 Unsupported Media Type| 客户端没有设置对应的媒体类型 |
|params = {"patientId"}| 400 Bad Request| 客户端没有穿啊书必要的参数 |
|headers = {"ABC"}| 404 Not Found| 客户端没有传输必要的headers |



