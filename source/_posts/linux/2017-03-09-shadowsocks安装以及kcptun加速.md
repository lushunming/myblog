---
layout: post
title: shadowsocks安装以及kcptun加速
description: shadowsocks安装以及kcptun加速
tagline: 
image: 
date: 2017/03/09
categories: [linux]
tags: [linux,shadowsocks]
---

# shadowsocks安装以及kcptun加速


最近买了搬瓦工的vps，因为搬瓦工很便宜而且可以一键安装，所以选了他，但是缺点是速度不快。经过研究发现可以很快搭建一个快速的梯子。

为了方便所以我都是选择一键安装的脚本。

1. 一键安装shadowsocks
    
     ```
        wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-go.sh
        chmod +x shadowsocks-go.sh
        ./shadowsocks-go.sh 2>&1 | tee shadowsocks-go.log

    ```
    
    输入下面命令就会显示
    
    ```
        Congratulations, shadowsocks-go install completed!
        Your Server IP:your_server_ip
        Your Server Port:8989
        Your Password:your_password
        Your Local Port:1080
        Your Encryption Method:aes-256-cfb
        Welcome to visit:http://teddysun.com/392.html
        Enjoy it!

    ```
    
    就是安装完了。
    这个脚本默认已经安装开机自启动，
    
    ```
    启动：/etc/init.d/shadowsocks start
        
    停止：/etc/init.d/shadowsocks stop
        
    重启：/etc/init.d/shadowsocks restart
    
    状态：/etc/init.d/shadowsocks status

    ```
         
    以上是对shadowsocks的操作。
    
2. 安装kcptun server
	
    ```
    	wget --no-check-certificate https://raw.githubusercontent.com/kuoruan/kcptun_installer/master/kcptun.sh
    	chmod +x ./kcptun.sh
    	./kcptun.sh
    ```
    比较重要的是设置加速端口，一定是你的shadowsocks端口才行。设置错误也没有什么关系，可以修改配置文件。
3. 修改配置文件

    首先是修改shadowsocks配置文件
    
    ```
     vim /etc/shadowsocks/config.json
    ```
    
    显示是：
    
    ```
        {
            "server":"0.0.0.0",
            "server_port":65534, //服务端的端口，将来要用来配置kcptun。
            "local_port":1080,
            "password":"你的密码",
            "method":"aes-256-cfb",
            "timeout":600
        }
    ```
    
    然后重启shadowsocks服务。
    
    ```
     /etc/init.d/shadowsocks restart
    ```
    
    然后修改kcptun的配置文件。
    
    ```
     vim /usr/local/kcp-server/server-kcptun.json
    ```
    
    显示是：
    
    ```
        {
            "listen": ":45678", //监听端口随意选一个
            "target": "127.0.0.1:65534",//加速的端口就是你的shadowsocks的服务端口
            "key": "你的kcptun密码",
            "mode": "fast2",
            "mtu": 1350,
            "sndwnd": 1024,
            "rcvwnd": 1024,
            "nocomp": false
        }
    
    ```
    
    重新启动kcptun 
      
    ```
     supervisorctl restart kcptun
    
    ```
    
    服务端的配置就结束了
4. 本地配置

    在你的电脑上你需要先下载shadowsocks的客户端和kcptun的客户端。
    在kcptun客户端的目录中新建一个json文件`conf.json`,内容是：
    
    ```
        {
            "localaddr": ":12948", //本地端口，在shadowsocks客户端配置要用
            "remoteaddr": "162.211.225.21:45678",//要和服务端监听的端口一致（vps ip+listen）。
            "key": "kcptun的密码",
            "mode": "fast2",
            "conn": 3,
            "autoexpire": 60,
            "mtu": 1350,
            "sndwnd": 1024,
            "rcvwnd": 1024,
            "datashard": 10,
            "parityshard": 3,
            "dscp": 0,
            "nocomp": false,
            "acknodelay": false,
            "nodelay": 0,
            "interval": 20,
            "resend": 2,
            "nc": 1,
            "sockbuf": 4194304,
            "keepalive": 10
        }
    ```
    
    我不需要加密，所以kcptun的加密被我删了。
    
    在kcptun客户端目录下：
    
    ```
     client_linux_amd64 -c config.json
    
    ```
    
    然后开启shadowsocks客户端，
    
    ```
        serverAddress：127.0.0.1
        port：kcptun 客户端的localaddr
        加密和密码和服务端shadowdocks的加密和密码一样

    ```
    
    这样启动就可以了。
    其实就相当于shadowsocks服务端和kcptun服务端连接，shadowsocks客户端和kcptun客户端相连。kcptun服务端和客户端相互通信，而不是shadowsocks的服务端和客户端通信。如下图
      ![加速原理](https://raw.githubusercontent.com/xtaci/kcptun/master/kcptun.png)
    到此就结束了。







