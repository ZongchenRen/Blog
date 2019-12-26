---
title: SpringBean配置方式
date: 2019-12-26 23:49:14
tags: Spring
---

{% asset_img image01.png  %}

## 简介

**Spring提供了BenaFactory和ApplicationContext两种类型的IOC容器实现，Spring又是如何将Bean装配到IOC容器中的呢？**

以前Java框架基本都采用了XML作为配置文件，但是现在Java框架又不约而同地支持基于Annotation的“零配置”来代替XML配置文件，Struts2、Hibernate、Spring都开始使用Annotation来代替XML配置文件了；而在Spring 3.x提供了三种选择，分别是：基于XML的配置、基于注解的配置和基于Java类的配置。

------

下面分别介绍下这三种配置方式：

## 基于XML的配置

应用场景：当使用三方类库时，比如DataSource、HibernateTemplate等，无法在类中标注注解信息，只能通过XML进行配置；而且命名空间的配置，比如aop、context等，也只能采用基于XML的配置。

- JavaBean

```java
public class HelloWorld {
   private String name;
   public void setName(String name) {
      this.name = name;
   }
   public void hello() {
      System.out.println("hello: " + name);
   }
   public HelloWorld() {
      System.out.println("HelloWorld类的无参构造器");
   }
   public void init() {
      System.out.println("do init  ......");
   }
   public void destroy() {
      System.out.println("do destroy  ...... ");
   }
}
```

- Xml

```XML
<bean id="helloWorld" class="spring01.HelloWorld" lazy-init="true" init-method="init"
      destroy-method="destroy">
    <property name="name" value="spring"/>
</bean>
```

通过属性注入，即setter方法注入Bean的值或者依赖的对象，**属性注入是最常用的注入方式**。

- 测试

```java
ConfigurableApplicationContext context = new ClassPathXmlApplicationContext(
  "applicationContext.xml");
HelloWorld helloWorld = (HelloWorld) context.getBean("helloWorld");
System.out.println(helloWorld);
helloWorld.hello();
context.close();
```

为了演示容器销毁时调用Bean的destroy-method，此处使用ConfigurableApplicationContext，ConfigurableApplicationContext扩展于ApplicationContext，新增了两个方法：refresh()和close()，让ApplicationContext具有启动，刷新和关闭上文的能力。

- 测试结果

```tex
HelloWorld类的无参构造器
setName: spring
do init  ......
spring01.HelloWorld@55fe41ea
hello: spring
do destroy  ...... 
```

从上出输出也可以看出，程序启动在获取Bean时，先创建Bean对象，然后设置属性setName，执行init方法，最后再获取Bean实例，当容器关闭时，调用Bean的destroy方法。

### XML装配常用的集合类

- JavaBean

```java
public class CollectionDemo {
   private String name;
   private List<String> list;
   private Map<String, Object> map;
   private Properties properties;
   private Set<String> set;
   private String[] array;
  //省略setter getter
}
```

- xml

```xml
<bean id="collectionDemo" class="spring01.CollectionDemo">
    <property name="name" value="装配集合类"/>
    <!--装配List类型的list-->
    <property name="list">
        <list>
            <value>value-list-1</value>
            <value>value-list-2</value>
            <value>value-list-3</value>
        </list>
    </property>
    <!--装配Map类型的map-->
    <property name="map">
        <map>
            <entry key="map1" value="value-key-1"/>
            <entry key="map2" value="value-key-2"/>
            <entry key="map3" value="value-key-3"/>
        </map>
    </property>
    <!--装配Properties类型的properties-->
    <property name="properties">
        <props>
            <prop key="pro1">value-pro-1</prop>
            <prop key="pro2">value-pro-2</prop>
            <prop key="pro3">value-pro-3</prop>
        </props>
    </property>
    <!--装配Set类型的set-->
    <property name="set">
        <set>
            <value>value-set-1</value>
            <value>value-set-2</value>
            <value>value-set-3</value>
        </set>
    </property>
    <!--装配String[]类型的array-->
    <property name="array">
        <array>
            <value>value-array-1</value>
            <value>value-array-2</value>
            <value>value-array-3</value>
        </array>
    </property>
</bean>
```

- 测试

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
CollectionDemo demo = ctx.getBean(CollectionDemo.class);
System.out.println(demo.getName());
System.out.println(demo.getProperties().getProperty("pro1"));
```

- 测试结果

```
装配集合类
value-pro-1
```



## 基于注解的配置

应用场景：当Bean的实现类是当前项目开发的，可以直接在Java类中使用基于注解的配置，配置相对比较简单。



## 基于Java类的配置

应用场景：当实例化Bean的逻辑比较复杂时，则比较适合基于Java类配置的方式。