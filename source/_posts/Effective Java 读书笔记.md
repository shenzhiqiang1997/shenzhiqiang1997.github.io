---
layout: effective
title: Effective Java 读书笔记
date: 2018-03-06 16:35:44
categories:
    - Java
tags:
    - Java
    - 进阶
---
### 用静态工厂方法代替构造器
* 有名字的静态工厂方法比构造器对返回的对象进行描述更确切。
* 在重载构造方法时，如果有两个构造方法原型冲突，那么可以用静态工厂方法来区别。
* 静态工厂方法可以重复利用创建的实例或者确保每次产生不同的实例或者不创建实例。
* 静态工厂方法可以返回返回类型的子类实例，或返回的类在实现静态工厂方法时根本不存在。例如服务提供者框架。
<!-- more -->
    ```java
    //服务提供者框架
    /*
        不同服务提供者返回不同服务实例，但服务提供者和服务类都未实现，
        根据不同实现返回不同的服务实例，扩展性较强。
    */
    import java.util.HashMap;
    import java.util.Map;
    
    //服务接口
    interface Service{
        //服务方法
        void serve();
    }
    
    //服务提供者接口
    interface Provider{
        Service newService();
    }
    
    public class Services{
        //构造方法私有化，不创建实例
        private Services(){};
    
        //用于存放服务提供者实例
        private static Map<String,Provider> providers=new HashMap<>();
    
        //默认的服务提供者
        public static final String DEFAULT_PROVIDER_NAME="default";
    
        //注册服务提供者API
        //注册服务提供者
        public static void registerProvider(String name,Provider p){
            providers.put(name,p);
        }
    
        //注册默认服务提供者
        public static void registerDefaultProvider(Provider p){
            registerProvider(DEFAULT_PROVIDER_NAME,p);
        }
    
        //访问服务API
        //根据服务提供者名称访问服务
        public static Service newInstance(String name){
          Provider p=providers.get(name);
          if (p==null){
              throw new IllegalArgumentException("没有服务提供者："+name);
          }
          return p.newService();
        }
    
        //访问默认服务提供者提供的服务
        public static Service newInstance(){
            return newInstance(DEFAULT_PROVIDER_NAME);
        }
    
        public static void main(String[] args) {
            //一个服务提供者具体实现
            Services.registerProvider("waiter", new Provider() {
                @Override
                public Service newService() {
                    return new WaiterService();
                }
            });
    
            //获得一个服务实例
            Service waiter=Services.newInstance("waiter");
    
            //服务者开始服务
            waiter.serve();
    
        }
    }
    
    //一个服务具体实现
    class WaiterService implements Service{
        @Override
        public void serve() {
            System.out.println("waiter serve...");
        }
    }
    
    ```
* 对于带有泛型的类，因为泛型是声明在类上的，所以实例化必须指明泛型。但当通过静态工厂方法创建实例，则可以直接返回类上的泛型类型对应的实例，使编译器根据左值自动类型推导完成实例化，从而只需在创建实例处只在左值类型指明一次泛型即可。构造函数和方法调用在最新的JDK中已经可以进行泛型自动推导。
* 若构造方法全为私有则其子类因无法调用其父类的构造方法而无法创建子类，则静态工厂方法则无法返回子类实例。
* 对静态工厂方法可以通过注释或者遵守命名规范可以使其更明确地标识出来。

### 用构建器（Builder）来替代多个构造器
* 当类构造器或静态工厂方法中有多个参数且许多参数是可选的情况下，使用Builder更易阅读和编写并且安全，需要在"setter"或build中加入约束检查来对参数检查。
  ```java
    import java.util.Date;
    
    class Student{
        //不可变量
        private final String studentId;
        private final Date birthDay;
    
        //可变量
        private String name;
        private double height;
        private int age;
    
        //构建器
        public static class Builder{
            //不可变量
            private final String studentId;
            private final Date birthDay;
    
            //可变量
            private String name="无名氏";
            private double height=0;
            private int age=0;
    
            public Builder(String studentId,Date birthDay){
                this.studentId=studentId;
                this.birthDay=birthDay;
            }
    
            //"setter" 对构建器属性赋值 返回Builder本身 以便连续调用方法
            public Builder name(String name){
                this.name=name;
                return this;
            }
    
            public Builder height(double height){
                this.height=height;
                return this;
            }
    
            public Builder age(int age){
                this.age=age;
                return this;
            }
    
            //用于构建实例 避免一致性问题
            public Student build(){
                return new Student(this);
            }
    
        }
    
        //构造函数 只能由构建器构建实例
        private Student(Builder builder){
            this.studentId=builder.studentId;
            this.birthDay=builder.birthDay;
            this.name=builder.name;
            this.height=builder.age;
            this.age=builder.age;
        }
    
    }
    
    public class BuilderTest {
        public static void main(String[] args) {
            Student student=new Student.Builder("20160101",new Date())
                    .name("构造器")
                    .height(180)
                    .age(20)
                    .build();
    
        }
    }
  ```
