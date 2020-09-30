---
title: 重学设计模式-观察者模式
date: 2020-02-24 22:00:17
categories:
- 设计模式
tags:
---

## 概述
### 定义
定义了一组对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。

<!--more-->

### UML图
<img src="/img/firstHead/2.jpg" width="90%" style="text-align:center">

### 代码实现
Subject抽象类
```java
public abstract class Subject {

    /**
     * 用来保存注册的观察者对象
     */
    private List<Observer> list = new ArrayList<>();

    /**
     * 注册观察者对象
     * @param observer 观察者对象
     */
    public void attach(Observer observer) {
        list.add(observer);
        System.out.println("Attached an observer");
    }

    /**
     * 移除观察者对象
     * @param observer 观察者对象
     */
    public void detach(Observer observer) {
        list.remove(observer);
    }

    /**
     * 通知所有注册的观察者对象
     */
    public void notifyObservers(String newState) {
        for (Observer observer : list) {
            observer.update(newState);
        }
    }
}


ConcreteSubject类
```java
public class ConcreteSubject extends Subject {

    private String state;

    public String getState() {
        return state;
    }

    public void setState(String newState) {
        state = newState;
        this.notifyObservers(state);
    }

}

```

Observere接口
```java
public interface Observer {

    /**
     * 更新接口
     * @param state 更新的状态
     */
    void update(String state);

}
```

ConcreteStrategy类
```java
public class ConcreteObserver implements Observer {

    // 观察者状态
    private String observerState;

    @Override
    public void update(String state) {
        /**
         * 更新观察者的状态，使其与目标的状态保持一致
         */
        observerState = state;
    }
}
```

test
```java
public class Client {
    public static void main(String[] args) {
        // 创建主题对象
        ConcreteSubject subject = new ConcreteSubject();
        // 创建观察者对象
        Observer observer = new ConcreteObserver();
        // 将观察者对象注册到主题对象上
        subject.attach(observer);
        // 改变主题对象的状态
        subject.change("new state");
    }
}
```
## 分析

观察者模式描述了如何建立对象与对象之间的依赖关系，以及如何构造满足这种需求的系统。

这一模式中的关键对象是观察目标和观察者，一个目标可以有任意数目的与之相依赖的观察者，一旦目标的状态发生改变，所有的观察者都将得到通知。

### 松耦合的威力
- 为了交互对象之间的松耦合设计而努力

当两个对象之间松耦合，他们依然可以交互，但是不太清楚彼此之间的细节。

观察者模式提供了一种对象设计，让主题与观察者之间松耦合。

关于观察者的一切，主题只知道观察者实现了某个接口，主题不需要知道观察者具体类是什么，做了什么细节操作，任何时候我们都可以增加新的观察者，改变主题或者观察者的一方，并不会影响另一方，因为二者是松耦合，只要他们之间的接口仍被遵守，我们就可以自由地改变他们。

松耦合的设计之所以能够让我们建立有弹性的OO系统，能够应对变化，是因为对象之间的互相依赖程度降到了最低。

### 适用环境
在以下情况下可以使用观察者模式：

- 一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用。
- 一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度。
- 一个对象必须通知其他对象，而并不知道这些对象是谁。
- 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象...，可以使用观察者模式创建一种链式触发机制。

### 优缺点
- 优点：
    - 观察者模式可以实现表示层和数据逻辑层的分离，并定义了稳定的消息更新传递机制，抽象了更新接口，使得可以有各种各样不同的表示层作为具体观察者角色。
    - 观察者模式在观察目标和观察者之间建立一个抽象的耦合。
    - 观察者模式支持广播通信。
    - 观察者模式符合“开闭原则”的要求。
-   缺点：
    - 如果一个观察目标对象有很多直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。
    - 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。
    - 观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

### 推模型和拉模型
在观察者模式中，又分为推模型和拉模型两种方式。

- 推模型

    主题对象向观察者推送主题的详细信息，不管观察者是否需要，推送的信息通常是主题对象的全部或部分数据。
- 拉模型

    主题对象在通知观察者的时候，只传递少量信息。如果观察者需要更具体的信息，由观察者主动到主题对象中获取，相当于是观察者从主题对象中拉数据。一般这种模型的实现中，会把主题对象自身通过update()方法传递给观察者，这样观察者需要获取数据的时候，就可以通过这个引用来获取了。

下面给出拉模型的实现

```java
public interface Observer {

    /**
     * 更新接口
     * @param subject 传入主题对象，方便获取相应的主题对象的状态
     */
    void update(Subject subject);

}

public class ConcreteObserver implements Observer {

    // 观察者的状态
    private String observerState;

    @Override
    public void update(Subject subject) {
        observerState = ((ConcreteSubject )subject).getState();
    }
}

public abstract class Subject {

    private List<Observer> list = new ArrayList<>();
    /**
     * 注册观察者对象
     * @param observer 观察者对象
     */
    public void attach(Observer observer) {
        list.add(observer);
    }

    /**
     * 删除观察者对象
     * @param observer 观察者对象
     */
    public void detach(Observer observer) {
        list.remove(observer);
    }

    /**
     * 通知所有注册的观察者对象
     */
    public void notifyObservers() {
        for (Observer observer : list) {
            observer.update(this);
        }
    }
}

public class ConcreteSubject extends Subject {

    private String state;

    public String getState() {
        return state;
    }

    public void change(String newState) {
        state = newState;
        // 状态发生改变，通知各个观察者
        this.notifyObservers();
    }

}
```

实现观察者模式的方法不止一种，但是包含Subject和Observer接口类的方法是最常见的。



