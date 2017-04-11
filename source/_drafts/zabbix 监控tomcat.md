---
layout: post
title: zabbix 监控windows下tomcat
description: zabbix 监控windows下tomcat
tagline: 
image: 
date: 2017/03/06
categories: [linux]
tags: [linux,zabbix,tocmat]
---

# zabbix 监控windows下tomcat 

zabbix 监控tomcat主要使用的是zabbix的java gateway。


1. 安装zabbix gateway。使用ubuntu，可以直接使用命令安装java gateway。

    ```
    apt-get install zabbix-java-gateway
    ```

    修改zabbix_server的配置文件，告知zabbix server java gateway在哪个端口，并且设置poller的个数。

    ```
    JavaGateway=安装ip
    JavaGatewayPort=10052 #监听端口
    StartJavaPollers=5
    ```

    开启java gateway，重启zabbix server

    ```
    service zabbix-java-gateway start
    service zabbix-server restart
    ```

2. 设置tomcat，开启jmx。

    在`tomcat/bin`下找到catalina.bat文件，在`set "CURRENT_DIR=%cd%"`下加上

    ```
    set JAVA_OPTS=-Dcom.sun.management.jmxremote.port=12345 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false
    ```
3. 下载支持`jmx` 的jar包`catalina-jmx-remote.jar`,把他放到tocmat目录下的`lib`包中。

4. 配置主机
    ![jmx端口配置](http://img.blog.csdn.net/20170324115028226?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamhmc2Rmcw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    jmx的端口一定要与在`catalina.bat`下配置的`jmxremote.port`一样。
    链接`Template JMX Generic`和`Template JMX Tomcat`这两个模板。



