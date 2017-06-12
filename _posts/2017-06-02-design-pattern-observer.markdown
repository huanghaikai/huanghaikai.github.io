---
title: "设计模式之观察者模式"
layout: post
date: 2017-06-02 18:34
tag:
- design pattern
- java
category: Design Pattern
blog: true
description: java design pattern observer 设计模式 观察者模式 发布订阅模式
---

### 引言
>在实际开发中，经常会碰到当一个对象发生变化时，其他某些对象也要随着做出相应的操作。这就好比电视剧中经常看到，一个敌对分子挟持了一个人质，一群持枪警察会时刻关注着犯罪分子的动态，一旦罪犯有危害人质的操作，不同人员都会做出相应的操作，已达到控制罪犯的目的。
罪犯是一个被观察的对象，一群警察则是观察者，一旦被观察者（罪犯）发生变化（比如杀害人质），观察者（警察）就会做出相应的操作（比如击杀罪犯）。这种依赖关系称作为观察者模式。


### 观察者模式中的要素

观察者模式，有时又被称为发布（publish ）-订阅（Subscribe）模式、模型-视图（View）模式、源-收听者(Listener)模式或从属者模式。 主要有两大要素
>  观察者(Observer) 例如武警（ArmedPolice）</p>
>  被观察者（Observable）例如罪犯（Robber）

为了逻辑的解耦，我们把具体对象抽象出来，
> 抽象观察者 用接口Police</p>
> 抽象被观察者 用抽象类AbstractCrime来表示


### 抽象观察者
```java
public interface Police {

    void update();

}
```
抽象观察者中定义做出反应的方法此处为`update()`


### 具体观察者

```java
public class ArmedPolice implements Police {

    @Override
    public void update() {
        System.out.print("我是武警（观察者），不许伤害人质，否则我将击毙你\n");
    }
}

```
具体观察者我们是用武警犯`ArmedPolice`表示，实现接口`Police`,在`upate()`方法中实现具体要做出的操作。

### 抽象被观察者

```java
public abstract class AbstractCrime {

    //用于存放所有观察者，以便于维护
    private List<Police> policeList = null;

    public AbstractCrime(){
        policeList = new ArrayList<>();
    }


    abstract void attack();

    public void removePolice(Police police) {
        this.policeList.remove(police);
    }

    public void addPolice(Police police) {
        this.policeList.add(police);
    }

    /**
     * 通知警察做出动作
     */
    public void notifyPolice() {
        for (Police police : policeList) {
            police.update();
        }
    }
}

```
在被观察者中，我们需要维护一个所有观察者的列表，有可能有多个观察者会观察，同时提供增加、删除、通知观察者等方法，此处的`attack()`是罪犯做出攻击行为时，然后通知所有警察做出反应。

### 具体被观察者

```java

public class Robber extends AbstractCrime {

    public Robber(Police police) {
        addPolice(police);
    }

    @Override
    void attack() {
        System.out.print("我是罪犯(被观察者)，我用刀抵住了人质的脖子\n");
        notifyPolice();
    }
}
```
具体被观察者是一个抢劫犯`Robber`，在构造方法中直接传入观察者对象police,当抢劫犯做出attack动作时，及时通知武警（观察者）。

### 测试

```java
public class ObserverTest{

    public static void main(String[] args) {

        Police police = new ArmedPolice();
        AbstractCrime robber = new Robber(police);
        robber.attack();

    }

}

```
执行结果：
```
我是嫌犯，我用刀抵住了人质的脖子
我是武警，不许伤害人质，否则我将击毙你
```

### 单例使用注意
1. 在存在多个观察者时，会通知每个观察者，最好在代码中处理某些观察者时只在具体条件下触发
2. 观察者和被观察者之间不要存在相互的循环调用，这样会导致程序崩溃，这种逻辑缺失要规避


### 总结
Java API中已经存在现成的观察者`Observer`和被观察者`Observable`设计，可以直接拿来使用。
观察者模式在GUI编程中使用的非常广泛，比如在Android中，在Button等View的点击事件触发之后，做出对应的操作.实际使用中，观察者模式用于异步调用非常非常多！很多操作都是耗时的，观察者毕竟不会一直处于等待的状态，只有做完了耗时操作才通知观察者，同时这种设计避免了使用轮询来监控某个条件是否成立！！！这简直就是程序员的福音！表示观察者模式是真的好！！！

