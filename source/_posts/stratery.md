---
title: 重学设计模式-策略模式
date: 2020-02-16 23:30:52
categories:
- [设计模式]
tags:
---

## 概述
### 定义
定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

<!--more-->

### UML图
<img src="/img/firstHead/1s.jpg" width="90%" style="text-align:center">


### 代码实现
Context类
```java
public class Context {
    private Strategy strategy;
    /**
     * 构造函数，传入一个具体策略对象
     * @param strategy 具体策略对象
     */
    public Context(Strategy strategy){
        this.strategy = strategy;
    }

    /**
    *替换策略方法
    */
    public setStrategy(Strategy strategy){
        this.strategy = strategy;
    }

    /**
     * 策略方法
     */
    public void contextInterface(){
        strategy.algorithmInterface();
    }
}
```

Strategy接口类
```java
public interface Strategy {
    /**
     * 策略方法
     */
    public void algorithmInterface();
}
```

Strategy接口实现类
```java
public class ConcreteStrategyA implements Strategy {

    @Override
    public void algorithmInterface() {
        //todo
    }

}
```

## 分析

在软件开发中的唯一不变真理就是-`CHANGE`，不管当初软件设计的多好，过一段时间之后总是需要成长与变化的

###  场景
软件开发中容易遇到这样的场景，对于相同的功能有多种算法或者策略来实现，我们可以根据环境或者条件选择不同的算法来完成功能。该场景的一种实现方案是将这些算法写到一个类中，在该类中提供多个方法，每一个方法对应一个具体的算法；当然也可以将这些算法封装在一个统一的方法中，通过if-else或者switch-case等条件判断语句来进行选择，这些统称为硬编码方式，而该类方式的问题就是不易维护和扩展，每当新增一种策略的时候我们需要修改封装类（函数），更换策略的时候我们需要修改调用端的代码，而最大的问题在于这个硬编码的封装类中保存了大量的实现代码，这样不易于维护；想象一下，一个2000行的实现类和一个200行的实现类，你更愿意看哪个

列举出两个实际的场景
- 在前端页面提供了用户自定义布局的功能，页面列表的展示有多种方式：单列、双列、不规则等方式，展示可以根据用户的偏好设置来决定。
- 在后端开发中文件在服务器的存储类型，有多种存储方式比如NAS、OSS或者是AWS提供的对象存储方式，根据业务场景的不同服务器端所选择的存储类型也不同。

### 核心问题
该类场景的核心问题在于将对象与行为分离，而策略模式就是将对象本身和运算规则区分开来，因为这个设计模式本身的核心思想就是面向对象编程的多形性。

我们可以在策略模式的UML类图看到这个关系，Context类就是对象类，而这个对象的一个行为被定义为Stratety接口，这个行为的实现并不在Context类中，而是被一个单独的类实现，Context类所做的是持有这行为。


### 策略模式中的设计原则
- 找出应用中可能需要的变化之处，把他们独立出来，不要和那些不需要变化的代码混在一起

- 针对接口编程，而不是针对实现编程

- 多用组合，少用继承

第一条就不再说明，来看第二条，`针对接口编程，而不是针对实现编程`这个设计原则在策略模式中的体现，我的理解是两点：
1. Context依旧拥有行为可以执行行为，但这个行为的实现并不是在Context中，而是一个其他的专门类，这个专门类实现了行为的接口也就达成了一种约定规范，Context类可以放心大胆地调用它；这样的做法迥异于以往，以前的做法是行为的实现来自于Context超类的具体实现，或者是Context继承自某个接口并实现该接口。这两种方式均依赖于`"实现"`，Context被实现绑的死死的，无法做到在不修改代码的情况下更改行为。

2. 针对接口编程，真正意思是针对超类型（supertype）编程，可以在UML图中看到Context持有的是strategy接口而不是具体的实现类，也就是变量的声明应该是超类型，通常是一个抽象类或者接口，如此做只要是实现这个超类型的类所产生的对象均可以指定给这个变量，做Java后端开发的小伙伴最常见的可能是后端单体应用程序的垂直三层结构，Controller调用Service层时，Controller持有的变量类型是Service接口。

`多用组合，少用继承`

在策略模式的定义中可以看到「让它们之间可以互相替换」，就是在程序运行过程中我们可以调用Context的`setStrategy()`方法动态的更换行为，允许这样的原因是行为是组合而来的，而不是继承而来的，使用组合的系统具有很大的弹性，不仅可以将算法族封装为算法类，更可以在运行时动态的改变行为（有点想起反射），因为“有一个”往往比“是一个”更好。

### 优缺点
- 优点
    - Context和具体策略是松耦合关系。Context只需要指导它要使用某一个实现Strategy接口类的实例，而不需要知道具体是哪一个类。 
    - 策略模式满足“开-闭原则”。当增加新的具体策略时，不需要修改Context类的代码，Context就可以引用新的具体策略的实例。
- 缺点
    - 策略模式只适用于客户端知道所有的算法或行为的情况。
    - 策略模式把每个具体的策略实现都单独封装成为类，所以如果策略很多的话，那么对象的数目就会比较爆炸，管理起来嗯～有那味儿了。

## 使用场景分析
在JDK中比较经典让人一下就能想起来的场景是JUC的线程池，这里就趁热分析一下，笔者已经好久没写Java了有些不太熟悉了😂

### 概念以及基本使用
线程池作用就是限制系统中执行线程的数量，线程池提供了对于线程的管理，减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，这样可以减少系统资源的浪费。同时还可以根据系统的承受能力调整线程池中工作线线程的数目，防止因为消耗过多的内存，将服务器搞宕机。

