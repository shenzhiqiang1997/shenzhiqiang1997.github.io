---
title: Callable和FutureTask执行异步任务
date: 2018-03-08 00:25:06
tags: 
    - 多线程
    - Java
categories:
    - 多线程
---
* 一般的用法就是通过Callable对象得到异步任务 FutureTask对象
再用FutureTask对象创建线程并执行该异步任务
<!-- more -->
这之后主线程可以去做一些其他耗时工作
等需要使用异步任务的计算结果时 
再通过FutureTask对象.get()得到异步任务计算结果
* **切忌**在启动线程执行异步任务之后
立马调用FutureTask对象.get()立马去获取结果
这样主线程会因为异步任务线程还未执行完任务而一直阻塞 浪费了系统资源
* 使用实例
    ```java
    public class CallableTest {
        @Test
        public void test() throws Exception {
            /*通过Callable对象新建一个异步任务*/
            FutureTask<Integer> task=new FutureTask<>(new MyCallable());
    
            /*创建并启动一个线程来执行异步任务*/
            new Thread(task).start();
    
            /*主线程做一些其他工作 模拟耗时*/
            Thread.sleep(3000);
    
            /*主线程完成其他工作之后 异步任务应该已经完成了
            查看异步任务结果*/
            System.out.println(Thread.currentThread().getName()+" get the asynchronous task result at "+new Date());
            System.out.println(task.get());
    
        }
    }
    class MyCallable implements Callable<Integer>{
    
        /*重写call() 异步执行带返回值的任务*/
        @Override
        public Integer call() throws Exception {
            System.out.println(Thread.currentThread().getName()+" start the asynchronous task at "+new Date());
            /*模拟计算耗时*/
            Thread.sleep(2000);
            /*返回一个0到10的随机值作为任务结果*/
            System.out.println(Thread.currentThread().getName()+" end the asynchronous task at "+new Date());
            return new Random().nextInt(10);
        }
    }
    ```