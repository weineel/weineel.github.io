---
title: docker 容器中使用 crontab
date: 2017.08.01 10:37:05
tags:
    - docker
    - crontab
categories: docker
---

# docker 中使用crontab

<!-- more -->

### 相关命令

* 使用 python:2.7.13 镜像，是基于buildpack-deps:jessie镜像的，是一个Debian系统。
* docker中使用cron执行python脚本时:
    ```
    sys.path = ['/var/code/XHTSpider/XHTSpider/es', '/usr/lib/python2.7', '/usr/lib/python2.7/plat-x86_64-linux-gnu', '/usr/lib/python2.7/lib-tk', '/usr/lib/python2.7/lib-old', '/usr/lib/python2.7/lib-dynload', '/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages']
    ```
  在直接使用python命令执行脚本时
    ```
    sys.path = ['/var/code/XHTSpider/XHTSpider/es', '/usr/local/lib/python27.zip', '/usr/local/lib/python2.7', '/usr/local/lib/python2.7/plat-linux2', '/usr/local/lib/python2.7/lib-tk', '/usr/local/lib/python2.7/lib-old', '/usr/local/lib/python2.7/lib-dynload', '/usr/local/lib/python2.7/site-packages']
    ```
  注意 dite-packages 和 site-packages。然而在使用pip安装包时，是安装在site-packages下的，所以在要在要执行的脚本开始部分加上
    ```
    import sys
    sys.path.append("/usr/local/lib/python2.7/site-packages")
    ```
    这可能和docker容器的用户管理有关，执行cron的用户环境变量的不同。
* debian 中 crontab 程序叫 cron，服务名也是cron，不同系统的使用方式会有所不同。

    ```
    service cron start/stop/restart/reload/status
    # 直接启动
    cron 
    ```

### Dockerfile

```
FROM python:2.7.13

WORKDIR /root

COPY ./etc/apt.sources.list /etc/apt/sources.list

RUN apt-get update -y && \
    apt-get install -y cron && \
    touch /var/log/cron.log

ADD ./etc/crontab /etc/cron.d/crontab

# tail 可以防止容器自动退出运行
CMD cron && tail -f /var/log/cron.log

```


### 相关文件
 1. `./etc/apt.sources.list`
 	
 	apt-get 使用阿里云源
 
 ```
deb http://mirrors.aliyun.com/debian wheezy main contrib non-free
deb-src http://mirrors.aliyun.com/debian wheezy main contrib non-free
deb http://mirrors.aliyun.com/debian wheezy-updates main contrib non-free
deb-src http://mirrors.aliyun.com/debian wheezy-updates main contrib non-free
deb http://mirrors.aliyun.com/debian-security wheezy/updates main contrib non-free
deb-src http://mirrors.aliyun.com/debian-security wheezy/updates main contrib non-free

 ```
 
 2. `./etc/crontab`
 
 ```
  */1 * * * * root echo $(date) >> /var/log/cron.log 2>&1
 ```

### 参考
1. [docker下计划任务crontab的使用方法](http://blog.csdn.net/lizhihua0925/article/details/52370479)
2. [crontab 使用](http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)
3. [Running cron within a docker debian:jessie container](https://stackoverflow.com/questions/34095397/running-cron-within-a-docker-debianjessie-container)
4. [Linux crontab定时执行任务 命令格式与详细例子](http://www.jb51.net/LINUXjishu/19905.html)
5. [apt-get安装源替换 阿里云源](http://www.cnblogs.com/gabin/p/6519352.html)
6. [[Python 安装路径， dist-packages 和 site-packages 区别](http://www.cnblogs.com/kevin922/p/3161411.html)](http://www.cnblogs.com/kevin922/p/3161411.html)
