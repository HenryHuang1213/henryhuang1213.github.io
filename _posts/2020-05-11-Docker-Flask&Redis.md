---
layout: post
title: '多容器复杂应用的部署'
subtitle: 'Docker下Python Flask + Redis的部署'
date: 2020.5.11
author: HenryHuang
tags:
    - docker
    - Redis
    - Flask
---

***准备工作：***

1. 准备一个python Flask的小程序：app.py
	
   该程序会在网页端展示被访问的次数以及主机ID

	```
	from flask import Flask
	from redis import Redis
	import os
	import socket

	app = Flask(__name__)
	redis = Redis(host=os.environ.get('REDIS_HOST', '127.0.0.1'), port=6379)


	@app.route('/')
	def hello():
	    redis.incr('hits')
	    return 'Hello Container World! I have been seen %s times and my hostname is %s.\n' % (redis.get('hits'),socket.gethostname())


	if __name__ == "__main__":
	    app.run(host="0.0.0.0", port=5000, debug=True)
	```

2. 准备好对应的Dockerfile：Dockerfile

	```
	FROM python:2.7
	LABEL maintaner="Peng Xiao xiaoquwl@gmail.com"
	COPY . /app
	WORKDIR /app
	RUN pip install flask redis
	EXPOSE 5000
	CMD [ "python", "app.py" ]
	```

3. 一台安装好Docker的机器（虚拟机/服务器）


## 第一步，创建Redis的容器。

```
sudo docker run -d --name redis redis
```

注意：此处不用使用-p参数将容器内Redis的6379接口暴露到容器外，因为该容器仅供Python-Flask在容器内访问，而应该禁止从外部对其进行访问。

## 第二步，创建Python Flask容器。

#### 首先，创建Flask程序的镜像文件（Docker image）

```
sudo docker build -t test/flask-redis .
```

#### 然后，通过这个新的镜像文件去创建容器。

```
sudo docker run -d -p 5000:5000 --link redis --name flask-redis -e REDIS_HOST=redis test/flask-redis
```

此处通过--link在容器内部连接上Redis容器，同时设置环境变量REDIS_HOST为Redis容器的名字。

此处可以进行一个小测试：
```
sudo docker exec -it flask-redis /bin/bash
```
进入容器内部，然后ping另一个容器。
```
ping redis
```

此时，在该容器内部：
```
curl 127.0.0.1:5000
```
即可得到打印出来的呈现效果：
```
Hello Container World! I have been seen 1 times and my hostname is e7fd318e75d6
```
反复多次尝试，可得到叠加结果。说明此处已经将访问次数存储到Redis数据库。

此时，退出容器（保持继续运行）。在容器外部进行尝试。由于暴露了容器内5000端口到容器外，所以在该机器内直接访问：
```
curl 127.0.0.1:5000
```
仍然能够得到预期的结果。


***由此可以轻易推广到其他各类多容器复杂应用。***