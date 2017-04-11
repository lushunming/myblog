---
layout: post
title: Zabbix 通过jmx监控windows 下ActiveMq
description: Zabbix 通过jmx监控windows 下ActiveMq
tagline: 
image: 
date: 2017/03/06
categories: [linux]
tags: [linux,zabbix,ActiveMq]
---

# Zabbix 通过jmx监控windows 下ActiveMq 

Zabbix 通过jmx监控windows 下ActiveMq

1.	开启ActiveMq的jmx

    打开对应版本的wrapper.conf配置文件，修改配置，使jmx可用

    ```
    # Uncomment to enable remote jmx
    wrapper.java.additional.n=-Dcom.sun.management.jmxremote.port=12346
    wrapper.java.additional.n=-Dcom.sun.management.jmxremote.authenticate=false
    wrapper.java.additional.n=-Dcom.sun.management.jmxremote.ssl=false
    ```


    修改conf目录下的activemq.xml，在broker上增加useJmx="true"
    并且修改 managementContext节点
    ```
    <managementContext>
                <managementContext createConnector="true" connectorPort="12346"/>
                
    </managementContext>
    ```
    虽然在wrapper.conf中已经配置了端口，但是在这里不配置connectPort就不能用jmx连接。

2.	新建主机的时候,需要设置jmx 端口为上面设置的端口,模板选择Template JMX Generic。
