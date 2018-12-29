---
layout:     post
title:      "Design pattern of Proxy pattern"
subtitle:   " \"代理模式\""
date:       2018-12-28 14:00:00
author:     "Chamberlain"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Design Pattern

---



> “工欲善其事，必先利其器”


## 前言

设计模式在面向对象的代码设计中如果运用的好，可以让我们的代码运行起来更加高效，相反，如果运用的不好，不仅会增加代码的复杂度，而且还会使得系统更加臃肿。下面，我就来简单的聊聊我对代理模式的一些见解。


<p id = "build"></p>
---

## 正文

### 模式动机 

在某些情况之下，客户端不想直接引用或者不能直接引用某个类的对象，此时可以通过一个**代理对象**来实现间接引用，代理对象在两者之间起到了第三者的作用，同时，代理对象还可以隐藏被引用对象的一些功能和添加一些服务，这就是代理模式的初衷。

### 模式定义

代理模式在设计模式分类中属于一种结构型模式，给某一个对象提供一个代理，并由代理对象控制对原对象的引用。



![类图如下](/img/proxy.jpg )

### 模式结构

代理模式包括以下角色：

+ Subject：抽象主题角色


+ Proxy：代理主题角色
+ RealSubject：真实主题角色

#### 典型的代理模式实现

```java
public class Proxy implements Subject
{
    private RealSubject realSubject = new RealSubject();
    public void preRequest()
    {…...}
    public void request()
    {
        preRequest();
        realSubject.request();
        postRequest();
    }
    public void postRequest()
    {……}
} 

```

### 模式优缺点

#### 优点

* 代理模式能够协调调用者和被调用者，在一定程度上降低了系统的耦合度。
* 虚拟代理通过使用一个小对象来代表一个大对象，可以减少系统资源的消耗，对系统进行优化并提高运行速度。
* 可以控制对真实对象的使用权限。

#### 缺点

* 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢。
* 实现代理模式需要额外的工作，有些代理模式的实现非常复杂。

### 模式应用

代理模式在面向对象程序设计中应用非常广泛。

#### 远程代理

> 为一个位于不同的地址空间的对象提供一个本地的代理对象，这个不同的地址空间可以是在同一台主机中，也可是在另一台主机中。

#### 虚拟代理

> 如果需要创建一个资源消耗较大的对象，先创建一个消耗相对较小的对象来表示，真实对象只在需要时才会被真正创建。

**EJB、WebService等分布式技术都是代理模式的应用。Spring 框架中的AOP技术也是代理模式的应用，在Spring AOP中应用了动态代理(Dynamic Proxy)技术。 **

#### 动态代理

> 在传统的代理模式中，客户端通过Proxy调用RealSubject类的`request()`方法，同时还在代理类中封装了其他方法(如`preRequest()`和`postRequest()`)，可以处理一些其他问题。如果按照这种方法使用代理模式，那么真实主题角色必须是事先已经存在的，并将其作为代理对象的内部成员属性。如果一个真实主题角色必须对应一个代理主题角色，这将导致系统中的类个数急剧增加，**因此需要想办法减少系统中类的个数,主要是减少代理类的个数**，此外，如何在事先不知道真实主题角色的情况下使用代理主题角色，这都是动态代理需要解决的问题。

Java动态代理实现相关类位于`java.lang.reflect`包，主要涉及两个类：

* `InvocationHandler`接口。它是代理实例的调用处理程序实现的接口，该接口中定义了如下方法：

  ```java
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;

  ```

  `invoke()`方法中第一个参数proxy表示代理类，第二个参数method表示需要代理的方法，第三个参数args表示代理方法的参数数组。

* `Proxy`类。该类即为动态代理类，该类最常用的方法为：

  ```java
  public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException。
  ```


  `newProxyInstance()`方法用于根据传入的接口类型interfaces返回一个动态创建的代理类的实例，方法中第一个参数loader表示代理类的类加载器，第二个参数interfaces表示代理类实现的接口列表（与真实主题类的接口列表一致），第三个参数h表示所指派的调用处理程序类。

  #### 典型的动态代理模式代码实现

  ##### AbstractSubject.java

```Java
  public interface AbstractSubject
  {
  	public void request();
  }
```

  ##### RealSubjectA.java

```java
  public class RealSubjectA implements AbstractSubject
  {	
  	public void request()
  	{
  		System.out.println("真实主题类A");
  	}
  }
```

  ##### RealSubjectB

```java
  public class RealSubjectB implements AbstractSubject
  {	
  	public void request()
  	{
  		System.out.println("真实主题类B");
  	}
  }
```

  ##### Client.java

```java
  import java.lang.reflect.InvocationHandler;
  import java.lang.reflect.Proxy;

  public class Client
  {
  	public static void main(String args[])
  	{
  	InvocationHandler handler =null;
      AbstractSubject subject=null;
      
      handler=new DynamicProxy(new RealSubjectA());
      subject=(AbstractSubject)Proxy.newProxyInstance(AbstractSubject.class.getClassLoader(), new Class[]{AbstractSubject.class}, handler);
      subject.request();
      
      System.out.println("------------------------------");
      
      handler=new DynamicProxy(new RealSubjectB());
      subject=(AbstractSubject)Proxy.newProxyInstance(AbstractSubject.class.getClassLoader(), new Class[]{AbstractSubject.class}, handler);
      subject.request();
  	} 
  }
```

  ##### DynamicProxy.java

```Java
  import java.lang.reflect.InvocationHandler;
  import java.lang.reflect.InvocationTargetException;
  import java.lang.reflect.Method;
  public class DynamicProxy implements InvocationHandler
  {
  	private Object obj;
  	
  	public DynamicProxy(){}
  	
  	public DynamicProxy(Object obj)
  	{
     		this.obj=obj;
     	}
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable
      {
  		preRequest();
          method.invoke(obj, args);
          postRequest();
          return null;
      }
      public void preRequest(){
      System.out.println("调用之前！");
      }
      public void postRequest(){
      System.out.println("调用之后！");
      }

  }
```

#### 图片代理

>
> 用户通过浏览器访问网页时先不加载真实的大图，而是通过代理对象的方法来进行处理，在代理对象的方法中，先使用一个线程向客户端浏览器加载一个小图片，然后在后台使用另一个线程来调用大图片的加载方法将大图片加载到客户端。当需要浏览大图片时，再将大图片在新网页中显示。如果用户在浏览大图时加载工作还没有完成，可以再启动一个线程来显示相应的提示信息。通过代理技术结合多线程编程将真实图片的加载放到后台来操作，不影响前台图片的浏览。
>

## 后记

关于代理模式，现在总结如下：

* 代理模式包含三个角色：抽象主题角色声明了真实主题和代理主题的共同接口；代理主题角色内部包含对真实主题的引用，从而可以在任何时候操作真实主题对象；真实主题角色定义了代理角色所代表的真实对象，在真实主题角色中实现了真实的业务操作，客户端可以通过代理主题角色间接调用真实主题角色中定义的方法。
* 代理模式的优点在于能够协调调用者和被调用者，在一定程度上降低了系统的耦合度；其缺点在于由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢，并且实现代理模式需要额外的工作，有些代理模式的实现非常复杂。
* 如果需要创建一个资源消耗较大的对象，先创建一个消耗相对较小的对象来表示，真实对象只在需要时才会被真正创建，这个小对象称为虚拟代理。虚拟代理通过使用一个小对象来代表一个大对象，可以减少系统资源的消耗，对系统进行优化并提高运行速度。