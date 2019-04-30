---
title: SpringMVC-工作原理
date: 2018-08-27 17:14:00
categories:
    - SpringMVC
tags: 
    - SpringMVC
    - 原理
    - 框架
    - Java
    - 设计模式
---
# 组件
SpringMVC的运行原理中主要依靠下面这些核心的组件
## DispatcherServlet
前端控制器，负责接收来自用户的请求，并控制协调其他多个组件共同工作，起到了解耦的作用。
<!-- more -->
## HandlerMapping
处理器映射器，负责根据请求映射出对应的处理器（若有拦截器则一并返回），一般通过配置或者注解来映射。
## Handler
处理器，负责对请求进行处理，处理的结果是ModelAndView（数据和视图逻辑地址名）。  
我们熟知的处理器有Controller、Servlet。
## HandlerAdapter
处理器适配器，负责适配不同的处理器，使得不同的处理器都能够按统一规范处理请求，以便扩展处理器族。
## ViewResolver
视图解析器，负责将ModelView解析为View。
# 工作流程
![image](https://note.youdao.com/yws/public/resource/708ad1b1ee41c2b335e0cac5c8d83d03/xmlnote/88523D3E159D41C8B82B1F43B29911FC/14759)  

<center>SpringMVC的工作流程图</center>  

1. DispatcherServlet接收用户请求。
2. DispatcherServlet调用HandlerMapping。
3. HandlerMapping根据请求找到相应的Handler和拦截器，返回HandlerExecutionChain（处理执行链，包含拦截器和处理器对象）给DispatcherServlet。
4. DispatcherServlet调用Handler对应的HandlerAdpater。
5. HandlerAdapter调用Handler来处理请求。
6. Handler处理完毕后返回ModelAndView给HandlerAdapter。
7. HandlerAdapter将ModelAndView返回给DispatcherServlet。
8. DispatcherServlet调用ViewResolver。
9. ViewResolver将ModelAndView解析为View。
10. ViewResovler将View返回给DispatcherServlet。
11. DispatcherServlet利用View渲染视图。
12. DispatcherServlet响应用户请求，返回渲染好的视图。  

# 参考资料
* [SpringMVC工作原理](https://www.cnblogs.com/xiaoxi/p/6164383.html)
