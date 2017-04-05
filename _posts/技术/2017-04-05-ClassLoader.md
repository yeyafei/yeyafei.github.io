---
layout: post
title: 对ClassLodader的理解
category: 技术
tags: ClassLoader
keywords: 
description: 
---





### 类加载器（classLoader）
类加载器可以动态的将java类加载到java虚拟机中，在一般java编程中并不会ClassLoader，但是在看了些框架源码时发现ClassLoader经常见到，我们在日常coding中也经常遇到一些异常比如ClassNotFoundException NoClassDefFoundEroor等等。觉得了解一下ClassLoader的加载机制也有利于自己对java的理解。
首先看看ClassLoader源码中的一些内置方法

``` java

/**
*加载名称为 name的类，返回的结果是 java.lang.Class类的实例。
*/
public Class<?> loadClass(String name) throws ClassNotFoundException {
	return loadClass(name, false);
    }
    
/**
* 得到该类的父加载器
*/
public final ClassLoader getParent() {
	if (parent == null)
	    return null;
	SecurityManager sm = System.getSecurityManager();
	if (sm != null) {
	    ClassLoader ccl = getCallerClassLoader();
	    if (ccl != null && !isAncestor(ccl)) {
		sm.checkPermission(SecurityConstants.GET_CLASSLOADER_PERMISSION);
	    }
	}
	return parent;
    }
 //。。。。。。。。。。。。。。。。。
``` 
#### 基本概念
Jvm将java文件编译成class文件后，类加载器通过读取java字节码来转换成一个class实例，
系统为所有载入内存里的类都会生成一个java.lang.class对象，同一个类只会被加载一次，在JVM中每一个不同的类都会有一个不同的类加载器负责。
使用示例：
``` java
public class ClassLoaderTest {
	public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
		 Class<?> cls = Thread.currentThread().getContextClassLoader().loadClass("com.yyf.V"); 
		 V v = (V)cls.newInstance(); 
		 System.out.println(v.getClass().getName());
		 //com.yyf.V
	}
}

``` 

#### 类加载器结构
系统给我们提供了三个类加载器，
 BootstrapClassloader ：在JVM运行的时候加载java核心的API
 ExtClassLoader   ：载java的扩展API
 AppClassLoader  ：加载用户机器上CLASSPATH设置目录中的Class
当运行一个程序的时候，JVM启动，运行bootstrapclassloader，该ClassLoader加载java核心API，然后调用ExtClassLoader加载扩展API，最后AppClassLoader加载CLASSPATH目录下定义的Class.
*结构是自下而上的父子结构*
我们可以通过一段代码来验证 ：

``` java

public class ClassLoaderTest {
	public static void main(String[] args) throws ClassNotFoundException,
			InstantiationException, IllegalAccessException {
		ClassLoader loader = ClassLoaderTest.class.getClassLoader();
		while (loader!=null) {
			System.out.println(loader.toString());	
			loader = loader.getParent();
		}
	} 
}
//输出结果：sun.misc.Launcher$AppClassLoader@1372a1a
//         sun.misc.Launcher$ExtClassLoader@ad3ba4

``` 

很直观的能看到现在使用的类加载器是AppClassLoader其父加载器为ExtClassLoder，由于为了保护核心加载器BootstrapClassloader不被访问其返回值为null。

#### 类加载器机制
类加载机制：
委托机制：当一个类加载器准备去加载某个Class时会先让其父类去加载，当其父类无法加载时才会用自己去加载当前的类。我们自定义的类加载器是父结构是连在在上系统提供的AppClassLoader。
代理机制：ClassLoder加载一个Class时，这个Class所依赖的和引用的所有Class也默认由这个ClassLoder负责载入，当然你可以强制指定由另一个ClassLoder加载。

#### 类加载方式
类加载有三种方式：
1、命令行启动应用时候由JVM初始化加载
2、通过Class.forName()方法动态加载
3、通过ClassLoader.loadClass()方法动态加载

#### 自定义类加载器
loading。。。。。。





	





