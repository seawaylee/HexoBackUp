---
title: Chapter04 面向切面的Spring 
date: 2017-03-09 23:35:55
tags: [JavaWeb,Spring]
---

## 4.1AOP所要解决的问题
将横切关注点（包裹在业务代码外层）与业务逻辑相分离，AOP实现将横切关注点与他们所影响的对象之间的解耦。除此之外，AOP还还会在声明式事务、安全和缓存进行应用。
AOP用于重用通用功能，传统方式最常见的是继承和委托。继承会造成对象体系非常脆弱，委托会对委托对象进行复杂的对象，AOP提供了另一种可选方案。我们通过声明式的方式定义这个功能要以何种方式在何处应用，而无需求改受影响的类。横切关注点可以被模块化为特殊的类，这些类被称为切面（aspect）。
优点：
    
- 1.每个关注点都集中于一个地方，而不是分散到代码中
- 2.服务模块更简洁，只包含核心代码，关注点的代码都被转移到切面中
    
<!--more-->

## 4.2 相关术语

- 通知（advice）
- 切点（pointcut）
- 连接点（joinpoint）

### 4.2.1 通知
切面的工作被称为通知。

- 前置通知（Before）：在目标方法被调用之前调用通知功能
- 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么
- 返回通知（After-returning）：在目标方法成功执行之后调用通知
- 异常通知（After-throwing）：在目标方法抛出异常后调用通知
- 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为

### 4.2.2 连接点
连接点是可以插入切面的一个点（时间点）。例如方法执行前、方法执行后、抛出异常时...等等。

### 4.2.3 切点
定义切面在何处（位置点）进行通知。

### 4.2.4 切面
切面是通知和切点的结合。

### 4.2.5 引入
引入允许我们向现有的类添加新的方法或属性。

### 4.2.6 织入
织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。
在目标对象的生命周期里有多个点可以进行织入：

- 编译期：切面在目标类编译时被织入。例如AspectJ的织入编译器。
- 类加载期：切面在目标被加载到JVM时被织入。例如AspectJ5.
- 运行期：切面在应用运行的某个时刻被织入。一般情况啊下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面的。（学习下反射和动态代理哦）

## 4.3 切点表达式语言
知道AspectJ是一种切点表达式语言就够啦。用起来超级简单。
以下注释部分是配置是说明如何通过AspectJ表达式来匹配该方法作为切点。

**创建一个表演类，我们要在表演前后做一些动作**

```java
@Component
public class Performance
{
    /**
     * 执行该方法的切点表达是语言 execution(* chapter04.Performace.peforne(...))
     * execution:表示执行该表达式语言
     * * 表示任意返回类型
     * chapter04.Performance.performe 分别是 包名、类名、方法名
     * ...表示使用任意参数
     * @Author NikoBelic
     * @Date 28/12/2016 19:04
     */
    public void perform()
    {
        System.out.println("我表演啦");
        //return;
    }
}
```

**创建观众类作为切面**

```java
@Aspect
public class Audience
{
    // 定义命名的切点
    @Pointcut("execution(* *.perform(..))")
    public void performance()
    {}



    @Before("performance()")
    public void silenceCellPhones()
    {
        System.out.println("观众们请关闭手机铃声...");
    }

    @Before("performance()")
    public void takSeats()
    {
        System.out.println("大家请坐...");
    }

    @AfterReturning("performance()")
    public void applause()
    {
        System.out.println("大家请鼓掌...");
    }

    @AfterThrowing("performance()")
    public void deamanRefund()
    {
        System.out.println("表演失败了,大家鼓励一下...");
    }
}

```



**基于类的Spring配置**


```java
@Configuration
@EnableAspectJAutoProxy
@ComponentScan
public class ConcertConfig
{
    @Bean
    public Audience audience()
    {
        return new Audience();
    }
}

```

**测试方法**


```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = ConcertConfig.class)
public class Main
{
    @Autowired
    Performance performance;

    @Test
    public void aopTest()
    {
        performance.perform();
    }
}

```

