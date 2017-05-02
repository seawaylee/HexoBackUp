---
title: Chapter03 高级装备 
date: 2017-03-09 23:34:08
tags: [JavaWeb,Spring]
---


## 3.1 环境与Profile
一般情况下，开发环境所需要的spring配置文件，例如dataSource数据源肯定与生产环境是不一样的，开发环境到生产环境或测试换几个时，我们都需要手动的去替换大量配置文件，非常麻烦，Spring的Profile为我们解决了这个问题，通过激活不同的profile来控制创建哪些bean。

<!--more-->

**下面举一个最简单的例子**
创建两个不同的类，假设他们是不同的配置文件


```java
public class Animal
{
    public Animal()
    {
        System.out.println("动物对象被创建了...");
    }
}

public class Student
{
    public Student()
    {
        System.out.println("学生对象被实例化了");
    }
}

```

Spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">

    <beans profile="dev">
        <bean id="object" class="chapter03.Student"/>
    </beans>

    <beans profile="prod">
        <bean id="object" class="chapter03.Animal"/>
    </beans>

</beans>

```

指定profile进行测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring/c3-applicationContext.xml")
@ActiveProfiles("prod")
public class Main
{
    @Test
    public void profileTest()
    {
        System.out.println("测试");
    }
}

```
输出
`
动物对象被创建了...
测试
`


修改

```java
@ActiveProfiles("dev")
```
输出
`
学生对象被实例化了
测试
`
在web开发中，我们可以使用以下方式切换和设置profile
需要依赖两个独立的属性：spring.profiles.active    spring.profiles.default
如果设置了active，指定的profile会被激活。
如果没有设置active，指定的default会被激活。
如果都没有设置，只会创建没有在profile范围内的bean。
有多钟方式来设置这两个属性：

1. 作为DispatcherServlet的初始化参数；
2. 作为Web应用的上下文参数；
3. 作为JNDI条目；
4. 作为环境变量；
5. 作为JVM的系统属性；
6. 在集成测试类上，使用@ActiveProfiles注解设置；


## 3.2 条件化的bean
如果有这种需求：只有某个特定的环境变量设置之后，才会创建某个bean。
那么我们就需要这个功能。一般情况下不会用到，因此不做详细描述。

```java
public class MagicBeanFactory
{
    @Bean
    @Conditional(MagicExistsCondition.class)
    public MagicBean magicBean()
    {
        return new MagicBean();
    }
}

class MagicBean
{
    public MagicBean()
    {
        System.out.println("MagicBean 被初始化了...");
    }
}

class MagicExistsCondition implements Condition
{
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata)
    {
        Environment env = conditionContext.getEnvironment();
        return env.containsProperty("magic");
    }
}
```
## 3.3 处理自动装配的歧义性
假设有一个Animal接口，Dog和Cat都是Animal接口的实现类，当我们使用@Autowired注入Animal时，Spring会因为不知道注入Dog还是Cat而报错。

解决方案：



```java
public interface Animal
{
}
```

```java
@Component
//@Qualifier("cat")
@Primary
public class Cat implements Animal
{
}
```


```java
@Component
//@Qualifier("dog")
//@Primary
public class Dog implements Animal
{
}
```


## 3.4 运行时值注入

```java
@Component
@PropertySource("classpath:db.properties")
public class MyProperties
{
    @Autowired
    Environment env;


    public void show()
    {
        System.out.println(env.getProperty("username") + "," + env.getProperty("password"));
    }
}

```


```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring/c3-applicationContext.xml")
//@ActivePcrofiles("dev")
public class Main
{
    @Autowired
    MagicBeanFactory factory;

    @Autowired
    @Qualifier("dog")
    Animal animal;

    @Autowired
    private MyProperties properties;

    @Test
    public void propertyTest()
    {
        properties.show();
    }
}

```


## 3.5 小结
本章是一些装配Bean的高级特性。包括使用profile初始化不同的beans；在spring4中使用@Conditional更加灵活细化的按条件初始化bean；使用@Qualifier解决@Autowired的歧义性；使用@PropertySource等一些列注解读取配置文件；其实还有SpEl表达式，用来动态的初始化bean，本文没有做讲解，在以后学习Spring-Security的时候会详细说明。


