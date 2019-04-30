---
title: tiny-spring-ioc-学习总结
date: 2018-04-11 01:31:49
tags:
    - Spring
    - IOC
    - 原理
    - 设计模式
    - Java
categories:
    - Java
---
# IOC容器实现相关
下面先简单介绍一下基本的类和接口
## 数据源相关
继承与实现关系  
Resource->UrlResource
<!-- more -->
### Resource
定义了从数据源获取输入流的规范，用于从文件中获取输入流。
```java
public interface Resource {
    InputStream getInputStream() throws IOException;
}
```
### UrlResource
实现了Resource接口，用于从统一资源定位符中获取输入流。
```java
public class UrlResource implements Resource {

    private final URL url;

    public UrlResource(URL url) {
        this.url = url;
    }

    @Override
    public InputStream getInputStream() throws IOException{
        /* 从统一资源定位符中获取资源连接 */
        URLConnection urlConnection = url.openConnection();
        /* 打开资源连接 */
        urlConnection.connect();
        /* 从资源连接中获取输入流 */
        return urlConnection.getInputStream();
    }
}
```
### ResourceLoader
用于从给定路径中获取数据源并封装为统一定位资源。
```java
public class ResourceLoader {
    public Resource getResource(String location){
        /* 根据资源文件中的指定路径创建统一资源定位符 */
        URL resource = this.getClass().getClassLoader().getResource(location);
        /* 将资源定位符封装为统一定位资源 */
        return new UrlResource(resource);
    }
}
```
## Bean元数据相关
### BeanDefiniton
bean的元数据，包含bean的实例、Class名称、Class对象、属性等信息。用于之后创建bean实例，对bean的属性进行注入以及存放bean实例。
```java
public class BeanDefinition {

    /* bean实例 */
    private Object bean;
    /* bean的Class对象 */
    private Class beanClass;
    /* bean的Class名称 */
    private String beanClassName;
    /* bean的属性 */
    private PropertyValues propertyValues = new PropertyValues();
    
    /*在设置bean的Class名称时可以直接通过bean的Class名称获取到它的Class对象 */
    public void setBeanClassName(String beanClassName) {
        this.beanClassName = beanClassName;
        try {
            this.beanClass = Class.forName(beanClassName);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    /* 其他方法省略 */
}
```
### PropertyValue与PropertyValues
bean的属性与属性列表，以键值对name-value的形式存储属性的名称与值，用于bean属性的注入。  
值得一提的是value可以是另一个bean，此时value为beanReference类型来表示是bean的引用。
```java
public class PropertyValue {
    
    /* 属性名称 */
    private final String name;
    /* 属性值 */
    private final Object value;
    
    /* 其他方法省略 */
}

/* 这里用一个类对属性列表进行封装 可以增加对属性列表的预处理 */
public class PropertyValues {
    
    /* 属性列表 */
    private final List<PropertyValue> propertyValueList = new ArrayList<PropertyValue>();

    public void addPropertyValue(PropertyValue pv) {
        /* TODO:这里可以对于重复propertyName进行判断，直接用list没法做到 */
        this.propertyValueList.add(pv);
    }
    
    /* 其他方法省略 */

}
```
### BeanReference
bean的引用，包含bean的名称与实例，用于对bean中的bean进行注入。
```java
public class BeanReference {
    /* bean的名称 */
    private String name;
    /* bean的实例 */
    private Object bean;
    
    /* 其他方法省略 */
}
```

