---
title: python 爬虫在 docker 中的应用实战
date: 2017.09.07 18:41:56
tags:
    - docker
    - python
    - spider
categories: python
---

-------
使用 docker 统一开发、测试、发布环境。[示例项目地址](https://github.com/weineel/docker-scrapy-example)。

<!-- more -->

### docker 的在线和离线安装

1. 在线安装
	* [centos docker环境搭建](http://www.jianshu.com/p/f2194618fd3c)
	
2. 离线安装

	官网下载对应平台的安装包。
	
3. 核心概念
	
	* 镜像，类似于虚拟机/操作系统镜像，可以理解为面向Docker引擎的只读模版。是创建docker容器的基础，可以看作面向对象中的类。
	* 容器，通过镜像创建的一个实例，类似于一个轻量级的沙箱，或者说是一个已经启动的操作系统。Docker使用容器来运行和隔离应用。可以看作面向对象中的对象。
	* 仓库，可以看作代码库(如果是java，可以看作是maven或者gradle的仓库)。是Docker集中存放镜像文件的地方。
	
4. 基本使用(对比git)，mac和windows都有相应的图形界面。
	* 镜像拉取: `docker pull imagename:tag`
	* 从镜像运行容器（镜像不存在时会自动pull，任务结束后容器就会自动停止）: `docker run --rm -v /var/code:/root/code -p 8080:80 --name=example imagename:tag`
	* `--rm` 和 `-d` 不能同时使用
	* 交互式运行容器中的指定程序（容器要处于运行状态）: `docker exec -it example bash`
	* 启动和停止容器: `docker start/stop example` 
	* 从容器中复制文件或目录: `docker copy `

5. 数据卷(Data Volumes)管理

	数据卷的使用，类似Linux下对目录或文件进行mount操作。
	
	* 创建默认数据卷
	
		```
		# 只能是rw模式，会分配一个默认的VolumesName(hash值)，容器被删除时会自动删除数据卷
		docker run -v /test_py --name=test_python --rm -it python:2.7.12 bash
		```

		1. 容器中的 /test_py 目录会被持久化在 /var/lib/docker/volumes/VolumesName (可以使用docker volume inspect 命令查看)中。
		2. Mac下无法直接查看 /var/lib/docker 到，需要在这个模式下 `screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty 查看`，具体可以查看[这里](https://stackoverflow.com/questions/38532483/where-is-var-lib-docker-on-mac-os-x)。

	* 给数据卷挂载宿主主机的目录或文件
	
		```
		# 容器被删除时不会删除数据卷
		docker run -v ~/project/pythonProject/test_py:/code:rw --name=test_python --rm -it python:2.7.12 bash
		docker run -v ~/project/pythonProject/test_py/helloworld.py:/code/helloworld.py:ro --name=test_python --rm -it python:2.7.12 bash
		```
	
* 命名数据卷的使用，使用这个数据卷的容器被删除时不会删除数据卷
	
     ![数据卷命令](http://upload-images.jianshu.io/upload_images/1855546-4e7cb5b3e4171017.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
	
6. 网络管理
	
	默认情况下，在容器外部是无法通过网络来访问容器内的网络应用和服务的。通过 `-p` 参数来指定端口映射。

	```
	docker run -p ip:hostport:containerport imagename
	```
	实例
	
	```
	# ip一般省略为127.0.0.1, 访问宿主主机的8080端口相当于访问容器的80端口
	docker run -p 8080:80 imagename
	```

### 实战--搭建基于docker的scrapy开发和发布环境。

* [示例项目地址](https://github.com/weineel/docker-scrapy-example)

* 使用[Dockerfile](https://jiajially.gitbooks.io/dockerguide/content/chapter_fastlearn/dockerfile_details.html)构建一个python scrapy爬虫的镜像，可以直接通过`docker build -t scrapyd-dev:1.0.0 .`命令构建镜像，最终会使用docker-compose来实现。
	
	```
	# Dockerfile
	# Author: weineel
	FROM python:2.7.13
	
	MAINTAINER weineel LiJF_wn@163.com
	
	WORKDIR /var/code
	
	# COPY/ADD(可以是一个压缩包，会自动解压)
	COPY ./etc/pip.conf /root/.pip/pip.conf
	COPY ./etc/scrapyd.conf /etc/scrapyd/scrapyd.conf
	
	RUN pip install --no-cache-dir scrapy scrapyd
	
	# 定义编译指令
	# 在编译时通过 docker build --build-arg _TZ=Asia/Shanghai,_TC=weiguo -t test_python:1.0.0 . 方式指定，
	# 可以用在run等指令中，做分支判断
	ARG _TZ=Asia/Shanghai
	ARG _TC
	# 在启动容器是 -e 指定环境变量，会覆盖编译时赋的值。
	ENV TZ $_TZ
	ENV TC $_TC
	
	VOLUME /var/code
	
	EXPOSE 6800
	
	# ENTRYPOINT ["/bin/bash"]
	# ENTRYPOINT 和 CMD的区别
	# 相当于开机启动项，container在启动时执行
	CMD ["scrapyd", "--pidfile="]

	```
	
* 使用docker-compose管理docker

	```
	# docker-compose.yml
	version: '3.1'

	services:
	
	  scrapyd:
	    build:
	      context: ./scrapyd
	      dockerfile: ${ScrapydDockerfileName}
	    ports:
	      - "6800:6800"
	    volumes:
	      - ./scrapyd_data:/var/lib/scrapyd
	      - ../:/var/code
	    # depends_on:
	    #   - splash
	    # links:
	    #   - splash
	    restart: always
	
	  splash:
	    image: scrapinghub/splash
	    ports:
	      - "8050:8050"
	      	
	```
	
* 构建步骤, 以下命令在项目根目录中的docker目录下执行
	0. 初始项目目录结构
             
        ![屏幕快照 2017-09-07 下午5.45.52.png](http://upload-images.jianshu.io/upload_images/1855546-17c9d1b6e358d6bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

	1. `docker-compose up scrapyd` 启动容器。
	2. `docker-compose exec scrapyd bash` 进入容器(相当于远程登录的服务器)，容器中安装了scrapy包，可以执行scrapy的命令行工具等，因为把项目根目录挂在到了容器的`/var/code` 所以在容器中的`/var/code`目录下的内容和项目根目录一致的。
	3. `scrapy startproject myproject` [创建爬虫项目](https://docs.scrapy.org/en/latest/topics/commands.html#creating-projects)，此命令是在容器中执行的，注意观察命令行提示符。
	4. `scrapy genspider gjfgw http://www.ndrc.gov.cn` [创建爬虫](https://docs.scrapy.org/en/latest/topics/commands.html#genspider)
	5. 接下来就是快乐的写爬虫了[scrapy](https://docs.scrapy.org/en/latest/index.html)。

### 参考
* [docker技术入门与实战](https://www.amazon.cn/Docker%E6%8A%80%E6%9C%AF%E5%85%A5%E9%97%A8%E4%B8%8E%E5%AE%9E%E6%88%98-%E6%9D%A8%E4%BF%9D%E5%8D%8E-%E6%88%B4%E7%8E%8B%E5%89%91-%E6%9B%B9%E4%BA%9A%E4%BB%91/dp/B00SMJ0VFA/ref=sr_1_2?ie=UTF8&qid=1504778990&sr=8-2&keywords=docker%E6%8A%80%E6%9C%AF%E5%85%A5%E9%97%A8%E4%B8%8E%E5%AE%9E%E6%88%98)
