---
layout: post
title: javamelody使用
description: javamelody使用,对tomcat 应用进行性能监控
tagline: 
image: 
date: 2019-09-14
categories: [web]
tags: [web性能监控,javamelody,sql监控]
---
# javamelody使用



---

## 前提
我的项目是一个`Spring+Spring MVC+Mybatis`的项目，用war 发布，所以我们是加入`javamelody`的jar包的方式集成javamelody。其他方式暂不介绍。

## 设置
1. 在pom中增加需要的依赖

    ```
    <!-- javamelody-core ,需要的-->
    <dependency>
    	<groupId>net.bull.javamelody</groupId>
    	<artifactId>javamelody-core</artifactId>
    	<version>1.79.0</version>
    </dependency>
    <!-- 可选用来导出pdf -->
    <dependency>
    	<groupId>com.lowagie</groupId>
    	<artifactId>itext</artifactId>
    	<version>2.1.7</version>
    	<exclusions>
    		<exclusion>
    			<artifactId>bcmail-jdk14</artifactId>
    			<groupId>bouncycastle</groupId>
    		</exclusion>
    		<exclusion>
    			<artifactId>bcprov-jdk14</artifactId>
    			<groupId>bouncycastle</groupId>
    		</exclusion>
    		<exclusion>
    			<artifactId>bctsp-jdk14</artifactId>
    			<groupId>bouncycastle</groupId>
    		</exclusion>
    	</exclusions>
    </dependency>
    ```

2. 配置web.xml
    如果应用server兼容servlet3.0 ，这一步可以忽略。否则在web.xml的自己定义的servlet之前加上
    ```
    <filter>
    	<filter-name>javamelody</filter-name>
    	<filter-class>net.bull.javamelody.MonitoringFilter</filter-class>
    	<async-supported>true</async-supported>
    </filter>
    <filter-mapping>
    	<filter-name>javamelody</filter-name>
    	<url-pattern>/*</url-pattern>
    	<dispatcher>REQUEST</dispatcher>
    	<dispatcher>ASYNC</dispatcher>
    </filter-mapping>
    <listener>
    	<listener-class>net.bull.javamelody.SessionListener</listener-class>
    </listener>
    ```
3. 启动访问`http://<host>/<context>/monitoring`就可以看到监控信息。这个时候可以发现`图标http`和`图标jsp`都已经有了统计信息，按照访问热度排序好了。但是`图标sql`还是空白的，我们接着设置监控sql。
4. 监控sql
    我们使用的spring定义的dataSource，所以数据源能被spring后置处理器监控。
    ```
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
        classpath:net/bull/javamelody/monitoring-spring.xml
        classpath:context/services.xml
        classpath:context/data-access-layer.xml
        /WEB-INF/applicationContext.xml
        </param-value>
    </context-param>
    ```
    在contextConfigLocation中引入`net/bull/javamelody/monitoring-spring.xml` 文件，就可以监控sql了。如果`monitoring-spring.xml`中的配置与aop冲突，就引入 `classpath:net/bull/javamelody/monitoring-spring-datasource.xml`。

5 . 当然这个工具不仅仅有这些功能，还有更多的功能我暂时用不到，就不去研究了，有需要的自己查看官方的文档。



