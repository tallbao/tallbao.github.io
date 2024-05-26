---
layout: post
author: ᴢʜᴀɴɢ
title: "Spring AOP经典应用场景(AOP 方法失效问题)"
date: 2023-05-17
music-id: 
permalink: /archives/2023-05-17/1
description: "Spring AOP"
---

### 关于学习redis优惠券秒杀实现一人一单遇到的问题
#### Spring AOP 的原理是在原有方法外面增加一层代理，所以在当前类调用 AOP 方法时，因为 this 指向的是当前对象，而不是代理对象，所以 AOP 会失效。
```java
1.pom.xml引入org.aspectj aspectjweaver
2.启动类添加注解暴露代理对象@EnableAspectJAutoProxy(exposeProxy = true)
3.同一类引用事务注解失效解决代码:
Long userId = UserHolder.getUser().getId();
synchronized(UserId.toString().intern()){   //intern()是加到字符串常量池里 常量池才会复用，new的话是每次在堆里开辟一个新对象，因此值一样对象地址也不同
    //获取代理对象(事务)
    IVoucherorderService proxy = (IVoucherorderService) AopContext.currentProxy();
    return proxy.createVoucherorder(voucherId);
}

@Transactiona
public Result createVoucherorder(Long voucherId){
    ...
}
```
---
### 一、日志处理
### 二、事务控制
### 三、参数校验
### 四、自定义注解
### 五、AOP 方法失效问题
1. ApplicationContext
2. AopContext
3. 注入自身
#### AOP 提供了一种面向切面操作的扩展机制，通常这些操作是与业务无关的，在实际应用中，可以实现：日志处理、事务控制、参数校验和自定义注解等功能。

Spring AOP 的原理参阅：[《Spring中的AOP和动态代理》](https://mp.weixin.qq.com/s?__biz=MzU1MzQ0NjU0Ng==&mid=2247485294&idx=1&sn=bf931565df839c98ff12b1bfdd14f89f&scene=21#wechat_redirect)
# 一、日志处理
#### 在调试程序时，如果需要在执行方法前打印方法参数，或者在执行方法后打印方法返回结果，可以使用切面来实现。
```java
@Slf4j
@Aspect
@Component
public class LoggerAspect {
 
    @Around("execution(* cn.codeartist.spring.aop.sample.*.*(..))")
    public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
        // 方法执行前日志
        log.info("Method args: {}", joinPoint.getArgs());
        Object proceed = joinPoint.proceed();
        // 方法执行后日志
        log.info("Method result: {}", proceed);
        return proceed;
    }
}
```
# 二、事务控制
#### Spring 提供的声明式事务也是基于 AOP 来实现的，在需要添加事务的方法上面使用 @Transactional 注解。
```java
@Service
public class DemoService {
 
    @Transactional(rollbackFor = Exception.class)
    public void insertBatch() {
        // 带事务控制的业务操作
    }
}
```
# 三、参数校验
#### 如果需要在方法执行前对方法参数进行校验时，可以使用前置通知来获取切入点方法的参数，然后进行校验。
```java
@Slf4j
@Aspect
@Component
public class ValidatorAspect {
 
    @Before("execution(* cn.codeartist.spring.aop.sample.*.*(..))")
    public void doBefore(JoinPoint joinPoint) {
        // 方法执行前校验参数
        Object[] args = joinPoint.getArgs();
    }
}
```
# 四、自定义注解
#### 因为 AOP 可以拦截到切入点方法，Spring 也支持通过注解的方式来定义切点表达式，所以可以通过 AOP 来实现自定义注解的功能。
#### 例如，自定义一个注解来实现声明式缓存，把方法的返回值进行缓存。
```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Cacheable {
 
    /**
     * 缓的Key
     */
    String key();
 
    /**
     * 缓存过期时间
     */
    long timeout() default 0L;
 
    /**
     * 缓存过期时间单位（默认：毫秒）
     */
    TimeUnit timeUnit() default TimeUnit.MILLISECONDS;
}
```
#### 然后定义一个切片来实现常规的缓存操作，先读缓存，缓存不存在时执行方法，然后把方法的返回结果进行缓存。
```java
@Aspect
@Component
public class AnnotationAspect {
 
    @Around("@annotation(cacheable)")
    public Object doAround(ProceedingJoinPoint joinPoint, Cacheable cacheable) throws Throwable {
        // 自定义缓存逻辑
        return joinPoint.proceed();
    }
}
```
# 五、AOP 方法失效问题
#### Spring AOP 的原理是在原有方法外面增加一层代理，所以在当前类调用 AOP 方法时，因为 this 指向的是当前对象，而不是代理对象，所以 AOP 会失效。
```java
@Service
public class DemoService {
 
    public void insert() {
        // 该方法事务会失效
        insertBatch();
    }
 
    @Transactional(rollbackFor = Exception.class)
    public void insertBatch() {
        // 带事务控制的业务操作
    }
}
```
#### 解决这个问题的常用方法有下面三种：
### 1. ApplicationContext
#### 使用 ApplicationContext 来手动获取 Bean 对象，来调用 AOP 方法。
```java
@Service
public class DemoService {
 
    @Autowired
    private ApplicationContext applicationContext;
 
    public void insert() {
        DemoService demoService = applicationContext.getBean(DemoService.class);
        demoService.insertBatch();
    }
 
    @Transactional(rollbackFor = Exception.class)
    public void insertBatch() {
        // 带事务控制的业务操作
    }
}
```
### 2. AopContext
#### 使用 AopContext 工具类来获取当前对象的代理对象。

```java
@Service
public class DemoService {
 
    public void insert() {
        ((DemoService) AopContext.currentProxy()).insertBatch();
    }
 
    @Transactional(rollbackFor = Exception.class)
    public void insertBatch() {
        // 带事务控制的业务操作
    }
}
```
### 3. 注入自身
#### 使用 Spring 注入自身来调用 AOP 方法。
```java
@Service
public class DemoService {
 
    @Autowired
    private DemoService that;
 
    public void insert() {
        that.insertBatch();
    }
 
    @Transactional(rollbackFor = Exception.class)
    public void insertBatch() {
        // 带事务控制的业务操作
    }
}
```