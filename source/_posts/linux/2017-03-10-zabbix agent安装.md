---
layout: post
title: zabbix agent 安装
description: zabbix agent 安装
tagline: 
image: 
date: 2017/03/10
categories: [linux]
tags: [linux,zabbix,zabbix agent]
---
# zabbix agent 安装

## 环境

windows服务器

## 安装

1. 下载

    到官网下载`Windows(All)`，就是windows下的agent包。链接是：[http://www.zabbix.com/downloads/3.0.4/zabbix_agents_3.0.4.win.zip](http://www.zabbix.com/downloads/3.0.4/zabbix_agents_3.0.4.win.zip)。

2. 配置

    下载完后解压，会得到一个文件夹，下面包含

    * bin 运行文件
    * config 配置文件夹

    打开config文件夹下的`zabbix_agentd.win.conf`，
    修改下面的配置

    ```
    Server=120.26.36.24
    ServerActive=120.26.36.24
    Hostname=101.37.27.100 windows

    ```
    *  Server是填写zabbix服务的ip
    * ServerActive也是填zabbix服务的ip，这是用来检测agent是否活跃的，不配置服务端就检测不到
    * Hostname 是agent的名字，需要全局唯一，和监控添加的hostName要一样的。
    
    然后用命令安装
    
    ```
    C:\zabbix\bin\win64\zabbix_agentd.exe -c C:\zabbix\conf\zabbix_agentd.win.conf  --install
    ```
    这是把zabbix安装为一个服务，就可以通过windows的服务来控制了。

3. 监控

    在frontend配置一个host,主机名和agent配置文件中的一样，填写agent的ip和端口（默认10050，如果修改了请自己修改）。链接windows server 模板，一会就会看到`ZBX`变绿了，就说明监控成功了。
    
    