线程池的基本使用构造函数
ThreadPoolExecutor的
```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
                          }
```
- corePoolSize是"核心池大小"，maximumPoolSize是"最大池大小"。它们的作用是调整线程池中实际运行的线程的数量，例如当新任务提交给线程池时
    - 如果此时线程池中运行的线程数量< corePoolSize，则创建新线程来处理请求。
    - 如果此时线程池中运行的线程数量> corePoolSize，但是却< maximumPoolSize；则仅当阻塞队列满时才创建新线程。
    - 设置的 corePoolSize 和 maximumPoolSize 相同，则创建了固定大小的线程池。
- keepAliveTime 线程存活时间，超过keepAliveTime时间之后，空闲的线程会被终止。
- workQueue是BlockingQueue类型，即它是一个阻塞队列。当线程池中的线程数超过它的容量的时候，线程会进入阻塞队列进行阻塞等待。通过workQueue，线程池实现了阻塞功能。
- handler是RejectedExecutionHandler类型。它代表线程池拒绝策略，也就是说当某任务添加到线程池中，而线程池拒绝该任务时，线程池会通过handler进行相应的处理。

`RejectedExecutionHandler`就是使用了策略模式，来看一下线程池拒绝策略的使用
```java
import java.lang.reflect.Field;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.ThreadPoolExecutor.DiscardOldestPolicy;

public class DiscardOldestPolicyDemo {

    private static final int THREADS_SIZE = 1;
    private static final int CAPACITY = 1;

    public static void main(String[] args) throws Exception {

        // 创建线程池。线程池的"最大池大小"和"核心池大小"都为1(THREADS_SIZE)，"线程池"的阻塞队列容量为1(CAPACITY)。
        ThreadPoolExecutor pool = new ThreadPoolExecutor(THREADS_SIZE, THREADS_SIZE, 0, TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(CAPACITY));
        // 设置线程池的拒绝策略为"DiscardOldestPolicy"
        pool.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardOldestPolicy());

        // 新建10个任务，并将它们添加到线程池中。
        for (int i = 0; i < 10; i++) {
            Runnable myrun = new MyRunnable("task-"+i);
            pool.execute(myrun);
        }
        // 关闭线程池
        pool.shutdown();
    }
}

class MyRunnable implements Runnable {
    private String name;
    public MyRunnable(String name) {
        this.name = name;
    }
    @Override
    public void run() {
        try {
            System.out.println(this.name + " is running.");
            Thread.sleep(200);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
线程池的拒绝策略，是指当任务添加到线程池中被拒绝而采取的处理措施。
当任务添加到线程池中之所以被拒绝，可能是由于：
1. 线程池异常关闭。
2. 任务数量超过线程池的最大限制。

线程池共包括4种拒绝策略，它们分别是：AbortPolicy, CallerRunsPolicy, DiscardOldestPolicy和DiscardPolicy。

- AbortPolicy         -- 当任务添加到线程池中被拒绝时，它将抛出     RejectedExecutionException异常。
- CallerRunsPolicy    -- 当任务添加到线程池中被拒绝时，会在线程池当前正在运行的Thread线程池中处理被拒绝的任务。
- DiscardOldestPolicy -- 当任务添加到线程池中被拒绝时，线程池会放弃等待队列中最旧的未处理任务，然后将被拒绝的任务添加到等待队列中。
- DiscardPolicy       -- 当任务添加到线程池中被拒绝时，线程池将丢弃被拒绝的任务。


这些类实现了RejectedExecutionHandler接口，而ThreadPoolExecutor中持有这个策略，而且既然是策略模式那么我们自己可以实现自定义侧拒绝策略。

### 复习一下别的
线程池的基本使用
```Java
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;

public class ThreadPoolDemo1 {

    public static void main(String[] args) {
        // 创建一个可重用固定线程数的线程池
        ExecutorService pool = Executors.newFixedThreadPool(2);
        // 创建实现了Runnable接口对象
        Thread ta = new MyThread();
        Thread tb = new MyThread();
        Thread tc = new MyThread();
        Thread td = new MyThread();
        Thread te = new MyThread();
        // 将线程放入池中进行执行
        pool.execute(ta);
        pool.execute(tb);
        pool.execute(tc);
        pool.execute(td);
        pool.execute(te);
        // 关闭线程池
        pool.shutdown();
    }
}

class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+ " is running.");
    }
}
```
主线程中创建了线程池pool，线程池的容量是2。即线程池中最多能同时运行2个线程。紧接着将ta,tb,tc,td,te这3个线程添加到线程池中运行。最后通过shutdown()关闭线程池。

这里并不能看出来哪里是用了策略模式，我们还需稍微深究一下代码

简单介绍一下JUC线程池这个大家族的关系:

- 最顶层的接口是`Executor`,它只定义了一个函数接口，用来执行任务的
    ```java
    void execute(Runnable command)
    ```
- `ExecutorService`这个接口继承了`Executor`接口，`ExecutorService`的作用是为`Executor`服务，提供了`invokeAll`等方法

- `AbstractExecutorService`是一个抽象类，它实现了`ExecutorService`接口,`AbstractExecutorService`存在的目的是为`ExecutorService`中的函数接口提供了默认实现。

- `ThreadPoolExecutor`就是大名鼎鼎的"线程池"。它继承于`AbstractExecutorService`抽象类。

- `Executors`是个静态工厂类。它通过静态工厂方法返回`ExecutorService`的类的对象


实例中创建线程池是通过` ExecutorService pool = Executors.newFixedThreadPool(2);`

这里是通过静态工厂方法的方式创建了一个固定大小的线程池，限制大小为2，Executors中还提供了其他的方法可以创建各种特点的线程池，这里之演示固定大小的线程池，我们看一下`newFixedThreadPool`的源代码
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```
它的实现是调用`ThreadPoolExecutor`的构造函数


