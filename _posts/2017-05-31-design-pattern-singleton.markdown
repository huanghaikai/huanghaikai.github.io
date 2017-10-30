---
title: "设计模式之单例模式"
layout: post
date: 2017-05-31 18:48
tag:
- design pattern
- java
category: Design Pattern
blog: true
description: java design pattern singleton 设计模式 单例模式
---

### 引言
>单例,顾名思义就是唯一一个单独的存在.在实际开发过程中,某些对象会反复创建,为了减少同一个对象new次数,减少系统内存的使用频率，减轻 GC 压力.因此在使用周期内只需要维持一个对象即可.

### 单例模式UML类图

![单例UML]({{ site.url }}/assets/images/design-pattern/singleton.png)

### 单例的特点
1. 私有的构造函数
2. 使用该类的公有方法获取该对象实例
3. 该类在程序运行期间有且只有一个实例


### 最简单的单例

```java
/**
 * 饿汉单例模式
 */
public class HungrySingleton {

    private static final HungrySingleton INSTANCE = new HungrySingleton();
    private HungrySingleton(){
        System.out.print("HungrySingleton created\n");
    }
    public static HungrySingleton getInstance(){
        return INSTANCE;
    }


    public static void doTest(){
        System.out.print("doTest\n");
    }

    public static void main(String[] args) {

        //验证是否为单例
//        HungrySingleton s1 = HungrySingleton.getInstance();
//        HungrySingleton s2 = HungrySingleton.getInstance();
//        System.out.printf(String.valueOf(s1 == s2));

        HungrySingleton.doTest();
    }
}
```

这种写法可以达到单例的目的,保证了多线程下的线程安全问题,但是这种单例的写法存在问题.在执行` HungrySingleton.doTest();`之后输出的结果如下

```
HungrySingleton created
doTest
```
在HungrySingleton被加载时就会创建一个HungrySingleton的实例.无论`getInstance()`是否被调用都会被初始化.这种单例模式也叫**饿汉式单例模式**.显然只是调用doTest()方法的话完全不需要初始化`INSTANCE`.所以我们可以将这种方式进行优化,进而引入懒汉式单例模式.

### 懒汉单例模式

```java
/**
 * 懒汉单例模式
 */
public class LazySingleton {

    private static LazySingleton instance;

    private LazySingleton(){}

    public static synchronized LazySingleton getInstance(){
        if(instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }

}
```
这样的写法只有在需要获取实例时才初始化`instance`,优化了LazySingleton加载的速度,同时加上了`synchronized`关键字使得该写法是线程安全的.但是这样的写法,在多线程下,即使instance已经初始化成功之后,每次访问`getInstance`方法时,只有一个线程能进入该方法,其他线程都在等待,这样就大大影响了程序的性能.所以我们还需要对这样的懒汉式写法进行改进,我们引入**双重检验锁模式**.

### 双重检验锁单例模式
双重检验锁模式（double checked locking pattern），是一种使用同步块加锁的方法。在回去实例对象时会有两次检查`instance == null`，一次是在同步块外，一次是在同步块内。先来看下代码

```java
/**
 * 双重检验锁单例模式
 */
public class DCLSingleton {

    private volatile static DCLSingleton instance = null;//(4)

    private DCLSingleton(){}

    public static DCLSingleton getInstance(){
        if(instance == null){  //语句(1) 同步块外
            synchronized (DCLSingleton.class){
                if(instance == null){ //语句(2) 同步块内,为了防止产生多个instance实例
                    instance = new DCLSingleton();//语句(3)
                }
            }
        }
        return instance;
    }
}
```
明明已经使用了同步代码块来进行加锁了,为什么还要检验两次呢?要解释这个问题之前,我们先要了解下在执行**语句(3)**时发生了什么. **语句(3)**并不是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情.
>1. 给 instance 分配内存
>2. 调用 DCLSingleton 的构造函数来初始化成员变量
>3. 将instance引用指向分配的内存空间（执行完这步 instance 不为 null ）

