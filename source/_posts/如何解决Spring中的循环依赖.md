---
title: 如何解决Spring中的循环依赖
date: 2018-11-09 20:38:00
tags: 
    - Spring
    - 框架
    - Java
categories:
    - Spring
---
# 构造器循环依赖异常
当Spring通过构造器进行依赖注入时，就有可能出现循环依赖异常。考虑下面的情况
<!-- more -->  
```java
@Component
class A{
    private B b;
    @Autowired
    public A(B b){
        this.b = b;
    }
}
```
```java
@Component
class B{
    private A a;
    @Autowired
    public B(A a){
        this.a = a;
    }
}
```
这样就会抛出异常，因为IOC容器在实例化a时，构造方法需要依赖b，IOC容器将会实例化b，而b的构造器又依赖a，此时a还在创建过程中，就产生一个循环依赖，这种情况Spring是处理不了的，从而抛出异常。
# 解决方法
通过setter或者注解进行依赖注入就能够解决上面的问题。
```java
@Component
class A{
    @Autowired
    private B b;
}
```
```java
@Component
class B{
    @Autowired
    private A a;
}
```
因为通过setter或者注解进行依赖注入时，IOC容器会用反射调用无参的构造方法实例化bean，然后将该bean先缓存到一个map中，从而提前暴露一个创建中的bean，之后再进行属性的注入。
现在整个过程变成了，IOC容器先实例化a，缓存创建中的a，然后再进行属性b的注入，IOC容器实例化b，缓存创建中的b，然后再进行属性a的注入，此时用之前缓存的a进行注入即可。这样就解决了循环依赖的问题。
# 问题
以上方法只能解决单例bean的循环依赖问题，当bean的作用域为prototype时，IOC容器将不会缓存对应的bean，无法提前暴露一个创建中的bean。
