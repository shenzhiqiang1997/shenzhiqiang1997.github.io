---
title: tiny-spring-aop-学习总结
date: 2018-04-18 00:21:02
tags:
    - Spring
    - AOP
    - 原理
    - 设计模式
    - Java
categories:
    - Java
---
# Spring AOP实现相关
## 概要
Spring AOP实现中最重要的步骤分别为：如何织入、织入匹配、何时织入。
## 如何织入
即如何根据被代理对象的元数据来生成相应的代理对象，下面开始讲解如何实现织入的。
<!-- more -->
### TargetSource
被代理对象的元数据，包含被代理对象，被代理对象的class以及被代理对象的接口。
```java
/* 被代理对象的元数据*/
public class TargetSource {
    
    /* 被代理对象的class*/
    private Class<?> targetClass;
    
    /* 被代理对象的接口*/
    private Class<?>[] interfaces;

    /* 被代理的对象*/
    private Object target;
    
    /* 其他方法省略*/
}
```
### AdvisedSupport
代理相关的元数据，包含被代理对象的元数据，以及对被代理对象的方法增强时用到的方法拦截器和决定是否对某个方法增强的方法匹配器。
```java
/* 代理相关的元数据*/
public class AdvisedSupport {
    
    /* 被代理对象的元数据*/
    private TargetSource targetSource;
    
    /* 对被代理对象增强时用到的方法拦截器*/
    private MethodInterceptor methodInterceptor;
    
    /* 决定是否要对方法增强的方法匹配器*/
    private MethodMatcher methodMatcher;

    /* 其他方法省略*/
}
```
### MethodInterceptor
方法拦截器接口，这个是aopalliance包中的类，因为Spring AOP中只对方法做AOP以及拦截，所以该拦截器所做的事情其实就是对被代理对象的增强。
```java
public interface MethodInterceptor extends Interceptor {
    /* 这里需要传入一个方法调用器，方法调用器执行前后加入一些其他增强逻辑。*/
    Object invoke(MethodInvocation invocation) throws Throwable;
}
```
### AopProxy
获取AOP代理对象的关键入口接口，暴露了获取代理对象的方法，这也是织入的入口处。
```java
public interface AopProxy {
    /* 获取代理对象的规范*/
    Object getProxy();
}

/* 进一步定义了代理相关元数据*/
public abstract class AbstractAopProxy implements AopProxy {
    protected AdvisedSupport advised;
}
```
### ReflectiveMethodInvocation
基于反射来调用被代理对象的原始方法，相当于method.invoke(target,args)，这里只是对其做了封装，就是基于反射的原方法调用器。
```java
public class ReflectiveMethodInvocation implements MethodInvocation {

    protected Object target;

    protected Method method;

    protected Object[] arguments;
    
    /* 这里直接调用被代理对象原方法*/
    @Override
    public Object proceed() throws Throwable {
        return method.invoke(target, arguments);
    }
    
    /* 其他方法省略*/
}
```
### AopProxy的两种实现
#### JdkDynamicAopProxy
基于JDK动态代理的AOP代理生成类，实现了getProxy方法来返回代理对象，自身也实现了InvocationHandler，即由它来决定如何对被代理对象做增强。
```java
public class JdkDynamicAopProxy extends AbstractAopProxy implements InvocationHandler {
    @Override
    public Object getProxy() {
        /* 这里直接根据代理相关元数据，通过JDK动态代理来生成代理类*/
        return Proxy.newProxyInstance(getClass().getClassLoader(), advised.getTargetSource().getInterfaces(), this);
    }
    
    /* 如何对被代理对象做增强*/
    @Override
    public Object invoke(final Object proxy, final Method method, final Object[] args) throws Throwable {
        /* 从代理相关元数据中取出方法拦截器*/
        MethodInterceptor methodInterceptor = advised.getMethodInterceptor();
        /* 这里将代理相关元数据中的方法匹配器与被代理对象中的方法匹配*/
        if (advised.getMethodMatcher() != null
                && advised.getMethodMatcher().matches(method, advised.getTargetSource().getTarget().getClass())) {
            /*只有通过匹配才能对方法进行增强
            就是将原方法调用器传入方法拦截器中并执行拦截器
            拦截器会调用原方法并做一些指定的增强*/
            return methodInterceptor.invoke(new ReflectiveMethodInvocation(advised.getTargetSource().getTarget(),
                    method, args));
        } else {
            /* 如果无法匹配则说明该方法不应该被增强就调用原方法*/
            return method.invoke(advised.getTargetSource().getTarget(), args);
        }
    }
    
    /* 其他方法省略*/
}
```
### CglibAopProxy
基于cglib的AOP代理生成类，实现了AopProxy的getProxy方法，通过生成被代理对象的子类来创建代理对象。
```java
public class Cglib2AopProxy extends AbstractAopProxy {
    @Override
    public Object getProxy() {
        /* Enhancer用来创建子类代理对象 需要设置父类以及增强的回调函数即方法拦截器*/
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(advised.getTargetSource().getTargetClass());
        enhancer.setInterfaces(advised.getTargetSource().getInterfaces());
        enhancer.setCallback(new DynamicAdvisedInterceptor(advised));
        Object enhanced = enhancer.create();
        return enhanced;
    }
    
    /* 这个类其实是对MethodInterceptor的代理*/
    private static class DynamicAdvisedInterceptor implements MethodInterceptor {
        
        /* 代理相关元数据*/
        private AdvisedSupport advised;
        /* 应当对匹配的方法增强的方法拦截器*/
        private org.aopalliance.intercept.MethodInterceptor delegateMethodInterceptor;

        private DynamicAdvisedInterceptor(AdvisedSupport advised) {
            this.advised = advised;
            /* 从代理相关元数据中取出方法拦截器*/
            this.delegateMethodInterceptor = advised.getMethodInterceptor();
        }

        @Override
        public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
            /* 这里就是增强部分 如果被代理对象的方法匹配
            则用指定的方法拦截器调用原方法并进行增强
            否则仍然调用被代理对象的原方法*/
            if (advised.getMethodMatcher() == null
                    || advised.getMethodMatcher().matches(method, advised.getTargetSource().getTargetClass())) {
                return delegateMethodInterceptor.invoke(new CglibMethodInvocation(advised.getTargetSource().getTarget(), method, args, proxy));
            }
            return new CglibMethodInvocation(advised.getTargetSource().getTarget(), method, args, proxy).proceed();
        }
    }
    
    /* 这里相比ReflectiveMethodInvocation多封装了一个MethodProxy*/
    private static class CglibMethodInvocation extends ReflectiveMethodInvocation {
        
        private final MethodProxy methodProxy;
        
        /* 通过代理方法对象来调用父类的原方法*/
        @Override
        public Object proceed() throws Throwable {
            return this.methodProxy.invoke(this.target, this.arguments);
        }
    }
    
    /* 其他方法省略*/
}
```
### 织入实现过程
1. 织入具体是以JdkDynamic/CglibAopProxy的getProxy为入口开始的。
2. 会根据被代理对象的元数据中的Class对象，接口，方法匹配器和方法拦截器来创建代理类。
3. 首先会先设置代理对象的Class对象以及接口。 
4. 紧接着，JDK/Cglib动态代理分别会为代理对象设置的回调方法(invoke/intercept)中去判断每个方法是不是能被方法匹配器匹配。
5. 如果匹配才进一步在回调函数中调用带有原方法调用器(MethodInvocation)的方法拦截器(MethodInterceptor)(这个方法拦截器中带有增强的逻辑，它会在原方法调用器调用被代理对象的原方法前后加入增强逻辑)
6. 如果不匹配则利用原方法调用器直接调用被代理对象的原方法。
7. 为被代理对象相应的方法做增强之后将会把生成的代理对象返回，至此实现了对一个对象的织入。  