这里涉及到JVM内存模型中的重排序问题,在不存在依赖性的情况下,JVM可以进行重排序优化.首先上述步骤1必须在步骤2和步骤3之前执行,但是步骤2,和步骤3之间并不存在依赖关系,所以有可能执行的顺序是`1->2->3`,也有可能是`1->3->2`.在执行顺序为`1->3->2`的情况下,如果线程1进入了同步块中,并执行到了**`语句(3)`**中的`1->3`,`instance`已经有指向的内存空间,所以此时`instance != null`.那么问题来了,刚好在此刻有线程2进入了`getInstance()`方法,并且刚好执行了**`语句(1)`**,此时的`if(instance == null)`返回的为`false`,所以返回将一个并未完全完成实例化的`instance`对象,那么此刻肯定会造成程序出现问题.所以我们引入了`volatic`关键字,用来防止JVM进行重排序优化(**JDK1.5**前`volatic`关键字存在问题,在**JDK1.5**之后进行了修复).这样使用`volatic`关键字和**`语句(1)`**就达到多线程下线程安全的目的.同时**`语句(1)`**还有一个作用就是在`instance`已经初始化之后,就无需进入同步代码块进行等待操作,大大的提高了程序的效率.**`语句(2)`** 的作用就是在多个线程同时执行了**`语句(1)`**,并在同步代码块之外进行等待时,为了防止产生多个instance实例.

### 静态内部类单例模式
可能有些人觉得双重检验锁模式的代码不够优雅,好吧,那就来种优雅的.使用静态内部类来实现单例模式.不多说,先上代码.

```java
/**
 * 静态内部类实现单例
 */
public class StaticInnerSingleton {


    private StaticInnerSingleton() {
    }

    //静态内部类
    private static class SingletonHolder {
        private static final StaticInnerSingleton INSTANCE = new StaticInnerSingleton();
    }

    public static StaticInnerSingleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
这样实现的好处既保证了线程安全,又是懒汉式的,简直无敌,个人比较推崇这种实现方式.毕竟双重检测锁的写法有些人并没有完全理解,而且写起来要考虑下逻辑也是有点麻烦.当然如果是直接引用标准的双重检测锁写法的当我没说`0.0`.
### 枚举实现单例模式
最后讲一种最少代码量实现单例的方案.同时也是&laquo;Effective Java&raquo;中推荐的,使用枚举(Enum).

```java
public enum  EnumSingleton {
    INSTANCE;

    public void doTest(){
        System.out.printf("doTest in enum singleton");
    }

    public static void main(String[] args) {
        EnumSingleton singleton = EnumSingleton.INSTANCE;
        singleton.doTest();
    }

}
```
创建枚举默认就是线程安全的，而且使用枚举还能防止反序列化导致重新创建新的对象,再看看代码量,乖乖,简直利器.但是用枚举实现单例,我看到确实不是很多.有可能很多人对枚举使用的不多,而且使用`JDK1.5`之后出来的,可能都不太熟.

### 单例使用注意
除了枚举实现的方案外,在Java处理序列化和反序列化时,会破坏单例的设计.因为反序列化得到的对象，和序列化之前的对象，不是同一个对象，它们的内存地址不相同。所以,假设一个单例类实现了Serializable接口，反序列化时内存里就会出现多个实例，这违背了当初设计单例的初衷。但是Java的序列化机制提供了一个钩子方法，即私有的`readresolve()`方法，可以让我们来控制反序列化时得到的对象。以双重检测锁单例模式为例

```java
/**
 * 双重检验锁单例模式
 */
public class DCLSingleton implements Serializable{

    private volatile static DCLSingleton instance = null;

    private DCLSingleton(){}

    public static DCLSingleton getInstance(){
        if(instance == null){ //语句(1) 同步块外
            synchronized (DCLSingleton.class){
                if(instance == null){ //语句(2)同步块内,为了防止产生多个instance实例
                    instance = new DCLSingleton(); //语句(3)
                }
            }
        }
        return instance;
    }

    /**
     * 反序列化时要做处理
     * @return 返回原来的实例
     * @throws ObjectStreamException
     */
    private Object readResolve() throws ObjectStreamException {
        return instance;
    }
}
```

### 总结

一般严格的来说,实现单例的方案总共为5种,分别为饿汉,懒汉,双重检测锁,静态内部类,枚举方案实现单例.有的人说是有7种,其中有些在多线程下会出现线程同步问题,如果不考虑多线程的话,其实还有几种不严格的写法,这里就不提了,百度上肯定有的.从性能上来说,静态内部类和枚举应该是比较好的.当然,如果从实际业务上来设计单例的话,还要考虑这个单例的功能如何,所以要具体问题具体分析.
