---
title: 在idea上使用docker作为java的开发环境
date: 2017.09.28 15:19:25
tags:
    - docker
    - idea
    - java
categories: docker
---

------

## 准备，开发环境使用的MacOS, windows 和 Linux理论上差异不大。

<!-- more -->

0. `idea 2017.2`, `docker integration 3.0.1`
1. 安装idea插件 ` Docker integration 3.0.1`
2. 安装docker for Mac 和 docker-compose (一般使用pip或brew安装)
3. 在idea中指定docker-compose的目录。
    ```
    # 安装完成docker-compose查看可执行文件目录
    which docker-compose
    # /usr/local/bin/docker-compose
    ```
    在idea中打开 IntelliJ IDEA > Preferences > Build, Execution, Deployment > Docker > Tools. 在`Docker Compose executable` 中配置`/usr/local/bin/docker-compose`
    
    ![配置docker-compose](http://upload-images.jianshu.io/upload_images/1855546-4246351b217b4316.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    
4. 安装方法自行百度或Google，文章结尾有部分参考。

## 配置连接本地docker daemon
1. 配置
![连接本地docker daemon](http://upload-images.jianshu.io/upload_images/1855546-05a07ffe7fcb855c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 连接
![链接到 docker daemo](http://upload-images.jianshu.io/upload_images/1855546-dd91c39faefdd812.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 配置连接远程docker daemon
1. 在服务器上配置可以远程连接的docker daemon
    * 远程登录到[安装docker的服务器](http://www.jianshu.com/p/f2194618fd3c)，编辑文件 `/etc/docker/daemon.json`, 在json最外层加上 `"hosts":["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"],`， 类似下面的结构。
    
    ```
    # 0.0.0.0表示任意IP的主机都可以访问，安全起见 0.0.0.0 改成允许访问的IP。
    {
      "hosts":["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"],
      "registry-mirrors": ["https://ftichs.mirror.aliyuncs.com"]
    }
    ```
    * 防火墙开启 2375 端口
    ```
    firewall-cmd --zone=public --add-port=2375/tcp --permanen
    firewall-cmd --reload
    ```
    * 重启docker，`systemctl restart docker`
    * 在本地机器(外网ip必须是daemon.json配置的IP)测试，`docker -H server_ip:2375 images`
2. 配置 idea，和配置本地基本一样。
    ![配置远程docker daemon](http://upload-images.jianshu.io/upload_images/1855546-ac1eddbc54a379e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3. 连接和连接本地docker一样。

## 编写一个配置文件，部署应用
1. 要先有一个docker-compose.yml/Dockerfile/docker镜像。任意一个都行，看你想用什么方式部署了。下面用[docker-compose](https://docs.docker.com/compose/compose-file/)做实例。
```
version: '3.1'

services:
  tomcat:
    image: tomcat:7.0.81-jre8
    ports:
      - "8088:8080"

```
2. 配置，使用docker-compose就可以忽略Container选项卡了。
![配置运行配置项](http://upload-images.jianshu.io/upload_images/1855546-15b47785006e923a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 运行(部署)
![运行(部署)](http://upload-images.jianshu.io/upload_images/1855546-737ed26bfd5af175.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

部署成功后访问 `http://localhost:8088`查看效果。可以通过编写Dockerfile(在docker-compose.yml中引用Dockerfile编译镜像)把java应用部署到docker 容器。

 
## idea配置远程调试, 调试部署到docker中的应用
下面是不使用docker时的远程调试
* [Intellij IDEA 配置Tomcat远程调试](http://blog.csdn.net/mingjie1212/article/details/52281847)
* [Debugging a Java app in a container](https://www.jetbrains.com/help/idea/debugging-a-java-app-in-a-container.html)


## 参考
* [idea 官方文档](https://www.jetbrains.com/help/idea/docker.html?search=docker)
* [ubuntu+docker+docker-compose+intellij idea 部署java web项目](https://yq.aliyun.com/articles/138741)
* [IDEA使用docker进行调试](http://www.cnblogs.com/xiaohunshi/p/5892067.html)
* [从零开始使用Docker构建Java Web开发运行环境](http://blog.csdn.net/u010397369/article/details/41349587)
* [centos docker环境搭建](http://www.jianshu.com/p/f2194618fd3c)
* [docker 配置外网访问](https://yq.aliyun.com/articles/86626?spm=5176.8091938.0.0.btZh0s)
* [Debugging a Java app in a container](https://www.jetbrains.com/help/idea/debugging-a-java-app-in-a-container.html)