输出

`观众们请关闭手机铃声...
大家请坐...
我表演啦
大家请鼓掌...`


使用XML配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.2.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置自动扫包-->
    <context:component-scan base-package="chapter04"/>
    <!--启动AspectJ自动代理-->
    <aop:aspectj-autoproxy/>


</beans>

```

测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
//@ContextConfiguration(classes = ConcertConfig.class)
@ContextConfiguration(locations = "classpath:spring/c4-applicationContext.xml")
public class Main
{
    @Autowired
    Performance performance;

    @Test
    public void aopTest()
    {
        performance.perform();
    }
}

```


**环绕通知**
修改切面类

```java
@Aspect
public class Audience
{
    // 定义命名的切点
    @Pointcut("execution(* *.perform(..))")
    public void performance()
    {}

    @Around(("performance()"))
    public void watchPerformance(ProceedingJoinPoint joinPoint)
    {
        try
        {
            System.out.println("环绕-手机静音");
            System.out.println("环绕-就坐");
            joinPoint.proceed();
            System.out.println("环绕-牛b牛b!再来一个!");

        } catch (Throwable throwable)
        {
            System.out.println("环绕-垃圾!退钱!");
        }
    }


    //@Before("performance()")
    public void silenceCellPhones()
    {
        System.out.println("观众们请关闭手机铃声...");
    }

    //@Before("performance()")
    public void takSeats()
    {
        System.out.println("大家请坐...");
    }

    //@AfterReturning("performance()")
    public void applause()
    {
        System.out.println("大家请鼓掌...");
    }

    //@AfterThrowing("performance()")
    public void deamanRefund()
    {
        System.out.println("表演失败了,大家鼓励一下...");
    }
}

```
其他不变，测试结果
`环绕-手机静音
环绕-就坐
我表演啦
环绕-牛b牛b!再来一个!`

**带参数的通知**

我们通过编写切面来统计CD磁盘中的某个歌曲被播放了几次
播放器类

```java
@Component
public class CDPlayer
{
    public void play(int songNum)
    {
        System.out.println("正在播放CD磁盘中第" + songNum + "首歌曲");
    }
}
```
切面类


```java
@Aspect
public class PlayCounter
{
    private HashMap<Integer,Integer> trackCounts = new HashMap<>();

    @Pointcut("execution(* *.play(int)) " + "&& args(songNum)")
    public void trackPlayed(int songNum){}

    @Before("trackPlayed(songNum)")
    public void countSong(int songNum)
    {
        int currentCount = getPlayCount(songNum);
        trackCounts.put(songNum,currentCount + 1);
        System.out.println("该歌曲播放过" + currentCount + "次\n");
    }

    public int getPlayCount(int songNum)
    {
        return trackCounts.containsKey(songNum)?trackCounts.get(songNum):0;
    }
}
```

配置类

```java
@Configuration
@EnableAspectJAutoProxy/*启动AspectJ自动代理*/
@ComponentScan
public class ConcertConfig
{
    // 声明Audience bean
    @Bean
    public Audience audience()
    {
        return new Audience();
    }

    @Bean
    public PlayCounter playCounter()
    {
        return new PlayCounter();
    }
}

```
测试

```java
@Test
public void aopArgsTest()
{
   for (int i = 0; i < 10; i++)
   {
       cdPlayer.play(new Random().nextInt(3));
   }
}
```
输出

```
该歌曲播放过0次
正在播放CD磁盘中第2首歌曲

该歌曲播放过1次
正在播放CD磁盘中第2首歌曲

该歌曲播放过0次
正在播放CD磁盘中第0首歌曲

该歌曲播放过2次
正在播放CD磁盘中第2首歌曲

该歌曲播放过3次
正在播放CD磁盘中第2首歌曲

该歌曲播放过1次
正在播放CD磁盘中第0首歌曲

该歌曲播放过2次
正在播放CD磁盘中第0首歌曲

该歌曲播放过4次
正在播放CD磁盘中第2首歌曲

该歌曲播放过3次
正在播放CD磁盘中第0首歌曲

该歌曲播放过0次
正在播放CD磁盘中第1首歌曲

```

