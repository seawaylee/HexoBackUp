---
title:  Timer与ScheduledThreadPoolExecutor
date: 2017-03-10 00:25:53
tags: [多线程]
---

```java


import java.text.SimpleDateFormat;    
import java.util.Date;    
import java.util.concurrent.ScheduledThreadPoolExecutor;    
import java.util.concurrent.TimeUnit;    
    
public class TestScheduledThreadPoolExecutor {    
    
    public static void main(String[] args) {    
            
        String time = new SimpleDateFormat("HH:mm:ss").format(new Date());    
        System.out.println("Start time : " + time);    
            
        ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(5);  //创建5个执行线程    
            
        Runnable runnable = new Runnable() {    
                
            @Override    
            public void run() {    
                // TODO Auto-generated method stub    
                String time = new SimpleDateFormat("HH:mm:ss").format(new Date());    
                System.out.println("Now Time : "  + time);    
            }    
        };    
            
        executor.scheduleWithFixedDelay(runnable, 2, 3, TimeUnit.SECONDS);    
            
    }    
        
}    
```
<!--more-->

在实际应用中，有时候我们需要创建一些个延迟的、并具有周期性的任务，比如，我们希望当我们的程序启动后每隔1小时就去做一次日志记录。在JDK中提供了两种方法去创建延迟周期性任务。

Timer
Timer是java.util包下的一个类，在JDK1.3的时候被引入，Timer只是充当了一个执行者的角色，真正的任务逻辑是通过一个叫做TimerTask的抽象类完成的，TimerTask也是java.util包下面的类，它是一个实现了Runnable接口的抽象类，包含一个抽象方法run( )方法，需要我们自己去提供具体的业务实现。
Timer类对象是通过其schedule方法执行TimerTask对象中定义的业务逻辑，并且schedule方法拥有多个重载方法提供不同的延迟与周期性服务。

下面是利用Timer去创建的一个延时周期性任务

```java


import java.text.SimpleDateFormat;    
import java.util.Date;    
import java.util.Timer;    
import java.util.TimerTask;    
    
public class TestTimer {    
    
    public static void main(String[] args) {    
            
        String time = new SimpleDateFormat("HH:mm:ss").format(new Date());    
        System.out.println("Start time : " + time);    
            
        Timer timer = new Timer();    
        TimerTask task = new TimerTask() {    
                
            @Override    
            public void run() {    
                // TODO Auto-generated method stub    
                String time = new SimpleDateFormat("HH:mm:ss").format(new Date());    
                System.out.println("Now Time : "  + time);    
            }    
        }; //end task    
            
        timer.schedule(task, 2000, 3000);    
            
    }    
        
}   
```


程序的输出：
Start time : 21:36:08  
Now Time : 21:36:10  
Now Time : 21:36:13  
Now Time : 21:36:16  
Now Time : 21:36:19  

ScheduledThreadPoolExecutor
在JDK1.5的时候在java.util.concurrent并发包下引入了ScheduledThreadPoolExecutor类，引入它的原因是因为Timer类创建的延迟周期性任务存在一些缺陷，ScheduledThreadPoolExecutor继承了ThreadPoolExecutor，并且实现了ScheduledExecutorService接口，ScheduledThreadPoolExecutor也是通过schedule方法执行Runnable任务的。
我们用ScheduledThreadPoolExecutor来实现和上述Timer一样的功能

程序的输出：
Start time : 22:12:25  
Now Time : 22:12:27  
Now Time : 22:12:30  
Now Time : 22:12:33  
Now Time : 22:12:36  

这样看来Timer和ScheduledThreadPoolExecutor好像没有声明差别，但是ScheduledThreadPoolExecutor的引入正是由于Timer类存在的一些不足，并且在JDK1.5或更高版本中，几乎没有利用继续使用Timer类，下面说明Timer存在的一些缺点。

单线程
Timer类是通过单线程来执行所有的TimerTask任务的，如果一个任务的执行过程非常耗时，将会导致其他任务的时效性出现问题。而ScheduledThreadPoolExecutor是基于线程池的多线程执行任务，不会存在这样的问题。
这里我们通过让Timer来执行两个TimerTask任务来说明，其中一个TimerTask的执行过程是耗时的，加入需要2秒。

