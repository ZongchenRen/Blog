---
title: Java回调函数
date: 2020-02-01 10:05:55
categories: 程序设计
tags: 
- Java
- 程序设计

---

{% asset_img image01.png  %}
n
<!-- more -->

# 在C或者C++中回调函数的定义

程序在调用一个函数时，将自己的函数的地址作为参数传递给程序调用的函数时（那么这个自己的函数称回调函数）

# Java中定义

Java中没有指针，不能传递方法的地址，一般采用接口回调实现：把实现某一接口的类创建的对象的引用赋给该接口声明的接口变量，那么该接口变量就可以调用被类实现的接口的方法。



#  回调实现原理

首先创建一个回调对象，然后再创建一个控制器对象，将回调对象需要被调用的方法告诉控制器对象。控制器对象负责检查某个场景是否出现或某个条件是否满足。当此场景出现或此条件满足时，自动调用回调对象的方法。

# 实现

1. 创建一个回调接口

```java
// 回调接口
public interface ICallBack {
  void run();
}
```

2. 创建实现类

```java
// 实现类
public class CallBack implements ICallBack {
  public void run() {
    System.out.println("执行run方法...");
  }
}
```

3. 控制类

```java
// 控制类
public class Controller {
  private ICallBack callBack;

  public Controller(ICallBack callBack) {
    this.callBack = callBack;
  }

  //***核心***
  public void doSomething() {
    callBack.run();
  }
}
```

* 运行

```java
public class ProgramRun {
  public static void main(String args[]) {
    Controller controller = new Controller(new CallBack());
    controller.doSomething();
  }
}
```

# 参考

https://blog.csdn.net/bjyfb/article/details/10462555

http://www.jcodecraeer.com/a/chengxusheji/java/2012/0822/370.html