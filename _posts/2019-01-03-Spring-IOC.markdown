---
layout:     post
title:      "Spring IOC—依赖注入"
subtitle:   "Spring依赖注入的几种方式 "
date:       2019-01-03 14:00:00
author:     "Chamberlain"
header-img: "img/post-bg-alibaba.jpg"
catalog: true
tags:
    - Spring
    - JAVA
---

> "纸上得来终觉浅，绝知此事要躬行"



##  前言

最近在研读Spring源码，对源码和文档内容是一头雾水，感觉虽然对Spring里的思想略知一二，但是具体怎么运用Spring还是不够熟练，所以决定自己动手写一些case来加深对Spring框架的理解。

Spring核心的框架中核心是IOC容器和AOP，我们先从IOC开始吧！



## 正文

**IOC**(Inverse of Control)全称叫依赖倒转，什么叫依赖倒转呢？依赖如果是正的该是什么样的呢？比如我们在类A方法中要引用类B的对象，我们就必须要在类A中实例化类B，这个时候对类B对象的控制权就在类A中，那么类A和类B就存在一条依赖，这个依赖就是正的。但是这样的做法会使得类A和类B之间存在高度耦合，高内聚低耦合是面向对象强调的设计思想，为了解决这个问题，依赖倒转的思想就提出来了，假如类A需要引用类B的对象，这个工作不需要由类B完成，由特定的框架先将类B的对象实例化出来，再由特定的框架注入类A中，这样一来，类A是否引用类B的对象就可以根据具体的业务场景灵活控制，这样一来，类B的控制权就交给了特定的框架。

> 依赖控制反转的实现有很多种方式。在Spring中IOC容器只是实现这个模式的载体。特别要注意的是，控制反转是关于一个对象如何获取它所依赖的对象的引用，在这里，反转是指创建对象责任的反转。在Spring中，这个责任就交给了IOC容器。

接下来，我就结合例子讲解Spring两大注入配置方式：XML配置文件和注解，和三大注入实例方法。

### XML配置文件

Spring-IOC的配置可以通过XML文件来实现，每一个类在XML配置文件都叫做bean，至于什么叫bean，简单的说，就是Spring操作的单元，类似于面向对象中的对象。类与类对象之间的依赖关系全部都定义在Bean中，然后Spring通过解析XML文件通过反射创建对象，最后通过不同的方式注入到相应的类中去。常见的注入方式有接口注入、setter方法注入、构造函数注入，第一种方式不常用，我们下面只讲解后两种方法。

#### setter方法注入

我们创建两个类，一个类Teacher，一个类Student。

```java
public class Teacher
{
    private Student student;
    public Student getStudent()
    {
        return student;
    }
   	public void setStudent(Student student)//通过setter方法注入时，Teacher类一定要有set方法。
    {
        this.student = student;
    } 
}

public class Student
{
    private String name;
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
}
```

下面来看看配置文件setter.xml(配置文件的名字无所谓)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student" class="com.chamberlain.Student" scope="prototype">
        <property name="name" value="张三"></property>
    </bean>
    <bean id="teacher" class="com.chamberlain.Teacher" scope="prototype">
        <property name="student" ref="student"></property>
    </bean>
</beans>
```

> 以上的配置文件里定义了两个bean，第一个bean时学生的bean，里面给出了学生的初始姓名张三，下面teacher bean里面依赖了一个学生的bean，正是这个才会在程序运行过程中，spring框架会讲student的实例注入到Teacher里面去。注意：***这里scope是bean的作用域，当scope为singleton时，全局只会创建一个对象；当为prototype时每一次请求都会生成一个新的bean实例，另外还有几个作用域request、session、global session***。

下面来看看测试类

```java
 @Test
    public void setter(){
     ApplicationContext tx=new ClassPathXmlApplicationContext("/setter.xml");
     Teacher teacher=(Teacher) tx.getBean("teacher");
     System.out.println(teacher.getStudent().getName());
    }
```

> 输出张三

#### constructor注入

构造器注入必须在Teacher里面声明相应的构造函数。两个类如下：

```java
public class Teacher
{
    private Student student;
    public Student getStudent()
    {
        return student;
    }
    public Teacher(Student student){//构造函数
        this.student=student;
    }
}

public class Student
{
    private String name;
    public String getName()
    {
        return name;
    }
    public void setName(String name)
    {
        this.name = name;
    }
}
```

下面来看看配置文件constructor.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student" class="com.chamberlain.Student">
        <property name="name" value="张三"></property>
    </bean>
    <bean id="teacher" class="com.chamberlain.Teacher" scope="prototype">
       <constructor-arg ref="student"/>
      <--! 构造函数的参数--!>
    </bean>
</beans>
```

测试方法：

```java
    @Test
    public void func3(){
     ApplicationContext tx=new ClassPathXmlApplicationContext("/constructor.xml");
     Teacher teacher=(Teacher) tx.getBean("teacher");
     System.out.println(teacher.getStudent().getName());
    }
```

> 输出张三

### 注解方法

Spring的配置注入还可以通过注解来完成，就是不需要XML文件也可以完成配置。不过要增加一个配置类来代替XML文件。

```java
@Component(value = "teacher")
public class Teacher
{
    @Resource(name = "student")
    private Student student;
    public Student getStudent()
    {
        return student;
    }
}

@Component(value = "student")
public class Student
{
    private String name="张三";
    public String getName()
    {
        return name;
    }
}

@Configuration
@ComponentScan(basePackages = {"com.chamberlain.entity"})//定义需要扫描的包
public class Config {//配置类
}

@Test
public void func4(){//测试方法
   ApplicationContext tx4=new AnnotationConfigApplicationContext(SpringConfig.class);
   Teacher teacher=(Teacher)tx4.getBean("teacher");
   System.out.println(teacher.getStudent().getName());

    }
```

> 输出张三

这种通过注解来完成配置注入的方法是目前的主流方法。



## 后记

关于Spring的IOC在结合例子的基础之上理解原理我觉得是至关重要的，在看别人的教程的同时手动实现一些case加深理解，当然要想正真弄明白Spring，还必须仔细研习源码。