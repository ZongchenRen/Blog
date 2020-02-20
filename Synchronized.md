---
title: Synchronized
date: 2020-02-20 22:49:05
categories: Java
tags:
- 并发
- 同步
- 锁
---

{% asset_img image01.png  %}

<!-- more -->



# 通常情况

如下代码，两个线程同时执行，修改`i`的值，执行1000000次，预期结果是输出2000000，但实际结果每次执行都不相同，到不了20000，这种多线程环境下结果不符合预期的情况，就是我们通常所谓的线程不安全。

```java
public class DisappearRequest1 implements Runnable {
   private static DisappearRequest1 instance = new DisappearRequest1();
   private static int i = 0;

   public static void main(String[] args) throws InterruptedException {
      Thread t1 = new Thread(instance);
      Thread t2 = new Thread(instance);
      t1.start();
      t2.start();
      t1.join();
      t2.join();
      System.out.println(i);
   }

   @Override public void run() {
      for (int j = 0; j < 1000000; j++) {
        i++;
      }

   }
}
```

# Sychronized的两个用法

## 对象锁（有两种形式）

$$
锁的对象是this或者新创建的对象，如下面的同步代码块的lock;
$$

### 形式1：方法锁

 `方法锁`（修饰普通方法，默认锁对象为this当前实例对象）

```java
public class SynchronizedObjectMethed3 implements Runnable {
   static SynchronizedObjectMethed3 instance = new SynchronizedObjectMethed3();

   public static void main(String args[]) throws InterruptedException {
      Thread t1 = new Thread(instance);
      Thread t2 = new Thread(instance);
      t1.start();
      t2.start();
      t1.join();
      t2.join();
      while (t1.isAlive() || t2.isAlive()) {

      }
      System.out.println("finish");
   }

   @Override public void run() {
      doSomething();
   }

   public synchronized void doSomething() {
      System.out.println("我是对象锁的方法修饰符形式，我叫" + Thread.currentThread().getName());
      try {
        Thread.sleep(3000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println(Thread.currentThread().getName() + "执行结束了");
   }
```

### 形式2：同步代码块

`同步代码块锁`（自己手动指定锁对象）

```java
public class SynchronizedObjectCodeBlock2 implements Runnable {
   private static SynchronizedObjectCodeBlock2 instance = new SynchronizedObjectCodeBlock2();
	 private final Object lock = new Object();
  
   @Override public void run() {
      synchronized (lock) { //此处lock 可以使用this代替
        System.out.println("我是对象锁的代码块形式，我叫" + Thread.currentThread().getName());
        try {
           Thread.sleep(3000);
        } catch (InterruptedException e) {
           e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "运行结束。");
      }
   }
  
   public static void main(String args[]) {
      Thread t1 = new Thread(instance);
      Thread t2 = new Thread(instance);
      t1.start();
      t2.start();
      while (t1.isAlive() || t2.isAlive()) {

      }
      System.out.println("finish");
   }
}
```

## 类锁

### 概念

**只有一个Class对象：**Java类可能有很多对象，但只有一个Class对象。

**本质：**所谓类锁，不过是Class对象的锁而已。

**用法和效果：** 类锁只能同一时刻被一个对象拥有。

### 形式1： synchronized可加在static方法上

```java
public class SynchronizedClassStatic4 implements Runnable {
   static SynchronizedClassStatic4 instance1 = new SynchronizedClassStatic4();
   static SynchronizedClassStatic4 instance2 = new SynchronizedClassStatic4();

   public static synchronized void doSomething() {//关键字加载静态方法上，是全局的
      System.out.println("我是类锁的static形式，我叫" + Thread.currentThread().getName());
      try {
        Thread.sleep(3000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println(Thread.currentThread().getName() + "执行结束了");
   }

   public static void main(String args[]) throws InterruptedException {
      Thread t1 = new Thread(instance1);
      Thread t2 = new Thread(instance2);
      t1.start();
      t2.start();
      t1.join();
      t2.join();
      while (t1.isAlive() || t2.isAlive()) {

      }
      System.out.println("finish");
   }

   @Override public void run() {
      doSomething();
   }
}
```

### 形式2： synchronized(*.class)代码块

```java
public class SynchronizedClassClass5 implements Runnable {
   static SynchronizedClassClass5 instance1 = new SynchronizedClassClass5();
   static SynchronizedClassClass5 instance2 = new SynchronizedClassClass5();

   public static void main(String args[]) throws InterruptedException {
      Thread t1 = new Thread(instance1);
      Thread t2 = new Thread(instance2);
      t1.start();
      t2.start();
      t1.join();
      t2.join();
      while (t1.isAlive() || t2.isAlive()) {

      }
      System.out.println("finish");
   }

   @Override public void run() {
      doSomething();
   }

   private void doSomething() {
      synchronized (SynchronizedClassClass5.class) {
        System.out.println("我是类锁的*.class形式，我叫" + Thread.currentThread().getName());
        try {
           Thread.sleep(3000);
        } catch (InterruptedException e) {
           e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "执行结束了");
      }

   }
}
```



文章开头的第一个问题就可以使用上面四种所得方式进行处理了。