---
title: Spring中@Autowired和@Resource的区别
date: 2018-11-09 20:33:00
tags: 
    - Spring
    - 框架
    - 区别
    - Java
categories:
    - Spring
---
# @Autowired
是属于Spring的注解。  
<!-- more -->
默认按照类型进行注入，当有没有或者有多个该类型的bean时将会抛出异常。  
当有多个该类型的bean的时候，可以通过`@Qualifier`指定要注入的bean的名称。 
# @Resource
是属于J2EE的注解。  
它有两个属性：`name`和`type`，分别表示要注入的bean名称和bean的类型。  
* 默认情况下，先按照名称进行注入；如果没有对应名称的bean，则按照类型进行注入；如果都不行，则抛出异常。 
* 当指定了name和type时，如果有相同类型和名称的bean则注入，否则抛出异常。
* 当指定了name时，如果有相同名称的bean则注入，否则抛出异常。
* 当指定了type时，如果有相同类型的bean则注入，如果没有或者有多个将抛出异常。