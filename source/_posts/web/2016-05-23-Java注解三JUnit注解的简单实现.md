---
layout: post
title: Java注解三 JUnit注解的简单实现
description: JUnit注解的简单实现
tagline: 
image: 
date: 2016/05/23
categories: [web]
tags: [java,annoation]
---
#Java注解三 JUnit注解的简单实现



上一次只是使用了一个简单的例子，讲解了一个自定义的注解实现。
今天借助实现一个简单的JUnit来讲解怎么使用注解进行测试。

首先我们先实现一个经典的入门类，HelloWorld.java

```java
package com.lu.testclass;

public class HelloWorld {

	private String message;

	public HelloWorld(String message) {
		this.message = message;
	}

	public String sayHello() {
		System.out.println(this.message);
		return this.message;
	}

}

```

然后我们要实现在JUnit中常用的三个注解：`@Test,@Before,@After`，代码如下。

```java
package com.lu.annnoation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 在方法之前执行
 * @author lusm
 *
 */
@Target(value=ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface After {

}
```
```java
package com.lu.annnoation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 测试后执行
 * @author lusm
 *
 */
@Target(value=ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Before {

}
```
```java
package com.lu.annnoation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 测试
 * 
 * @author lusm
 *
 */
@Target(value=ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {

}

```
接着我们写一个测试类

```java
package com.lu.testCase;

import com.lu.annnoation.After;
import com.lu.annnoation.Before;
import com.lu.testclass.HelloWorld;

public class HelloTest {
	private HelloWorld helloWorld;

	@Before
	public void init() {
		helloWorld = new HelloWorld("it is  a test");
	}

	@com.lu.annnoation.Test
	public void test() {
		helloWorld.sayHello();
	}

	@After
	public void after() {
		System.out.println("测试结束");

	}

}
```
接下来我们就要实现最重要的注解工具。思想就是`@Before`先执行，`@Test`再执行，`@After`最后执行。下面是代码

```java
package com.lu.runner;

import java.lang.annotation.Annotation;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;

import com.lu.annnoation.After;
import com.lu.annnoation.Before;
import com.lu.annnoation.Test;
import com.lu.testCase.HelloTest;

/**
 * 解析注解，执行测试
 * 
 * @author lusm
 *
 */
public class Runner {
	// 被注解的类
	private Class<?> clazz;
	// 被注解的类的实例
	private Object object;
	// 先执行的方法list
	private List<Method> beforeMethods = new ArrayList<Method>();
	// 后执行的方法list
	private List<Method> afterMethods = new ArrayList<Method>();
	// test的方法list
	private List<Method> testMethods = new ArrayList<Method>();

	/**
	 * 构造类
	 * 
	 * @param clazz
	 */
	public Runner(Class<?> clazz) {
		this.clazz = clazz;
	}

	/**
	 * 执行测试
	 */
	public void doRun() {
		// 获取所有的方法并将他们分别放入各自的集合中
		Method[] methods = clazz.getMethods();
		for (Method method : methods) {
			AddIntoList(method, beforeMethods, Before.class);
			AddIntoList(method, afterMethods, After.class);
			AddIntoList(method, testMethods, Test.class);
		}
		// 执行各自集合中的方法
		runBefore();
		runTest();
		runAfter();
	}

	/**
	 * 测试前执行的方法
	 */
	private void runBefore() {
		for (Method method : beforeMethods) {
			try {
				object = clazz.newInstance();
				method.invoke(object, new Object[] {});
				System.out.println("test before-----------");
			} catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException | InstantiationException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

	/**
	 * 测试后执行的方法
	 */
	private void runAfter() {
		for (Method method : afterMethods) {
			try {
				method.invoke(object, new Object[] {});
				System.out.println("test after-----------");
			} catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

	/**
	 * 测试方法
	 */
	private void runTest() {
		for (Method method : testMethods) {
			try {
				System.out.println("testing -----------");
				method.invoke(object, new Object[] {});
				System.out.println("testing -----------");
			} catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

	/**
	 * 查看method是不是被clazz注解，如果是，就把他加入到list中
	 * 
	 * @param method 测试类中的方法
	 * @param list
	 * @param clazz 注解类
	 */
	private void AddIntoList(Method method, List<Method> list, Class<? extends Annotation> clazz) {
		if (method.isAnnotationPresent(clazz)) {
			list.add(method);
		}
	}
}


```
最后写一个main方法测试下这个测试类。

```java
package com.lu.testCase;

import com.lu.runner.Runner;

public class main {
	public static void main(String[] args) {
		Runner runner = new Runner(HelloTest.class);
		runner.doRun();
	}
}

```
得到的结果是：

```
test before-----------
testing -----------
it is  a test
testing -----------
测试结束
test after-----------

```
和JUnit中一样的，简直完美，但是这只是一个简单的例子，真正的测试比这个复杂多了，希望大家通过这个例子对注解有更深的认识。样例代码托管在https://git.oschina.net/shunming/Annoation.git ，可以用git拷贝。

由于jekyll的环境非常难维护，所以转转阵营了，blog换为Hexo的了，而且换了主题。博客的地址是http://www.lushunming.com.cn    ，我的博客中的排版更为美观，欢迎大家去看我的博客，也可以用邮箱订阅。
