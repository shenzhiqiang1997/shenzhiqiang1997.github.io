---
title: Reference-引用
date: 2018-07-31 14:36:00
tags:
    - Java
    - 基础
categories:
    - Java
---
# 概述
Java中的引用分为StrongReference、SoftReference、WeakReference、PhantomReference分别为强引用、软引用、弱引用和虚引用，且他们的强度从前往后依次减弱。
<!-- more -->
# StrongReference（强引用）
JVM中默认的引用，只有当对象不再被引用时该对象才会被回收，即使是内存不足或者发生OOM时也不会因其他原因被回收。
# SoftReference（软引用）
当对象只有软引用时并且内存不足快要OOM时才会将该对象回收。一般用于对内存敏感的高速缓存。
```java
public class ReferenceTest {
    public static void main(String[] args) throws InterruptedException {
        Object o = new Object();
        SoftReference<Object> sr = new SoftReference<>(o);
        /* 此时该对象具有强引用，不会被回收 */
        System.out.println(sr.get());

        /* 这之后对象只剩下了软引用 */
        o = null;

        /* 但由于内存充足 该对象仍然不会被回收 */
        System.out.println(sr.get());
    }
}

```
# WeakReference（弱引用）
当对象只有弱引用时，无论内存状况如何，在触发GC后，该对象将被尽快回收。  
一般用于映射但又不介意对象被回收的情况，亦或是用于缓存。  
```java
public class ReferenceTest {
    public static void main(String[] args) throws Exception {
        Object o = new Object();
        WeakReference<Object> wr = new WeakReference<>(o);
        /* 此时该对象具有强引用，不会被回收 */
        System.out.println(wr.get());

        /* 这之后对象只剩下了弱引用 */
        o = null;
        System.gc();

        /* 对象只剩下弱引用之后会被尽快回收 */
        System.out.println(wr.get());

    }
}
```
**WeakHashMap**是一个使用虚引用的典型的例子，被放入其中的键会被包装为WeakReference，允许键在只剩下弱引用时将对应的键值回收。
```java
class Key{
    private int id;
    public Key(int id){
        this.id = id;
    }

    @Override
    protected void finalize() throws Throwable {
        System.out.println("finalize Key "+ id);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Key key = (Key) o;
        return id == key.id;
    }

    @Override
    public int hashCode() {

        return Objects.hash(id);
    }
}

public class ReferenceTest{
    public static void main(String[] args) {
        WeakHashMap<Key,Object> whm = new WeakHashMap<>();

        for (int i = 0; i < 10; i++) {
            whm.put(new Key(i),new Object());
        }

        System.gc();

    }
}
```
# PhantomReference（虚引用）
当对象只有虚引用时，其生存时间不受影响，在该对象被回收时，会被加入到引用队列来指示该对象已经被回收。  
一般用于代替对象的finalize方法，在对象回收前做相应的清理工作。  
虚引用的使用需要与ReferenceQueue配合，将虚引用引用的对象放入ReferenceQueue队列中。
```java
class ReferenceObject{
    @Override
    protected void finalize() throws Throwable {
        System.out.println("finalize");
    }
}
public class ReferenceTest {
    public static void main(String[] args){
        ReferenceObject ro = new ReferenceObject();
        PhantomReference<ReferenceObject> sr = new PhantomReference<>(ro,new ReferenceQueue<>());
        /* 虚引用总是返回null */
        System.out.println(sr.get());

        /* 这之后对象只剩下了虚引用 */
        ro = null;

        /* 对象只剩下虚引用 将得到null */
        System.out.println(sr.get());

        /* 触发GC */
        System.gc();
        
        /*
         虽然触发了GC 但因为用虚引用引用了对象
         所以可以在回收前代替finalize进行回收前的清理工作
        */
        System.out.println("do some work");

        /* 之后才会执行对象的finalize方法 */
    }
}
```
