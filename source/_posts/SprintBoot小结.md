---
title: SpringBoot小结
date: 2018-10-31 18:12:00
tags: 
    - SpringBoot
    - 框架
    - Java
categories:
    - SpringBoot
---
# SprintBoot特点
* 简化Maven配置  
Maven中依赖starter包中帮我们自动依赖了许多其他组件，简化了maven配置。
* 自动配置  
SpringBoot可以根据我们项目引用的jar进行相关bean的自动配置。
* 无XML配置  
SpringBoot直接使用properties文件和纯Java的方式进行配置，可以完全无XML配置。
* 独立运行  
SpirngBoot内嵌了Tomcat、Jetty等Web容器，可以直接打包成jar来运行
* 应用监控  
SpringBoot提供actuator来进行状态监控。 
<!-- more -->
# 自动配置原理
主类上的@SpringBootApplication是SpringBoot项目启动的关键
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
这个注解本身其实也用了三个重要的注解进行注解
```java
// 不重要的内容进行了省略
@Configuration
@EnableAutoConfiguration
@ComponentScan
......
public @interface SpringBootApplication {
    ......
}
```
即@Configuration @ComponentScan @EnableAutoConfiguration 这三个注解
## @Configuration
它即表明主类也是一个配置类，其中的@Bean定义将会被IOC容器读取并实例化到IOC容器中。
## @ComponentScan
这个注解即是IOC容器扫描被注解了的组件的位置，默认是主类所在的根包。
## @EnableAutoConfiguraiton
这个就是自动配置实现的根本。
它会借助SpringFactoriesLoader这个工具去classpath中搜索所有META-INF/spring.factories配置文件，将其中key为org.springframework.boot.autoconfigure.EnableAutoConfiguration的*AutoConfig类通过反射实例化，加载到IOC容器中，之后再根据这些配置实例来加载bean，完成bean的自动配置。
```java
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
......
```
