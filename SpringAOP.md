---
title: SpringAOP
date: 2020-01-21 17:08:12
categories: Spring
tags: 
- AOP
---


{% asset_img image01.png  %}

<!-- more -->

# 在Spring中启用@AspectJ注解支持

- 必须依赖`aopalliance.jar`、`aspectjweaver.jar`、`spring-aspects.jar`
- xml的<beans>根元素中配置aop Schema
- 要在SpringIoc容器中启用AspectJ注解支持，只要在Bean配置文件中定义一个空的XML元素`<aop:aspectj-autoproxy/>`，当SpringIoc容器检测到Bean配置文件中的`<aop:aspectj-autoproxy/>`元素时，会自动为与AspectJ切面匹配的Bean创建代理

# 如何把类声明为一个切面

把横切关注点的代码抽象到切面的类中，在AspectJ注解中，切面只是一个带有`@AspectJ`注解的`Java类`。

- 需要把该类放入到IOC容器中` @Component`
- 需要声明为一个切面 `@AspectJ`

# AspectJ支持五种类型的通知注解

通知是标注有某种注解的简单的java方法。在方法前添加通知注解如@Before注解，并将切入点表达式的值作为注解值。

```java
@Before("execution(public int com.aop.math.add(int,int))") public void beforeMethod(JoinPoint joinPoint){
    	System.out.println("The method begins....");
		  System.out.println(joinPoint.getTarget());
		  System.out.println(joinPoint.getSignature().getName());
		  System.out.println(Arrays.asList(joinPoint.getArgs()));
}
```

## @Before 

**前置通知**，在方法执行之前执行。

## @After

**后置通知**，在方法执行之后（无论是否发生异常）执行，即连接点返回结果或者抛出异常的时候，后置通知来记录方法的终止。

在后置通知中，还不能访问目标方法执行的结果，需要在返回通知中访问。

## @AfterReturning

**返回通知**，在方法正常结束执行代码之后执行。返回通知是可以访问方法的返回值的

```java
@AfterReturning(value = "execution(public int com.aop.math.*(int,int))", returning = "result") public void afterReturning(
   JoinPoint joinPoint, Object result) {
  System.out.println(joinPoint.getSignature().getName());
  System.out.println("The method returning....result:" + result);
}
```

## @AfterThrowing

**异常通知**，在方法抛出异常之后执行。可以访问异常对象，切可以指定在出现特定异常是再执行通知代码。

```java
@AfterThrowing(value = "execution(public int com.aop.math.*(int,int))", throwing = "e") public void afterThrowing(
   Exception e) {
  System.out.println("The method AfterThrowing...." + e);

}
```

## @Around

**环绕通知**，围绕着方法执行。

- 环绕通知需要携带ProceedingJoinPoint类型的参数；
- 环绕通知类似于动态代理的全过程，ProceedingJoinPoint类型的参数可以决定是否执行目标方法；
- 且环绕通知必须有返回值，返回值即为目标方法的返回值

```java
@Around(value = "execution(public int com.aop.math.*(int,int))") public Object afteAroundrThrowing(
   ProceedingJoinPoint proceedingJoinPoint) {
  Object result = null;
  try {
    System.out.println("前置通知...");
    //执行目标方法
    result = proceedingJoinPoint.proceed();
    System.out.println("返回通知...");
  } catch (Throwable throwable) {
    throwable.printStackTrace();
    System.out.println("异常通知...");
  }
  System.out.println("后置通知...");
  System.out.println("The method around...." + result);
  return result;
}
```

# 切面的优先级

当一个实现类有多个切面时，可以使用`@Order`注解制定切面的优先级，值越小，优先级越高。`@Order`的默认值为int的最大值`2<<31-1`

```java
public @interface Order {
    int value() default 2147483647;
}
```

```java
@Order(0) @Aspect @Component public class LoggingAspect {}
@Order(1) @Component @Aspect public class ValidationAspect {}
```

如上面两个切面，`LoggingAspect`中的切面优先执行。

# 重用切点表达式

- 定义一个方法，声明切入点表达式，一般情况下，该方法不需要添加其他的代码。
- 使用`@Pointcut`来声明切入点表达式
- 后面的其他同志直接使用方法名来直接阴影切入点表达式。

```java
@Component @Aspect public class ValidationAspect {
   @Pointcut(value = "execution( public int aop.ArithmeticCaculator.*(..))") 
  public void declareJoinPointExpression() {}
   @Before("declareJoinPointExpression()") public void validateArgs(JoinPoint joinPoint) {
      System.out.println("validate:" + Arrays.asList(joinPoint.getArgs()));
   }
}
```