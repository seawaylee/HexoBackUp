---
title: SpringInAction读书笔记(二) 装备Bean 
date: 2017-03-09 22:17
tags: [JavaWeb,Spring]
---



## 2.1 Sprig装配Bean的方式

Spring装配Bean的方案有三种，任选其一。

- 在XML中进行显示配置
- 在Java中进行显示配置
- 隐式的bean发现机制和自动装配


**自动化装配Bean**
Spring从两个角度来实现自动化装备：

- 组件扫描component scanning：Spring自动发现应用上下文所创建的bean
- 自动装配autowiring：Spring自动满足bean之间的依赖

<!--more-->

## 2.2 Spring自动扫描Bean
我们不使用上一章那种XML的方式将bean配置，而是采用基于Java的配置方法。

创建一个接口

```
public interface CompactDisc
{
    void play();
}
```
创建实现类，@Component注解将该类标识为由Spring管理的组件


```
@Component
public class Jay implements CompactDisc
{
    private String title = "七里香";
    private String artist = "周杰伦";

    public void play()
    {
        System.out.println("播放器正在播放 " + title + "-" + artist);
    }
}
```

创建Spring的Java配置类，其中ComponentScan默认扫描当前包下的所有被标示为@Component（其实还包括@Server、@Repository等等），ComponentScan可以配置扫描路径，如果不写就只扫描当前包。


```
@Configuration
@ComponentScan
public class CDPlayerConfig
{
}

```

如果不采用上面的Java配置方法，我们可以使用XML配置要扫描的路径。

```
<!--自动扫描-->
<context:component-scan base-package="chapter02"/>

```

然后我们就可以使用测试一下是否注入成功了
有两种读取配置的方式可以选择，被注释的部分是读取Java配置类，后者是读取xml配置文件。

```
@RunWith(SpringJUnit4ClassRunner.class)
//@ContextConfiguration(classes = CDPlayerConfig.class)
@ContextConfiguration(locations = "classpath:spring/applicationContext.xml")
public class CDPlyaerTest
{
    @Autowired
    private CompactDisc cd;

    @Test
    public void componentTest()
    {
        cd.play();
    }
}
```


输出
`播放器正在播放 七里香-周杰伦`



## 2.3 Spring自动装配Bean
@Autowired自动装配注解可以用在类的属性、setter方法、其他方法上。
不过一般来说我们都是用在属性上，如上面的装配CD的例子。
当然，如果不愿意使用Spring特有的@Autowired来注解，也可以使用@Inject（Java依赖注入规范），来注解。在我们声明类为组件时使用@Component来注解，也可以使用Java提供的@Named来注解。推荐使用Spring特有的注解。