### 用私有构造器或者枚举类型实现Singleton
* 实现Singleton的三种方法
    1. 饿汉式
    ```java
    class Singleton{
        public static final Singleton s=new Singleton();
        private Singleton(){}
        public void method(){
            //Do Something
        }
    }
    ```
    2. 懒汉式
    ```java
    class Singleton{
        private static Singleton s;
        private Singleton(){}
        public static Singleton getInstance(){
            if (s==null){
                s=new Singleton();
            }
            return s;
        }
        public void method(){
            //Do Something
        }
    }
    ```
    3. 用包含单个元素的枚举类实现。
    ```java
    public enum Singleton {
        SINGLETON;
        public void method(){
            //Do Something
        }
    }
    ```
* 反射对单例的破坏<br>
当使用了AccessibleObject.setAccessible之后<br> 可以通过反射从外部调用构造函数获取实例从而会破坏单例特性<br>
为了防止这样的事情发生就需要在第二次调用构造函数的时候抛出异常
    ```java
    import java.lang.reflect.AccessibleObject;
    import java.lang.reflect.Constructor;
    
    class Singleton {
        private static int instanceNum=0;
        public static final Singleton s=new Singleton();
        private Singleton(){
            instanceNum++;
            if (instanceNum>=2){
                throw new AssertionError();
            }
        }
    }
    public class SingletonTest{
        public static void main(String[] args) {
            Singleton s1=Singleton.s;
    
            Class c=Singleton.class;
            try {
                Constructor constructor=c.getDeclaredConstructor();
                AccessibleObject.setAccessible(new Constructor[]{constructor},true);
                Singleton s2=(Singleton) constructor.newInstance();
                System.out.println(s1==s2);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    ```
* 序列化对单例的破坏<br>
对单例对象序列化 在反序列化时会通过反射生成新的实例继而破坏单例特性<br>
可以通过实现readResolve方法来在反序列化时调用从而指定生成对象的策略<br>
就能够避免在反序列化时生成新的实例
    ```java
    import java.io.*;
    
    class Singleton implements Serializable {
        private transient static int instanceNum=0;
        public transient static final Singleton s=new Singleton();
        private Singleton(){
            instanceNum++;
            if (instanceNum>=2){
                throw new AssertionError();
            }
        }
        private Object readResolve(){
            return s;
        }
    }
    public class SingletonTest{
        public static void main(String[] args) {
            ObjectOutputStream oos=null;
            ObjectInputStream ois=null;
            FileOutputStream fos=null;
            FileInputStream fis=null;
            try {
                fos=new FileOutputStream("singleton");
                oos=new ObjectOutputStream(fos);
                oos.writeObject(Singleton.s);
    
                fis=new FileInputStream("singleton");
                ois=new ObjectInputStream(fis);
                Singleton s=(Singleton) ois.readObject();
    
                System.out.println(s==Singleton.s);
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try{
                    if (ois!=null){
                        ois.close();
                    }
                    if (oos!=null){
                        oos.close();
                    }
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }
    }
    ```

### 通过私有构造器来实现不可实例化类
* 对于工具类或者只含静态属性和方法的类不希望被实例化 <br>可以通过将构造方法私有化来实现
* 为了防止从内部调用构造函数或通过反射来实例化
<br>可以通过在构造方法中抛出异常来阻止调用
    ```java
    public class StaticClass {
        private StaticClass(){
            throw new AssertionError();
        }
    }
    ```
### 避免创建不必要的对象
* 一个极端反面例子
    ```java
        String s=new String("string");
    ```
    "string" 本身就创建了一个实例<br>
    再调用new String("string")又一次创建了实例<br>
    一共创建了两个实例 创建了不必要的实例<br><br>
    改进后的版本：字符串字面常量的实例可以反复使用
    ```java
        String s="string";
    ```
* 可以使用静态工厂方法控制实例产生而不是使用构造器，构造器每次被调用都会创建新的实例。
* 对于重复使用的不被修改的对象，可以将其设置为类的静态成员，并在静态代码块中只进行一次初始化，完成实例的创建。
    ```java
    import java.util.Calendar;
    import java.util.Date;
    import java.util.TimeZone;
    
    class Person{
        //生日
        private final Date birthDay;
        //生日范围
        private static final Date BEGIN_DATE;
        private static final Date END_DATE;
    
        //用于获取东八区的生日
        private static final Calendar calendar;
    
        static {
            //设置时区为东八区
            calendar=Calendar.
                    getInstance(TimeZone.getTimeZone("UTC+8"));
    
            //获取比较的开始日期
            calendar.set(1990,Calendar.JANUARY,1);
            BEGIN_DATE=calendar.getTime();
    
            //获取比较的结束日期
            calendar.set(1999,Calendar.DECEMBER,31);
            END_DATE=calendar.getTime();
        }
    
        //初始化东八区的生日
        public Person(int year,int month,int day){
            calendar.set(year,month,day);
            birthDay=calendar.getTime();
        }
    
        //判断是否是90后
        public boolean isPostNinety(){
            return birthDay.compareTo(BEGIN_DATE)>=0 &&
                    birthDay.compareTo(END_DATE)<=0;
        }
    }
    ```
* 装箱将创建新的对象会增大开销 所以要避免自动装箱 最好使用基本类型。
* 小对象的创建和回收是十分廉价的，而对于重量级的对象，需要用对象池来维护对象的创建和回收，比如数据库连接池和Socket连接池。
* 不要自己维护轻量级对象池，不仅代码复杂而且会增加内存占用，直接靠JVM高度优化的垃圾回收来管理性能就很可观。