## 织入匹配
并不是所有对象和方法都被增强，织入谁即解决什么样的对象和方法应该被织入增强，下面开始讲解织入匹配的实现过程。

### 切点相关
#### ClassFilter
类匹配器接口，定义了匹配类的规范。
```java
public interface ClassFilter {
    boolean matches(Class targetClass);
}
```
#### MethodMatcher
方法匹配器接口，定义了匹配方法的规范
```java
public interface MethodMatcher {
    boolean matches(Method method, Class targetClass);
}
```
#### Pointcut
切点接口，它暴露了获取方法匹配器和类匹配器的入口，即用于匹配是否对某个对象、方法织入的关键接口。
```java
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();
}
```
#### AspectJExpressionPointcut
基于AspectJ表达式的切点类，它实现了Pointcut,ClassFilter和MethodMatcher接口，由其自身提供基于AspectJ表达式对对象和方法的匹配的功能。
```java
public class AspectJExpressionPointcut implements Pointcut, ClassFilter, MethodMatcher {
    /* AspectPointcut AspectJ表达式分析器 可以由他来解析AspectJ表达式*/
    private PointcutParser pointcutParser;
    /* 字符串AspectJ表达式*/
    private String expression;
    /* 由字符串表达式通过分析器来解析为切点表达式*/
    private PointcutExpression pointcutExpression;
    
    /* 由自身来提供类匹配和方法匹配功能*/
    @Override
    public ClassFilter getClassFilter() {
        return this;
    }

    @Override
    public MethodMatcher getMethodMatcher() {
        return this;
    }
    
    /* 通过AspectJ切点表达式来匹配类*/
    /* checkReadyToMatch在解析前构建AspectJ切点表达式*/
    @Override
    public boolean matches(Class targetClass) {
        checkReadyToMatch();
        return pointcutExpression.couldMatchJoinPointsInType(targetClass);
    }

    /* 通过AspectJ切点表达式来匹配方法*/
    @Override
    public boolean matches(Method method, Class targetClass) {
        checkReadyToMatch();
        ShadowMatch shadowMatch = pointcutExpression.matchesMethodExecution(method);
        if (shadowMatch.alwaysMatches()) {
            return true;
        } else if (shadowMatch.neverMatches()) {
            return false;
        }
        /* TODO:其他情况不判断了！见org.springframework.aop.aspectj.RuntimeTestWa*/lker
        return false;
    }
    
    /* 其他方法省略*/
}

```
### 通知器相关
#### Advisor
通知器接口，暴露了获取Advice(通知/增强)的入口。
```java
public interface Advisor {
    Advice getAdvice();
}
```
#### PointcutAdvisor
切点通知器接口，是切点和通知接口的合体，继承了Advisor接口，进一步定义了获取切点(Pointcut)的规范，也就是说可以通过它暴露的方法同时获得通知和切点。
```java
public interface PointcutAdvisor extends Advisor{
   Pointcut getPointcut();
}

```
### AspectJPointcutAdvisor
基于AspcetJ的切点通知器，定义了AspectJ切点和具体的Advice(通知/增强)，它是获取具体织入匹配功能的入口。这是一个配套的功能集合，对匹配的对象及方法做指定的增强。
```java
public class AspectJExpressionPointcutAdvisor implements PointcutAdvisor {

    /* 用AspectJ表达式来匹配对象及其方法*/
    private AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();

    /* 匹配的方法具体的增强*/
    private Advice advice;
    
    /* 其他方法省略*/
}
```
### 织入匹配实现过程
假设已经通过配置等手段向IOC容器注册了AspectJExpressionPointcutAd对象和其中的增强对象(即MethodInterceptor)。如下
```
<bean id="timeInterceptor" class="us.codecraft.tinyioc.aop.TimerInterceptor"></bean>

<bean id="aspectjAspect" class="us.codecraft.tinyioc.aop.AspectJExpressionPointcutAdvisor">
    <property name="advice" ref="timeInterceptor"></property>
    <property name="expression" value="execution(* us.codecraft.tinyioc.*.*(..))"></property>
</bean>

/* 这是一个时间拦截器，在原方法盗用的前后加入计时增强*/
public class TimerInterceptor implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        long time = System.nanoTime();
        System.out.println("Invocation of Method " + invocation.getMethod().getName() + " start!");
        Object proceed = invocation.proceed();
        System.out.println("Invocation of Method " + invocation.getMethod().getName() + " end! takes " + (System.nanoTime() - time)
                + " nanoseconds.");
        return proceed;
    }

}
```
1. 从配置中读入expression和advice，将其注入到AspectJExpressionPointcutAdvisor中去。其中对于expression会用其构建AspectJExpressionPointcut对象，而PointcutExpressionPointcut对象会从expression字符串中解析出AspectJExpression。
2. 维护一个PointcutAdvisor集合，对于每个bean，先用ClassFilter判断是否匹配AspectJ表达式中的类。如果匹配则将PointAdvisor中的MethodMatcher和的MethodInterceptor以及bean的元数据来构成代理相关数据交给JdkDynamic/CglibAopProxy来进行进一步的方法匹配和生成代理对象；如果不匹配则不对该bean进行织入。
3. 如果bean的类匹配，在JdkDynamic/CglibAopProxy会对bean的代理对象设置回调函数(invoke/interceptor)，在这个回调函数中会通过代理相关数据中的MethodMatcher对需要代理的对象的每一个方法进行匹配。
如果方法匹配了AspectJ表达式则会调用方法拦截器MethodIntercpetor对方法进行增强，否则将利用原方法调用器MethodInvocation来调用原方法。至此完成了对一个bean的织入匹配和织入。