```java


import java.text.SimpleDateFormat;    
import java.util.Date;    
import java.util.Timer;    
import java.util.TimerTask;    
    
public class SingleThreadTimer {    
    
    public static void main(String[] args) {    
            
        String time = new SimpleDateFormat("HH:mm:ss").format(new Date());    
        System.out.println("Start time : " + time);    
            
        Timer timer = new Timer();    
            
        TimerTask task1 = new TimerTask() {    
                
            @Override    
            public void run() {    
                // TODO Auto-generated method stub    
                String time = new SimpleDateFormat("HH:mm:ss").format(new Date());    
                System.out.println("Task1 time : " + time);    
            }    
        };    
            
        TimerTask task2 = new TimerTask() {    
                
            @Override    
            public void run() {    
                // TODO Auto-generated method stub    
                try {    
                    Thread.sleep(2000);    
                } catch (InterruptedException e) {    
                    // TODO Auto-generated catch block    
                    e.printStackTrace();    
                }    
                String time = new SimpleDateFormat("HH:mm:ss").format(new Date());    
                System.out.println("task2 time : " + time);    
            }    
        };    
            
        timer.schedule(task1, 2000, 1000);    
        timer.schedule(task2, 2000, 3000);    
    }    
        
}    

```

这里定义了两个任务，任务1，程序启动2秒后每隔1秒运行一次，任务2，程序启动2秒后，每隔3秒运行1次，然后让Timer同时运行这两个任务
程序的输出如下：
Start time : 22:22:37  
Task1 time : 22:22:39  
task2 time : 22:22:41  
Task1 time : 22:22:41  
Task1 time : 22:22:42  
task2 time : 22:22:44  
Task1 time : 22:22:44  
Task1 time : 22:22:45  
task2 time : 22:22:47  
Task1 time : 22:22:47  
Task1 time : 22:22:48  

可以分析，无论是任务1还是任务2都没有按照我们设定的预期进行运行，造成这个现象的原因就是Timer类是单线程的。

Timer线程不捕获异常
Timer类中是不捕获异常的，假如一个TimerTask中抛出未检查异常（P.S：java中异常分为两类:checked exception(检查异常)和unchecked exception(未检查异常),对于未检查异常也叫RuntimeException(运行时异常). ），Timer类将不会处理这个异常而产生无法预料的错误。这样一个任务抛出异常将会导致整个Timer中的任务都被取消，此时已安排但未执行的TimerTask也永远不会执行了，新的任务也不能被调度（所谓的“线程泄漏”现象）。
下面就已常见的RuntimeException，ArrayIndexOutOfBoundsException数组越界异常，来演示这个缺点：

```java



import java.text.SimpleDateFormat;    
import java.util.Date;    
import java.util.Timer;    
import java.util.TimerTask;    
    
public class TestTimerTask {    
    
    public static void main(String[] args) {    
        System.out.println(new SimpleDateFormat("HH:mm:ss").format(new Date()));    
        Timer timer = new Timer();    
        TimerTask task1 = new TimerTask() {    
            
            @Override    
            public void run() {    
                System.out.println("1: " + new SimpleDateFormat("HH:mm:ss").format(new Date()));    
            }    
        };    
            
        TimerTask task2 = new TimerTask() {    
                
            @Override    
            public void run() {    
                int[] arr = {1,2,3,4,5};    
                try {    
                    Thread.sleep(1000);    
                        
                } catch (InterruptedException e) {    
                    e.printStackTrace();    
                }    
                int index = (int)(Math.random()*100);    
                System.out.println(arr[index]);    
                    
                System.out.println("2: " + new SimpleDateFormat("HH:mm:ss").format(new Date()));    
            }    
        };    
            
        timer.schedule(task1, 2000, 3000);    
        timer.schedule(task2, 2000, 1000);    
            
    }    
        
}    
```


程序会在运行过程中抛出数组越界异常，并且整个程序都会被终止，原来完好的任务1也被终止了。

基于绝对时间
Timer类的调度是基于绝对的时间的，而不是相对的时间，因此Timer类对系统时钟的变化是敏感的，举个例子，加入你希望任务1每个10秒执行一次，某个时刻，你将系统时间提前了6秒，那么任务1就会在4秒后执行，而不是10秒后。在ScheduledThreadPoolExecutor，任务的调度是基于相对时间的，原因是它在任务的内部存储了该任务距离下次调度还需要的时间（使用的是基于System#nanoTime实现的相对时间，不会因为系统时间改变而改变，如距离下次执行还有10秒，不会因为将系统时间调前6秒而变成4秒后执行）。

基于以上3个弊端，在JDK1.5或以上版本中，我们几乎没有理由继续使用Timer类，ScheduledThreadPoolExecutor可以很好的去替代Timer类来完成延迟周期性任务。
