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

**锁的对象是this或者新创建的对象，如下面的同步代码块的lock;**

## 形式1：方法锁

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



# Synchronized性质

## 1.可重入

**可重入：**指的是同一线程的外层函数获得锁之后，内层函数可以直接再次获取该锁，也叫递归锁。

**好处：**避免死锁、提升封装性

**粒度：**线程而非调用（用3中情况来说明和pthread的区别）

1. 情况1：证明同一个方法是可重入的

```java
//可重入粒度测试：递归调用本方法
public class SynchronizedRecoursion6 {
   private int a = 0;
   public static void main(String args[]) {
      SynchronizedRecoursion6 synchronizedRecoursion6 = new SynchronizedRecoursion6();
      synchronizedRecoursion6.doSomething();
   }
   //递归调用本身
   private synchronized void doSomething() {
      System.out.println("doSomething , a=" + a);
      if (a == 0) {
        a++;
        doSomething();
      }
   }
}
```

输出>：

```shell
doSomething , a=0
doSomething , a=1
```



2. 情况2：证明可重入不要求是同一个方法

```java
//可重入粒度测试：调用类内部另外的方法
public class SynchronizedOtherMethod7 {
   public static void main(String args[]) {
      SynchronizedOtherMethod7 synchronizedOtherMethod7 = new SynchronizedOtherMethod7();
      synchronizedOtherMethod7.method1();
   }
   private synchronized void method1() {
      System.out.println("我是method1");
      method2();
   }

   private synchronized void method2() {
      System.out.println("我是method2");
   }
}
```

输出>:

```shell
我是method1
我是method2
```



3. 情况3：证明可重入不要求是同一个类中的

```java
//可重入粒度测试：调用父类方法
public class SynchronizedSuperClass8 {
   public synchronized void doSomething() {
      System.out.println("我是父类方法");
   }

   static class TestClass extends SynchronizedSuperClass8 {
      @Override public synchronized void doSomething() {
        System.out.println("我是子类方法");
        super.doSomething();
      }

      public static void main(String args[]) {
        TestClass testClass = new TestClass();
        testClass.doSomething();
      }
   }
}
```

输出>:

```shell
我是子类方法
我是父类方法
```

## 2.不可中断

一但这个锁已经被别人获得了，如果我还想获得，我只能选择等待或者阻塞，直到别的线程`释放`这个锁。如果别人永远不释放锁，那么我只能永远地等下去。

对比Lock类，可以拥有中断能力，第一点，如果我觉得我等的时间太长了，有权中断现在已经获取到锁的线程的执行；第二点，如果我觉得等待的时间太长了，不想再等了，也可以退出。



# 原理

## 加锁和释放锁的原理

**现象：**每一个类的实例对应一把锁，每一个synchronized方法都必须首先获得调用该方法的类的实例的锁，方能执行，否则线程会阻塞，而方法一旦执行，就独占了锁，直到该方法返回或抛出异常，才释放锁，释放以后，阻塞的线程才能获取这把锁。

**获取和释放锁的时机：**内置锁（监视器锁）

每一个java对象都可以用作一个实现同步的锁，这个锁就是内置锁，也叫监视器锁，线程在进入同步代码块之前，会自动获取这个锁，并且在退出（正常退出或者异常退出）同步代码块的的时候，会释放锁。

获取内置锁的唯一途径就是进入到这个锁所保护的同步代码块或者方法中。

**等价代码**

```java
//第一个方法和第二个方法是等价的
public class SynchronizedToLock9 {
   Lock lock = new ReentrantLock();

   public synchronized void method1() {
      System.out.println("我是synchronized形式的锁");
   }

   public void method2() {
      lock.lock();  //获取锁
      try {
        System.out.println("我是lock形式的锁");
      } finally {
        lock.unlock();  //释放锁
      }
   }
   public static void main(String args[]) {
      SynchronizedToLock9 synchronizedToLock9 = new SynchronizedToLock9();
      synchronizedToLock9.method1();
      synchronizedToLock9.method2();
   }
}
```

**深入JVM看字节码：** 反编译、monitor指令

```java
public class Decompilation {
   private Object object;
   public void insert(Thread thread) {
      synchronized (object) {

      }
   }
}
```
执行反编译

```shell
javac Decompilation.java
javap -verbose Decompilation.class
```

```shell
  public void insert(java.lang.Thread);
    descriptor: (Ljava/lang/Thread;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=2
         0: aload_0
         1: getfield      #2                  // Field object:Ljava/lang/Object;
         4: dup
         5: astore_2
         6: monitorenter
         7: aload_2
         8: monitorexit
         9: goto          17
        12: astore_3
        13: aload_2
        14: monitorexit
        15: aload_3
        16: athrow
        17: return
      Exception table:
         from    to  target type
             7     9    12   any
            12    15    12   any
      LineNumberTable:
        line 12: 0
        line 14: 7
        line 15: 17
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 12
          locals = [ class src/Decompilation, class java/lang/Thread, class java/lang/Object ]
          stack = [ class java/lang/Throwable ]
        frame_type = 250 /* chop */
          offset_delta = 4
}
SourceFile: "Decompilation.java"
```

