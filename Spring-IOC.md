---
title: Spring IOC
date: 2019-12-22 23:37:15
tags: Spring
---

{% asset_img image01.png  %}

# 简介

## IOC（Inversion of Control） 

其思想是反转资源获取的方向，传统的资源查找方式是要求组件想容器发送请求来查找资源，作为回应，容器适时地返回资源。

而应用了IOC之后，则是容器主动地将资源推送给它所管理的组件，组件所要做的仅是选择一种合适的方式来接收资源，这种行为也被称为查找的被动形式。


## DI（Dependency Injection）

IOC的另一种表述方式：即组件以一写预先定义好的方式（如：setter方法）接收来自如容器的资源注入。相较于IOC而言，这种表述更直接。


# Spring IOC 实现

> 在Spring IOC容器读取Bean配置 创建Bean实例之前，必须对容器进行实例化。只有在容器实例化后，才可以从IOC容器里获取Bean实例并使用。

Spring提供了两种类型的IOC容器实现


- BeanFactory

    IOC的基本实现。


- AppliacationContext

    提供了更多的高级特性，是BeanFactory的子接口,代表的是一个IOC容器.


    BeanFactory是Spring框架的基础设施，面向的是Spring本身；
    AppliacationContext面向的是使用Spring框架的开发者;
    几乎所有的应用场景都直接使用AppliacationContext而非底层的BeanFactory;
    无论使用何种方式，配置文件是相同的.


## AppliacationContext
AppliacationContext主要实现类：
- ClassPathXmlApplicationContext
 
    从类路径下加载配置文件。
    
- FileSystemXmlApplicationContext
    从系统文件中加载配置文件。
    
       相关父类或拓展类可查看spring源码。
       

# 推荐阅读

1. [  Spring IOC 容器源码分析](https://javadoop.com/post/spring-ioc)





