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
> 观察者(Observer) 例如武警（ArmedPolice）
> 被观察者（Observable）例如罪犯（Robber）
为了逻辑的解耦，我们把具体对象抽象出来，观察者用接口Police，被观察者用AbstractCrime来表示

```java
public interface Police {

    void update();

}
```

```java

public class ArmedPolice implements Police {

    @Override
    public void update() {
        System.out.print("我是武警（观察者），不许伤害人质，否则我将击毙你\n");
    }

    public static void main(String[] args) {

        Police police = new ArmedPolice();
        AbstractCrime robber2 = new Robber(police);

//        robber1.attack();
        robber2.attack();

    }
}

```

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

```java

public class ArmedPolice implements Police {

    @Override
    public void update() {
        System.out.print("我是武警（观察者），不许伤害人质，否则我将击毙你\n");
    }

    public static void main(String[] args) {

        Police police = new ArmedPolice();
        AbstractCrime robber2 = new Robber(police);

//        robber1.attack();
        robber2.attack();

    }
}

```


### 单例使用注意


### 总结