解读monitorenter和monitorexit



## 可重入原理

**加锁次数计数器**

> JVM负责跟踪对象被加锁的次数

> 线程第一次给对象加锁的时候，计数变为1，每当这个相同线程在此对象向上再次获得锁时，技术会递增

> 每当有任务离开时，计数递减，当计数为0的时候，锁被完全释放




## 保证可见性的原理

**Java内存模型**

|                              |                              |
| ---------------------------- | ---------------------------- |
| {% asset_img image02.png  %} | {% asset_img image03.png  %} |

```tex
线程通信如何做到？
如上如，两个线程A B，本地内存都是共享变量的副本，都是是从主内存中复制一份，如果A B要实现通信，需要实现两个步骤：
1.线程A要把副本写到主内存中去；
2.线程B从主内存中读取；
这个过程由JMM（java内存模型）控制。
```

**synchronized若何做到可见性实现的**

代码块被`synchronized`关键字修饰，一旦执行完毕之后，被锁住的对象所做的任何修改，都要在**锁释放锁之前**，从**线程内存**中**写回到主内存**，同理，在进入代码块的**得到锁之**后，得到的锁对象数据，也是从**主内存中取出**出来的，从而保证了每一次的指向性都是可靠地。



# 缺陷

## 效率低

锁的释放情况少，视图获得锁的时候，不能设定超市时间，不能中断一个正在视图获得锁的线程。只有两种情况可以释放锁：

1. 代码执行完毕；

2. 执行异常；

## 不够灵活

加锁和释放锁的时机单一，每个锁仅有单一的条件（某个对象），可能是不够的

相较而言读写锁更灵活，读的时候不加锁，写的时候加锁。



## 无法知道是否成功获取到锁

Lock可以尝试，tryLock();



# 常见面试题

**1.使用注意点**

 `锁对象不能为空`：锁对象必须是实例对象，因为锁的信息是保存在对象头中的。

 `作用于不宜过大`：影响程序执行速度

` 注意避免死锁`：

**2.如何选择Lock和synchronized关键字？**

如果可以的话，尽量不要使用`Lock`和`synchronized`关键字，而是使用`juc`中的各种类，如Atomicxxx，CountDownLatch等，使用这些类的时候不需要我们做同步处理，可以避免出错。没有的话尽量使用synchronized，最后考虑lock.

> juc > synchronized > lock

**3.多线程访问同步方法的各种情况**

> 两个线程同时访问一个对象的同步方法
>
> 2.两个线程访问的是两个对象的同步方法
>
> 3.两个线程访问的是synchronized的静态方法
>
> 4.同时访问同步和非同步方法
>
> 访问同一个对象的不同的普通同步方法
>
> 6.同时访问静态synchronized方法和非静态synchronized方法
>
> 7.方法抛出异常后，会释放锁

# 思考

1.多个线程等待同一个synchronized锁的时候，JVM如何新选择下一个获取锁的是哪一个线程？

> 随机不可控  不公平锁



2.synchronized使得同时只有一个线程可以执行，性能较差，有什么办法可以提升性性能？

> 1、优化作用范围，不宜范围过大
>
> 2、使用其他类型的锁，如读写锁

3.我想更灵活地控制锁的获取和释放（现在释放锁的时机都被规定死了），怎么办？

> 自己实现一个lock接口

4.什么是锁的升级、降级？什么是JVM里的偏斜锁、轻量级锁、重量锁？

> 所谓锁的升级、降级，就是JVM优化synchronized运行的机制，当JVM检测到不同竞争状况时，会自动切换到合适的锁实现，这种切换就是锁的升级、降级

> 偏斜锁： 当竞争不激烈的时候，会默认使用偏斜锁。

> 轻量级锁： 如果某个线程试图锁定某个已经被偏斜的对象，JVM就需要撤销偏向锁，升级成为轻量锁。如果获取失败，则进入自旋锁，如果自旋到一定的次数还是不能得到锁，就进入重量锁。

> 自环锁： 线程获取锁失败后，为了避免让线程进入阻塞状态而采取的循环一定次数去试着获取锁的行为。

> 重量锁： 就是Java一开始对Synchronized实现的阻塞锁。一旦到了重量级锁的话，其他的线程都会被阻塞等待获得锁

# 总结

## 一句话介绍synchronized

JVM会自动通过使用monitor来加锁和解锁，保证了同时只有一个线程可以执行指定代码，从而保证了线程安全，同时具有可重入和不可中断的性质。