## Bean元数据读取相关
继承与实现关系  
BeanDefinitionReader->AbstractDefinitionReader->XmlBeanDefinitonReader
### BeanDefinitionReader
bean元数据读取接口，定义了从指定路径的文件中中加载bean元数据的规范。
```java
public interface BeanDefinitionReader {
    void loadBeanDefinitions(String location) throws Exception;
}
```
### AbstractBeanDefinitonReader
抽象bean元数据读取类，实现了BeanDefinitonReader接口。  
定义了ResourceLoader属性负责得到统一定义资源和registry属性(Map类型)来存放已经读取的bean元数据。
```java
public abstract class AbstractBeanDefinitionReader implements BeanDefinitionReader {
    /* 存放已注册的bean元数据 */
    private Map<String,BeanDefinition> registry;
    /* 用于获取统一定位资源 */
    private ResourceLoader resourceLoader;

    /* 其他方法省略 */
}
```
### XmlBeanDefinitonReader
继承了AbstractBeanDefinitonReader，实现了从xml配置文件中加载bean元数据的方法。
```java
public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {
    
    /* 调用父类构造器传入资源加载器 */
    public XmlBeanDefinitionReader(ResourceLoader resourceLoader) {
        super(resourceLoader);
    }
    
    @Override
    public void loadBeanDefinitions(String location) throws Exception {
        /* 用资源加载器从指定路径中的xml文件加载资源并获取输入流 */
        InputStream inputStream = getResourceLoader().getResource(location).getInputStream();
        /* 从xml文件的输入流中解析xml文件得到bean的元数据
         *(即解析bean节点中的name节点、class节点和property节点
         * 以及property节点中name节点、value节点和ref节点 
         * 特殊的对于ref节点创建的属性值为BeanReference便于之后对bean的bean注入)
         *并将创建的所有BeanDefiniton对象将其暂时放入register中
         */
        doLoadBeanDefinitions(inputStream);
    }
    
    /*对xml文件的解析操作方法省略 */
}
```
## 最基本的IOC容器
继承与实现关系  
BeanFactory->AbstractBeanFactoryr
### BeanFactory
bean工厂接口，定义了从IOC容器中获取bean的规范。
```java
public interface BeanFactory {
    Object getBean(String name) throws Exception;
}
```
### AbstractBeanFactory
抽象bean工厂，实现了BeanFacoty接口，实现了BeanFactory接口的getBean方法。  
其中定义了一个Map属性来存放bean容器中所有bean的bean的元数据，定义了一个doCreateBeanDefintion模板方法留给子类实现，用于创建和初始化化bean。  
另外一个重要的一点是暴露了bean元数据的注册接口，可以在BeanDefinitonReader读取了元数据之后将bean元数据注册到Map中。
```java
public abstract class AbstractBeanFactory implements BeanFactory {
    
    /* 用于存放bean的元数据 */
    private Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>();
    
    /* 用于存放bean的名称 */
    private final List<String> beanDefinitionNames = new ArrayList<String>();
    
    /* 实现了bean工厂接口的获取bean的方法 */
    @Override
    public Object getBean(String name) throws Exception {
        BeanDefinition beanDefinition = beanDefinitionMap.get(name);
        if (beanDefinition == null) {
            throw new IllegalArgumentException("No bean named " + name + " is defined");
        }
        /** 
         * 这里实现了bean的懒加载  
         * 如果已经IOC容器中已经有bean则直接获取  
         * 否则才创建并初始化bean
         **/
        Object bean = beanDefinition.getBean();
        if (bean == null) {
            bean = doCreateBean(beanDefinition);
        }
        return bean;
    }
    
    /* 暴露给外界用于注册bean元数据 */
    public void registerBeanDefinition(String name, BeanDefinition beanDefinition) throws Exception {
        beanDefinitionMap.put(name, beanDefinition);
        beanDefinitionNames.add(name);
    }
    
    /* 如果不想懒加载 该方法实现了bean容器的预加载  
     * 预先把容器中注册过的所有bean创建并初始化 
     */
    public void preInstantiateSingletons() throws Exception {
        for (Iterator it = this.beanDefinitionNames.iterator(); it.hasNext();) {
            String beanName = (String) it.next();
            getBean(beanName);
        }
    }
    
    /* 模板方法 用于创建和初始化bean 留给不同的子类实现 */
    protected abstract Object doCreateBean(BeanDefinition beanDefinition) throws Exception;

}
```
### AutoCapableBeanFactory
自动装配的bean工厂，继承自AbstractBeanFactory，实现了doCreateBean来实例化bean并初始化bean(为bean注入属性)。
```java
public class AutowireCapableBeanFactory extends AbstractBeanFactory {

    @Override
    protected Object doCreateBean(BeanDefinition beanDefinition) throws Exception {
        /* 先实例化bean */
        Object bean = createBeanInstance(beanDefinition);
        /* 将bean置入相应的元数据中 以后通过map访问复用 */
        beanDefinition.setBean(bean);
        /* 再为bean初始化 注入属性等 */
        applyPropertyValues(bean, beanDefinition);
        return bean;
    }
    
    /* 从bean的元数据中得到Class来实例化一个bean */
    protected Object createBeanInstance(BeanDefinition beanDefinition) throws Exception {
        return beanDefinition.getBeanClass().newInstance();
    }
    
    /* 从bean的元数据中得到该bean的所有属性 通过反射来设置bean属性 */
    protected void applyPropertyValues(Object bean, BeanDefinition mbd) throws Exception {
        for (PropertyValue propertyValue : mbd.getPropertyValues().getPropertyValues()) {
            Field declaredField = bean.getClass().getDeclaredField(propertyValue.getName());
            declaredField.setAccessible(true);
            Object value = propertyValue.getValue();
            /* 如果属性值是另一个bean 则从bean容器中获取该bean再设置到bean中 */
            if (value instanceof BeanReference) {
                BeanReference beanReference = (BeanReference) value;
                value = getBean(beanReference.getName());
            }
            declaredField.set(bean, value);
        }
    }
}
```
## 增强了的IOC容器  
它是应用中直接使用的IOC容器  
继承和实现关系  
BeanFactory->ApplicationContext->AbstractApplicationContext->ClassPathXmlApplicationContext
### ApplicationContext
应用上下文接口，继承了BeanFactory接口，具有获取bean的能力。
```java
public interface ApplicationContext extends BeanFactory {
}
```
### AbstractApplicationContext
抽象应用上下文，实现了ApplicationContext接口。  
ApplicationContext对BeanFacoty增强，可以认为是对BeanFactory代理，定义了BeanFacoty属性，直接利用它来注册bean的元数据和获取bean。  
另外一个是定义了模板方法refresh()留给不同子类实现对bean元数据的读取和注册。
```java
public abstract class AbstractApplicationContext implements ApplicationContext {
    /* 对基本的IOC容器 BeanFactory进行增强 */
    protected AbstractBeanFactory beanFactory;
    
    /* 留给不同子类实现bean元数据的读取和注册 */
    public abstract void refresh() throws Exception{
    }
    
    /* 直接调用基本IOC容器来获取bean */
    @Override
    public Object getBean(String name) throws Exception {
        return beanFactory.getBean(name);
    }
    
    /*其他方法省略 */
}
```
### ClassPathXmlApplicationContext
根据类路径中的Xml配置文件构建的应用上下文，继承自AbstractApplicationContext。  
定义了一个configLocation属性用于定位xml文件在类路径中的地址，实现了refresh方法来从xml文件中读取bean元数据与注册。
```java
public class ClassPathXmlApplicationContext extends AbstractApplicationContext {
    /* xml文件在类路径中的地址 */
    private String configLocation;
    
    /* 这里直接使用自动装配的BeanFactory作为基础的IOC容器 */
    public ClassPathXmlApplicationContext(String configLocation) throws Exception {
        this(configLocation, new AutowireCapableBeanFactory());
    }
    
    /* 这里在初始化ClassPathXmlApplicationContext  */
    /* 后会直接读取并注册所有bean元数据 */
    public ClassPathXmlApplicationContext(String configLocation, AbstractBeanFactory beanFactory) throws Exception {
        super(beanFactory);
        this.configLocation = configLocation;
        refresh();
    }
    
   /**
    * 这部分是Application对BeanFacoty增强的主要部分
    * 将BeanFactory与BeanDefinitonReader结合起来工作
    * 这里会初始化一个XmlBeanDefinitonReader来从xml文件中读取bean的元数据
    * 之后再将读取到的所有bean数据注册到BeanFacoty中
    **/
    @Override
    public void refresh() throws Exception {
        XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(new ResourceLoader());
        xmlBeanDefinitionReader.loadBeanDefinitions(configLocation);
        for (Map.Entry<String, BeanDefinition> beanDefinitionEntry : xmlBeanDefinitionReader.getRegistry().entrySet()) {
            beanFactory.registerBeanDefinition(beanDefinitionEntry.getKey(), beanDefinitionEntry.getValue());
        }
    }

}
```
## IOC容器实现过程  
为了方便描述，以下的分析全部用接口来描述相应的组件
1. 在指定了配置路径并初始化ApplicationContext后，将初始化BeanDefinitionReader来从相应文件中解析并读取bean的元数据。  
2. 这之后将通过BeanFactory暴露的接口将所有读取到的bean的元数据注册到BeanFactory中的bean的元数据Map中存储（注意这个时候还没有实例化任何bean）。
3. 当通过ApplicationContext的getBean尝试获取bean时，ApplicationContext将直接使用BeanFactory来获取bean，将会先检查bean已经在容器则直接获取并返回，否则将根据bean的元数据实例化和初始化bean。
4. 当BeanFactory中没有相应的bean时，则会根据bean的元数据先实例化bean，这之后再根据元数据通过反射来为bean注入属性。当bean的属性为bean时，则会通过调用自身的getBean()来获取相应的bean之后再注入到属性中。（这里可以看到通过BeanFactoy通过懒加载和getBean避免了bean之间的循环依赖）
5. 最后将会把初始化好的bean放置到相应bean的元数据中bean实例属性，以便之后再次获取时的复用，这之后再将bean返回给调用处就完成了一次bean的获取。