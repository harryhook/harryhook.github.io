---
title: Adapter模式
date: 2019-11-30 15:34:56
tags: [结构型模式]
categories: [设计模式]
---

联想到日常生活中的适配器， 来看下程序设计中的适配器模式是什么样的
<!--more-->

# 什么是适配器模式

> 将一个目前自己不能使用的类，通过适配器进行包装，得到我们想要的功能


# 适配器组成

源角色（Adaptee）： 需要适配的类
适配器（Adapter）：把源角色转换成目标接口
目标（Target）： 客户所期待的接口
Client：...

# 适配器uml

![UML](/适配器模式-UML.jpg)

通过uml图可以看到，适配器可以将Target一个类的接口变换成客户端所期待的另一种接口，从而使原本接口不匹配而无法在一起工作的两个类能够在一起工作。也有人把这种模式叫做包装（Wrapper）模式。
# 具体例子
## 类适配器模式
```java
public class Client {
    public static void main(String[] args) {
        ITiger flyTiger = new FlyTiger("飞天虎");
        flyTiger.eat();
    }
}

interface ITiger {
    void eat();
}

class Falcon {
    private String name;

    public Falcon(String name) {
        this.name = name;
    }

    public void fly() {
        System.out.printf("%s 会飞！\n", this.name);
    }
}

class FlyTiger extends Falcon implements ITiger {

    private String name;

    public FlyTiger(String name) {
        
        super(name);

        this.name = name;
    }

    @Override
    public void eat() {
        this.fly();
        System.out.printf("%s 要吃肉！\n", this.name);
    }
}
```

**输出**:

```java
飞天虎 会飞！
飞天虎 要吃肉！
```
看一下类适配器的结构
![类适配器模式](/类适配器模式.jpg)


## 对象适配器模式

```java
public class Client {
    public static void main(String[] args) {
        ITiger flyTiger = new FlyTiger();
        flyTiger.eat();
    }
}

interface ITiger {
    void show();
}

class Falcon {

    public Falcon() {
    }

    public void fly() {
        System.out.printf("我会飞！\n");
    }
}

class FlyTiger implements ITiger {
    private Falcon falcon = new Falcon();

    public FlyTiger() {
    }

    @Override
    public void show() {
        falcon.fly();
    }
}
```
看一下对象适配器的结构

![对象适配器模式](/对象适配器模式.jpg)

# 类装配器与对象适配器的区别

## 类适配器模式
**优点：**

* 使用方便， 代码简单
* 只需引用一个字段，不需要额外的字段引用Adaptee实例

**缺点：**

* 高耦合，灵活性差
* 使用继承


## 对象适配器模式

**优点：**

* 灵活性高、低耦合
* 采用组合， 而非继承

**缺点:**：

* 复杂
* 需要引入对象

## 总结

> 需要重新定义Adaptee时， 采用类适配器;

> 需要同时使用源类和其子类时采用对象适配器;

> 建议多使用组合（对象适配器），少使用继承（类适配器）

# 使用场景

* 系统复用现有的类， 当该类的接口不满足当前需求时， 利用适配器可以使得原本不兼容的接口一起工作
* 多个组件类似， 但接口不同意需要经常来回切换， 使用适配器模式可以让客户端统一调通， 节约成本

# JDK中的应用

```java
java.util.Arrays#asList()

//InputStreamReader 继承了Readr类，但要创建它的对象必须在构造函数中传入一个InputStream）(InputStream→Reader 字节到字符)
java.io.InputStreamReader

// 同理 OutputStreamWriter
java.io.OutputStreamWriter(OutputStream)
```