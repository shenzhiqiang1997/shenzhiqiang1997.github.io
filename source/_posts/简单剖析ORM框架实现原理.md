---
title: 简单剖析ORM框架实现原理
date: 2018-03-04 15:33:05
categories:
    - Java
tags: 
    - Java 
    - 反射
    - 注解
    - ORM
---
## 概述<br>
首先要介绍注解和反射这两个重要的概念
### 注解<br>
* 这里用到注解的方式进行实现 当然用xml配置文件也可以 就需要对xml进行解析
* 自定义注解需用元注解进行标记
* 一般元注解指明<br>
    1. 自定义注解的标记的目标（类、属性、方法等）<br>
    2. 自定义注解的作用范围 （SOURCE、CLASS、RUNTIME）<br>
    3. 自定义注解内的属性一般就是该注解携带的一些信息<br> 
    4. 来表明一些被注解对象的性质
<!-- more -->
* 让我们看看这两个注解
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Table {
        String value();
    }
    
    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Column {
        String  value();
    }
    ```
* 为了实现ORM工具 我们需要注解为**运行时**注解 在运行时获取注解信息
### 反射<br>
* 每个类都有其对应的Class<br>
* 它可以被称为类类型 一个类的Class对象包含了一个类的信息 包括方法 属性 注解 构造函数…<br>
* 甚至是可以通过它来调用类的一些方法（就算是private的也可以）和构造函数<br>
* 总的来说 只要你知道类里面的情况是什么样的 你就能为所欲为的访问这个类的一切<br>
* 注解和反射是Java中框架的基石 我这里只是在我实现该工具的角度上去草草带过<br>
---
## 实体类<br>
看下面这个实体类 已经被我用自定义的注解标注好了
    ```java
    @Table("student")
    public class Student {
        @Column("student_id")
        private String studentId;
        @Column("name")
        private String name;
        @Column("sex")
        private Integer sex;
        @Column("major")
        private String major;
        @Column("contact_way")
        private String contactWay;
    
        public String getStudentId() {
            return studentId;
        }
    
        public void setStudentId(String studentId) {
            this.studentId = studentId;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public Integer getSex() {
            return sex;
        }
    
        public void setSex(Integer sex) {
            this.sex = sex;
        }
    
        public String getMajor() {
            return major;
        }
    
        public void setMajor(String major) {
            this.major = major;
        }
    
        public String getContactWay() {
            return contactWay;
        }
    
        public void setContactWay(String contactWay) {
            this.contactWay = contactWay;
        }
    
        @Override
        public String toString() {
            return "Student{" +
                    "studentId=" + studentId +
                    ", name='" + name + '\'' +
                    ", sex=" + sex +
                    ", major='" + major + '\'' +
                    ", contactWay='" + contactWay + '\'' +
                    '}';
        }
    }
    ```
这里的getter setter是必不可少的<br>
因为实现ORM功能时需要通过反射调用实体类的getter和setter方法
---
## 实现一个搜索小工具
```java
public class DBUtil {
    //数据库连接参数
    private static final String URL="jdbc:mysql://localhost:3306/";
    private static final String DRIVER="com.mysql.jdbc.Driver";
    private static final String USER="root";
    private static final String PASSWORD="1234";

    private Connection connection;
    private PreparedStatement ps;
    private ResultSet rs;

    //加载数据库驱动
    static {
        try {
            Class.forName(DRIVER);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    //根据传入的数据库获取相应的连接
    public DBUtil(String databaseName){
        try {
            connection=DriverManager.getConnection(URL+databaseName,USER,PASSWORD);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public <T> List<T> query(T entity){
        //获取类类型 其中包括该实体类的信息 包括属性 方法 构造函数
        Class entityClass=entity.getClass();

        //用于盛放搜索结果
        List<T> results=new ArrayList<>();

        //如果用Table标记过则表明映射为数据库表
        if (entityClass.isAnnotationPresent(Table.class)){
            //获取注释
            Table tableAnnotation= (Table) entityClass.getAnnotation(Table.class);

            //获取注释value即对应表名
            String tableName=tableAnnotation.value();

            //拼接sql语句 SELECT * FROM tableName WHERE 1=1
            StringBuffer sql=new StringBuffer("SELECT * FROM ").append(tableName).append(" WHERE 1=1");

            //用于放搜索条件的顺序和搜索值
            Map<Integer,Object> searchColumnParams=new HashMap<>();
            Integer paramIndex=1;

            //用于存放搜索结果的顺序和用于绑定返回数据的setter方法名以及参数类型
            List<Map<String,Object>> resultColumnParams=new ArrayList<>();

            //获取该实体类的所有自己声明的属性
            Field[] fields=entityClass.getDeclaredFields();
            for (Field field:fields) {
                //如果用Column标记过则说明要映射为表中的一列
                if (field.isAnnotationPresent(Column.class)){
                    //存放映射的每列的setter方法和参数类型
                    Map<String,Object> resultColumnMap=new HashMap<>();
                    resultColumnMap.put("setter","set"+field.getName().substring(0,1).toUpperCase()+field.getName().substring(1));
                    resultColumnMap.put("paramClass",field.getType());
                    resultColumnParams.add(resultColumnMap);

                    //获取到要搜索的列名
                    Column columnAnnotation=field.getAnnotation(Column.class);
                    String columnName=columnAnnotation.value();
                    String fieldName=field.getName();


                    try {
                        //调用相应的getter 获取到搜索值
                        Method method=entityClass.getDeclaredMethod("get"+fieldName.substring(0,1).toUpperCase()+fieldName.substring(1));
                        Object searchColumnValue=method.invoke(entity);

                        //如果获得到搜索值说明要该列有搜索条件
                        if (searchColumnValue!=null){
                            //拼接相应的SQL语句 AND 列名=/LIKE ?
                            if (searchColumnValue instanceof String){
                                sql.append(" AND "+columnName+" LIKE ?");
                                searchColumnValue="%"+searchColumnValue+"%";
                            }else {
                                sql.append(" AND "+columnName+" = ?");
                            }
                            //将搜索条件按顺序存放 用于preparedStatement里设置搜索条件
                            searchColumnParams.put(paramIndex++,searchColumnValue);
                        }
                    } catch (NoSuchMethodException e) {
                        e.printStackTrace();
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    } catch (InvocationTargetException e) {
                        e.printStackTrace();
                    }
                }
            }


            try {

                //根据拼接好的sql语句获取到prepareStatement
                ps=connection.prepareStatement(sql.toString());
                Set<Map.Entry<Integer,Object>> entrySet=searchColumnParams.entrySet();

                //将之前存放的搜索条件依次设置到prepareStatement里
                for (Map.Entry<Integer,Object> entry: entrySet) {
                    ps.setObject(entry.getKey(),entry.getValue());
                }

                //执行搜索
                rs=ps.executeQuery();

                //遍历结果集
                while (rs.next()){
                    try {
                        //获取到对应实体类的构造函数
                        Constructor entityConstructor=entityClass.getDeclaredConstructor();
                        try {
                            //调用构造函数获取到实例
                            T resultRow=(T)entityConstructor.newInstance();

                            //根据之前存放的setter信息调用创建的实例的各个setter 绑定返回值
                            for (int i=0;i<resultColumnParams.size();i++){
                                Map<String,Object> resultMap=resultColumnParams.get(i);
                                Method setter=entityClass.getDeclaredMethod((String)resultMap.get("setter"),(Class)resultMap.get("paramClass"));
                                setter.invoke(resultRow,rs.getObject(i+1));
                            }

                            //将得到的每行对应的实例放到结果里
                            results.add(resultRow);
                        } catch (InstantiationException e) {
                            e.printStackTrace();
                        } catch (IllegalAccessException e) {
                            e.printStackTrace();
                        } catch (InvocationTargetException e) {
                            e.printStackTrace();
                        }
                    } catch (NoSuchMethodException e) {
                        e.printStackTrace();
                    }
                }
            } catch (SQLException e) {
                e.printStackTrace();
            } finally {
                if (rs!=null){
                    try {
                        rs.close();
                    } catch (SQLException e) {
                        e.printStackTrace();
                    }
                }
                if (ps!=null){
                    try {
                        ps.close();
                    } catch (SQLException e) {
                        e.printStackTrace();
                    }
                }

            }


        }
        //将搜索结果返回
        return results;
    }
}

-- 测试类
public class TestDBUtil {
    private DBUtil dbUtil;

    public TestDBUtil(){
        dbUtil=new DBUtil("daily");
    }
    @Test
    public void testQueryStudent(){
        Student student=new Student();
        student.setStudentId("201622010100");
        List<Student> students=dbUtil.query(student);
        System.out.println(students);

    }
    @Test
    public void testQueryTeacher(){
        Teacher teacher=new Teacher();
        teacher.setDuty("辅导员");
        List<Teacher> teachers=dbUtil.query(teacher);
        System.out.println(teachers);
    }
    @Test
    public void generateTestData(){
        String sql="INSERT student VALUES ";
        Long studentId=2016220101001L;
        String name="测试姓名";
        String major="测试专业";
        String contactWay="测试联系方式";
        Random random=new Random();
        FileOutputStream fileOutputStream=null;
        BufferedWriter bufferedWriter=null;
        try {
            fileOutputStream=new FileOutputStream("test.sql");
            bufferedWriter=new BufferedWriter(new OutputStreamWriter(fileOutputStream));
            for (int i=0;i<50;i++){
                String insertSql=sql;
                insertSql+="('"+(studentId+i)+"','"+name+i+"',"+random.nextInt(2)+",'"+major+i+"','"+contactWay+i+"');";
                bufferedWriter.write(insertSql);
                bufferedWriter.newLine();
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try{
                if (bufferedWriter!=null){
                    bufferedWriter.close();
                }
                if (fileOutputStream!=null){
                    fileOutputStream.close();
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```
这里看的出来 这个小工具是跟类型无关的 有较强的通用性 大多数ORM框架的实现机理其实也就是这样的