## 何时织入
### 何时
已经能够织入并且能够织入匹配了，最终的目的还是要将以上两步融入到整个IOC容器的生命周期中，让程序实现自动为匹配表达式的bean及其中的方法做织入，生成代理对象并“偷天换日”，将原来的bean换成代理bean并且放回IOC容器中。我们自然会想到在IOC容器的某个时刻我们可以“趁虚而入”玩一把“狸猫换太子”的把戏，完成我们的目标。其实，这个时刻就是在IOC容器中**bean被实例化并完成初始化(属性注入)之后**进行的。
### 无侵入地扩展IOC容器
既然已经知道在何时做bean的织入了，那么之后就是如何实现无侵入地扩展IOC容器，让我们能在bean实例化并初始化之后对bean进行织入。
### 核心接口
#### BeanPostProcessor
bean后置处理器接口，暴露了在bean初始化前后的处理方法，这使得可以让实现该接口的类可以在bean初始化前后对bean做一些处理，比如我们这里想要将bean换成代理bean。
```java
public interface BeanPostProcessor {
    /* 在bean初始化前做处理的规范*/
    Object postProcessBeforeInitialization(Object bean, String beanName) throws Exception;
    
    /* 在bean初始化后做处理的规范*/
    Object postProcessAfterInitialization(Object bean, String beanName) throws Exception;
}
```
#### BeanFactoryAware
暴露beanFactory的接口，使得实现该类的接口可以设置BeanFactory来调用BeanFactory的一些方法，比如这里对bean做处理时需要通过BeanFactory来获取所有AspectJExpressionPointcutAdvisor来对bean做织入匹配和增强。
```java
public interface BeanFactoryAware {
   void setBeanFactory(BeanFactory beanFactory) throws Exception;
}
```
### 核心实现类
#### AspectJAwareAdvisorAutoProxyCreator
在IOC生命周期中自动创建代理对象的类，它实现了上述的BeanPostProcessor接口和BeanFactoryAware接口，能够从BeanFactory中获取到AspectJExpressionPointcutAdvisor来匹配哪些bean及方法做织入，并且能够在bean初始化后为匹配的bean生成代理对象，并玩“变换戏法”，把代理对象作为原来的bean的替代对象返回。
```java
public class AspectJAwareAdvisorAutoProxyCreator implements BeanPostProcessor, BeanFactoryAware {

    private AbstractBeanFactory beanFactory;
    
    /* 这里没有在bean初始化之前做处理 直接返回的原bean*/
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws Exception {
        return bean;
    }
    
    /* 在bean初始化之后生成代理对象并返回到原处去替换原bean*/
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws Exception {
        /* AspectJExpressionPointAdvisor和MethodInterceptor自然是不需要做代理的*/
        if (bean instanceof AspectJExpressionPointcutAdvisor) {
            return bean;
        }
        if (bean instanceof MethodInterceptor) {
            return bean;
        }
        
        /* 对于其他的普通bean 才考虑生成代理对象*/
        /**
        * 从BeanFacotry中取出并遍历所有AspectJExpressionPointcutAdvisor对象 
        * 依次匹配这个bean是否满足类匹配，如果满足才进一步根据产生代理对象
        * 这里根据bean的元数据，以及切点通知器中的通知(即增强)和切点中的方法匹配器来生成代理相关的元数据
        * 生成JdkDynamic/CglibAopProxy 来创建bean的代理对象 最后将其返回来替代原bean
        */
        
        /* 这里从BeanFactory的获取所有切点通知器bean */这是BeanFactory为了实现AOP所做的扩展 之后会细谈
        List<AspectJExpressionPointcutAdvisor> advisors = beanFactory
                .getBeansForType(AspectJExpressionPointcutAdvisor.class);
        for (AspectJExpressionPointcutAdvisor advisor : advisors) {
            if (advisor.getPointcut().getClassFilter().matches(bean.getClass())) {
                AdvisedSupport advisedSupport = new AdvisedSupport();
                advisedSupport.setMethodInterceptor((MethodInterceptor) advisor.getAdvice());
                advisedSupport.setMethodMatcher(advisor.getPointcut().getMethodMatcher());

                TargetSource targetSource = new TargetSource(bean, bean.getClass(), bean.getClass().getInterfaces());
                advisedSupport.setTargetSource(targetSource);
                /* 这里根据bean是否实现了接口来选择使用JDK/Cglib动态代理生成代理对象*/
                bean = new Cglib2AopProxy(advisedSupport).getProxy();
            }
        }
        return bean;
    }
    
    /* 通过实现BeanFactoryAware接口使得可以引用BeanFactory*/
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws Exception {
        this.beanFactory = (AbstractBeanFactory) beanFactory;
    }
}
```
### 对IOC容器进行扩展
已经有了自动生成代理对象的AspectJAwareAdvisorAutoProxyCreator，接下来就要把这个过程融入到IOC容器的生命周期中，BeanFactory和ApplicationContext必须扩展才能适应这新添加的处理功能。  
1. 首先是要保证实现了BeanPostProcessor的bean后置处理器要先于所有普通bean被实例化，并在IOC容器启动时就把这些对象存放到beanFactory维护的一个容器中以便在每个bean实例化后能够立即调用他们来处理bean。  
这个工作交给BeanFactory并不合适，因为它应该是高内聚的，只提供bean元数据的注册和创建/初始化bean以及提供bean的功能。所以自然就应该由BeanFacotry的代理类ApplicationContext来完成这一工作。  
这样的话BeanFactory需要提供实例化一种特定类型的所有bean来让ApplicationContext来获取所有实现了BeanPostProcessor接口的bean。  
另外是要提供一个注册BeanPostProcessor的接口来让Application将获取到的所有bean后置处理器加入到BeanFactory维护的bean后置处理器列表中。  
2. 其次是BeanFactory在为实现了BeanPostProcessor接口的bean注入属性时，还要将BeanFactory注入到这些bean中使得他们能从BeanFactory中获取所有AspectJExpressionPointcutAdvisor（这个过程中可能会装载该类型的bean，注入AspectJ匹配表达式和Advice增强）来对bean进行织入匹配和增强。 
3. 最后就可以在BeanFactory中在bean实例化后，遍历BeanPostProcessor集合在bean初始化之前，bean初始化之后做额外处理，之后再将更改后的bean返回。  
这里只看IOC容器中为了实现AOP扩展的部分 其他部分有删减
#### BeanFactory做出的扩展 
```java
public abstract class AbstractBeanFactory implements BeanFactory {
    /* 这里添加了一个BeanPostProcessor列表*/
    /* 用于存放所有在bean初始化前后对bean做处理的后置处理器*/
    private List<BeanPostProcessor> beanPostProcessors = new ArrayList<BeanPostProcessor>();

    @Override
    public Object getBean(String name) throws Exception {
        BeanDefinition beanDefinition = beanDefinitionMap.get(name);
        if (beanDefinition == null) {
            throw new IllegalArgumentException("No bean named " + name + " is defined");
        }
        Object bean = beanDefinition.getBean();
        if (bean == null) {
            bean = doCreateBean(beanDefinition);
            /* 实例化bean之后将调用这个方法来对bean做处理*/
            bean = initializeBean(bean, name);
            beanDefinition.setBean(bean);
        }
        return bean;
    }

    protected Object initializeBean(Object bean, String name) throws Exception {
        /* 遍历bean后置处理器列表来对bean做初始化前置处理*/
        for (BeanPostProcessor beanPostProcessor : beanPostProcessors) {
            bean = beanPostProcessor.postProcessBeforeInitialization(bean, name);
        }

        /* TODO:call initialize method*/
        
        /* 遍历bean后置处理器列表来对bean做初始化后置处理*/
        for (BeanPostProcessor beanPostProcessor : beanPostProcessors) {
            bean = beanPostProcessor.postProcessAfterInitialization(bean, name);
        }
        
        /* 将处理后的bean返回替换原bean*/
        return bean;
    }
    
    /* 这个是注册BeanPostProcessor的接口 提供给ApplciationContext来注册*/
    public void addBeanPostProcessor(BeanPostProcessor beanPostProcessor) throws Exception {
        this.beanPostProcessors.add(beanPostProcessor);
    }
    
    /* 这里是将BeanFactory中特定类型的bean全部返回*/
    /* 不仅是给ApplciationContext*/
    public List getBeansForType(Class type) throws Exception {
        List beans = new ArrayList<Object>();
        for (String beanDefinitionName : beanDefinitionNames) {
            if (type.isAssignableFrom(beanDefinitionMap.get(beanDefinitionName).getBeanClass())) {
                beans.add(getBean(beanDefinitionName));
            }
        }
        return beans;
    }

}
```
#### ApplicationContext做出的修改
```java
public abstract class AbstractApplicationContext implements ApplicationContext {
    
    public void refresh() throws Exception {
        loadBeanDefinitions(beanFactory);
        /* 在加载完bean元数据之后立即向BeanFactory中注册所有BeanPostProcessor对象*/
        registerBeanPostProcessors(beanFactory);
        onRefresh();
    }
       
    /* 从beanFactory中获取素有BeanPostProcessor并将其注册到BeanFacotry中*/
    protected void registerBeanPostProcessors(AbstractBeanFactory beanFactory) throws Exception {
        List beanPostProcessors = beanFactory.getBeansForType(BeanPostProcessor.class);
        for (Object beanPostProcessor : beanPostProcessors) {
            beanFactory.addBeanPostProcessor((BeanPostProcessor) beanPostProcessor);
        }
    }
```
### 整个Aop实现过程总结
下面就自顶向下地对整个AOP实现过程进行分析
1. IOC容器启动时会在ApplicationContext加载完bean元数据并向BeanFacotry注册之后会从BeanFactory中获取所有实现了BeanPostProcessor接口的对象（这个过程会实例化这些bean并将BeanFactory对象注入到其属性中）并将其注册到BeanFactory维护的BeanPostProcessor列表中，保证BeanPostProcessor在所有bean装载之前实例化。
2. 之后BeanFactory对于每个第一次获取的bean，在实例化和初始化bean之后，就会遍历自己维护的BeanPostProcessor列表，对bean进行处理，并将处理完的bean返回做后续操作 。下面主要讨论AspectJAwareAdvisorAutoProxyCreator的处理过程。
3. AutoProxyCreator对象会通过自身引用的BeanFactory去获取所有对象然后针对每个AspectJExpressionPointcutAdvisor对象对bean进行判断之后决定是否要织入。下面主要讨论一个AspectJExpressionPointcutAdvisor对bean的判断与织入过程。
4. 先是从AspectJExpressionPointcutAdvisor对象中取出AspectJExpressionPointcut，再利用其中的ClassFilter对象对bean做类匹配。只有匹配的bean才能进行织入，如果不匹配则返回原bean。接下来对类匹配成功的bean进行讨论。
5. 对于类匹配成功的bean，根据bean来创建被代理对象的元数据，再根据AspectJExpressionPointcut中的Advice（因为Spring AOP只对方法增强，所以这里就是MethodInterceptor）以及MethodMatcher构建代理元数据。
6. 这之后会根据代理元数据对象来创建JdkDynamic/CglibAopProxy对象（这由bean是否实现了接口来决定，如果实现了接口就是JdkDynamicAopProxy；如果没有实现接口则是CglibAopProxy）来获取代理对象。下面对代理对象的创建进行讨论，JDK/Cglib动态代理虽然有差异，但我会用统一的方式来叙述。
7. 首先是设置bean元数据信息（bean的Class对象，实现的接口，类加载器等），随后就是回调函数(JDK动态代理是InvocationHandler的invoke，Cglib动态代理是Cglib包下的MethodInterceptor的intercept)。
8. 在这个回调函数中，会通过代理元数据中的MethodMatcher对每个方法进行方法匹配。如果这个方法匹配，则会在回调函数中调用代理元数据中的Advice即MethodInterceptor对象的invoke来对原方法方法进行增强（会向其中传入原方法调用器MethodInvocation对象（如果是JDK动态代理是ReflectiveMethodInvocation（它会基于反射调用原bean的原方法），如果是Cglib动态代理是CglibMethodInvocation（它会通过方法代理Method来调用原bean的原方法）））。
9. 至此实现了AOP的基本功能。
