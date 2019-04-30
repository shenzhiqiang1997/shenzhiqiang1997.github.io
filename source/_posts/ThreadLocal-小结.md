---
title: ThreadLocal-小结
date: 2018-09-12 21:03:00
categories:
    - Java
tags: 
    - Java
    - 原理
    - 多线程
---
# ThreadLocal
## 什么是ThreadLocal
每个线程使用一个ThreadLocal对象时都会在线程中创建一个对应的副本，之后每次访问都是访问这个副本而不是ThreadLocal本身，ThreadLocal是线程私有的，是一种线程安全的方案。
<!-- more -->
## 实现原理
每个线程都有一个ThreadLocalMap，每当调用一个ThreadLocal对象的set()，将会在线程对象中的ThreadLocalMap中插入/更新key为该ThreadLocal对象，value为设置的值的键值对。  
每当调用一个ThreadLocal对象的get()，将会在线程对象的ThreadLocalMap中获取key为该ThreadLocal对象的键值对，然后将对应的value返回。
## 内存泄露
#### 为什么会内存泄露
在线程长时间不终止时，就算外部不再对ThreadLocalMap中的key和value引用，但ThreadLocalMap中始终对它们有着引用。这样的话，没用的键值对就始终不会被回收，造成内存泄露。
#### 如何避免内存泄露
其实ThreadLocalMap设计上就已经考虑到内存泄露的问题并做了一些工作。  
ThreadLocalMap中的Entry中是以虚引用的形式在引用着key，当外部不再引用key时，只剩下Entry对key的虚引用，在下次GC时该key会被回收，此时key变为null，但value仍然被Entry强引用。  
只要调用ThreadLocal的get()、set()、remove()，就会顺带把key为null的Entry清理掉，但如果我们不去调用，仍然会造成内存泄漏。  
所以避免内存泄露最好的方法就是在使用完ThreadLocal对象之后调用remove()清理掉它。