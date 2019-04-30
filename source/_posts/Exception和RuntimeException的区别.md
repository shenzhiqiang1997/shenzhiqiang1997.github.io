---
title: Exception和RuntimeException的区别
date: 2018-07-24 16:03:00
tags: 
    - Java
    - 基础
categories:
    - Java
---
* 首先两者的本质不同在于  
**Exception**为编译时异常，也称为被检查的异常，必须在编译阶段进行检查，并由**try-catch程序处理**。  
**RuntimeException**为运行时异常，也称为不被检查的异常，在编译阶段不会被检查，当发生时交由**JVM处理**。  
<!-- more -->
* 其次两者的使用场景不同在于  
**RuntimeException**:当发生异常时，**无论怎么补救，程序都无法继续执行**下去时，考虑使用或者继承RuntimeExcpetion。  
例如0作为除数进行运算的异常发生之后，对该结果数的后续运算无法继续下去了，就要考虑采用RuntimeException将该异常交由JVM来处理。  
**Exception**:当发生异常时，如果**通过一定的处理来补救，程序仍能继续执行下去**，考虑使用或者继承Exception。  
例如创建File对象时目标的文件不存在的异常发生之后，只是一个文件无法打开而已，接下来的部分可以通过try-catch程序来进行提示并从而使得后续程序继续运行下去，就要考虑采用Exception在编译时强制程序员考虑该异常情况。