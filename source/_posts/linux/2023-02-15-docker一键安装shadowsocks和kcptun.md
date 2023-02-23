---
layout: post
title: docker一键安装shadowsocks和kcptun
description: docker一键安装shadowsock很kcptun
tagline:
image:
date: 2023/02/15
categories: [linux]
tags: [linux,shadowsocks,kcptun,docker]
---
# docker一键安装shadowsocks+kcptun
1. 准备：一台海外Linux VPS（推荐Ubuntu18及以上），一个ssh连接工具。
2.  
	- 安装docker：`wget -qO- https://get.docker.com/ | sh`
	- 运行ss+kcp：
    ```
        docker run -t -i --rm --network=host \
        -v /var/run/docker.sock:/var/run/docker.sock \
        lushunming/kcpss:3.0.0 bootstrap
    ```
    ，运行后会打印出连接URL和二维码链接

	```
		Current container: 2de5d0362e16ef4b134e471ec9ce7ecf47624496e5d92a4c404aaf669108d2d4
		Current image: yaleh/kcp-shadowsocks-server:runit
		Network interface: wlp58s0
		Host name: 172.18.0.25
		Exported Shadowsocks port: 15858
		Exported KCPTUN port: 9974
		Password: ohHoh4bi

		docker run -d --restart=always --name ss-15858-kcp-9974 -e SS_PASSWORD=ohHoh4bi -e SS_METHOD=aes-256-cfb -e KCPTUN_MODE=normal -e KCPTUN_PASSWORD=ohHoh4bi -e KCPTUN_SNDWND=256 -e KCPTUN_RCVWND=256 -e SS_LINK=ss://YWVzLTI1Ni1jZmI6b2hIb2g0YmlAMTcyLjE4LjAuMjU6MTU4NTg=#SS:172.18.0.25:15858 -e KCPTUN_SS_LINK=ss://YWVzLTI1Ni1jZmI6b2hIb2g0Ymk=@172.18.0.25:9974?plugin=kcptun%3Bmode%3Dnormal%3Brcvwnd%3D256%3Bsndwnd%3D256%3Bkey%3DohHoh4bi%3Bmtu%3D1350#KCP_SS%3A172.18.0.25%3A9974 -p 15858:8338/tcp -p 9974:41111/udp yaleh/kcp-shadowsocks-server:runit

		Worker container: 29d1d7206b99a809520792382a99ee350b2b8030e4bdf5566577e017e1dd8a5e
		Worker container name: ss-15858-kcp-9974

		----

		Shadowsocks link: ss://YWVzLTI1Ni1jZmI6b2hIb2g0YmlAMTcyLjE4LjAuMjU6MTU4NTg=#SS:172.18.0.25:15858

		QR code: https://api.qrserver.com/v1/create-qr-code/?size=300x300&data=ss%3A//YWVzLTI1Ni1jZmI6b2hIb2g0YmlAMTcyLjE4LjAuMjU6MTU4NTg%3D%23SS%3A172.18.0.25%3A15858

		----

		KCPTUN SS link (for Android and Windows clients): ss://YWVzLTI1Ni1jZmI6b2hIb2g0Ymk=@172.18.0.25:9974?plugin=kcptun%3Bmode%3Dnormal%3Brcvwnd%3D256%3Bsndwnd%3D256%3Bkey%3DohHoh4bi%3Bmtu%3D1350#KCP_SS%3A172.18.0.25%3A9974

		QR code: https://api.qrserver.com/v1/create-qr-code/?size=300x300&data=ss%3A//YWVzLTI1Ni1jZmI6b2hIb2g0Ymk%3D%40172.18.0.25%3A9974%3Fplugin%3Dkcptun%253Bmode%253Dnormal%253Brcvwnd%253D256%253Bsndwnd%253D256%253Bkey%253DohHoh4bi%253Bmtu%253D1350%23KCP_SS%253A172.18.0.25%253A9974

    ```
       
		
	
	
	- 安装bbr加速

    ```
        wget --no-check-certificate -O tcp.sh https://raw.githubusercontent.com/Mufeiss/Linux-NetSpeed/master/tcp.sh && chmod +x tcp.sh && ./tcp.sh
    ```

    ，到此就可以使用了。
	
3. 最后推荐两个我用的vps ，便宜流量足，一年一两百以内的价格，满足查资料看看油管的需求。
	- [俄罗斯justhost](https://justhost.ru/?ref=53818)
	- [美国cloudcone](https://app.cloudcone.com/?ref=9200)