## 4.4 在XML中声明切面
当你需要声明切面，但又不能为通知类添加注解的时候，那么就必须转向XML配置了。


| AOP配置元素 | 用途 |
| --- | --- |
| <aop:advisor> | 定义AOP通知器 |
| <aop:after> | 定义AOP后置通知（不管被通知的方法是否执行成功） |
| <aop:after-returnning> | 定义AOP返回通知 |
| <aop:after-throwing> | 定义AOP异常通知 |
| <aop:around> | 定义AOP环绕通知 |
| <aop:aspect> | 定义一个切面 |
| <aop:aspectj-autoproxy> | 启动@AspectJ注解驱动的切面 |
| <aop:before> | 定义一个AOP前置通知 |
| <aop:config> | 顶层的AOP配置元素。大多数的<aop:*>元素必须包含在<aop:config>元素内 |
| <aop:declare-parents> | 以透明的方式呗通知的对象引入额外的接口 |
| <aop:pointcut> | 定义一个切点 |


将切面的注解去掉

```java
@Component
public class Audience
{
    // 定义命名的切点
    //@Pointcut("execution(* *.perform(..))")
    //public void performance()
    //{}

    //@Around(("performance()"))
    public void watchPerformance(ProceedingJoinPoint joinPoint)
    {
        try
        {
            System.out.println("环绕-手机静音");
            System.out.println("环绕-就坐");
            joinPoint.proceed();
            System.out.println("环绕-牛b牛b!再来一个!");

        } catch (Throwable throwable)
        {
            System.out.println("环绕-垃圾!退钱!");
        }
    }


    //@Before("performance()")
    public void silenceCellPhones()
    {
        System.out.println("观众们请关闭手机铃声...");
    }

    //@Before("performance()")
    public void takSeats()
    {
        System.out.println("大家请坐...");
    }

    //@AfterReturning("performance()")
    public void applause()
    {
        System.out.println("大家请鼓掌...");
    }

    //@AfterThrowing("performance()")
    public void deamanRefund()
    {
        System.out.println("表演失败了,大家鼓励一下...");
    }
}

```


配置xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.2.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--配置自动扫包-->
    <context:component-scan base-package="chapter04"/>
    <!--启动AspectJ自动代理-->
    <aop:aspectj-autoproxy/>

    <aop:config>
        <!--切面-->
        <aop:aspect ref="audience">
            <!--切点-->
            <aop:pointcut id="dosth" expression="execution(* *.perform(..))"/>
            <!--前置通知-->
            <aop:before method="silenceCellPhones" pointcut-ref="dosth"/>
            <aop:before method="takSeats" pointcut-ref="dosth"/>
            <!--后置通知-->
            <aop:after-returning method="applause" pointcut-ref="dosth"/>
            <!--异常通知-->
            <aop:after-throwing method="deamanRefund" pointcut-ref="dosth"/>
        </aop:aspect>


    </aop:config>

</beans>

```

注意 修改以下Main测试类的配置读取方式，将Java配置类改为配置文件配置。


```java
@RunWith(SpringJUnit4ClassRunner.class)
//@ContextConfiguration(classes = ConcertConfig.class)
@ContextConfiguration(locations = "classpath:spring/c4-applicationContext.xml")
public class Main
{...}
```


## 5 小结
本章讲解了通过切面类和配置文件配置实现Spring的AOP功能，使模块之间进行解耦，有效减少了代码冗余，让我们只关注类自身的功能。
通过这几章我们学习了Spring和核心功能，DI和AOP。但是我们现在只停留在了会用的阶段，要深入其原理还需要看看源码，下一章开始学习SpringMVC。开始构建真正的Web应用。

