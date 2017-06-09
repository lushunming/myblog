---
layout: post
title: zabbix 监控windows下kafka
description: zabbix 监控windows下kafka
tagline:
image:
date: 2017/06/09
categories: [linux]
tags: [linux,zabbix,kafka]
---

# zabbix 监控windows下kafka

zabbix 监控kafka主要使用的是zabbix的java gateway。


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

2. 设置kafka，开启jmx。

    在`bin\windows`下找到kafka-server-start.bat文件，在
    ```
    IF ["%KAFKA_HEAP_OPTS%"] EQU [""] (
    set KAFKA_HEAP_OPTS=-Xmx1G -Xms1G

    )
    ```
    下加上`set JMX_PORT=9999 `变为

    ```
    IF ["%KAFKA_HEAP_OPTS%"] EQU [""] (
    set KAFKA_HEAP_OPTS=-Xmx1G -Xms1G
    set JMX_PORT=9999  
    )
    ```


3. 配置主机
    ![jmx端口配置](http://img.blog.csdn.net/20170609162931573?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamhmc2Rmcw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    jmx的端口一定要与JMX_PORT一样。
    链接[`Kafka`模板](https://raw.githubusercontent.com/lushunming/zabbix_file/master/kafka/zbx_kafka_templates.xml)。
