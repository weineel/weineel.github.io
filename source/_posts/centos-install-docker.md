---
title: centos docker环境搭建
date: 2017.08.15 10:06:58
tags: 
    - centos
    - docker
categories: docker
---

********
## 概述
基于centos搭建docker部署或开发环境。使用docker-compose实现单机的容器集群。

<!-- more -->

## 步骤
---------
#### 更换yum源
**参考**: 
1. [Ali-OSM-CentOS](http://mirrors.aliyun.com/help/centos)
2. [centos7 修改yum源为阿里源](http://mirrors.aliyun.com/help/centos)
#### 安装pip，python的包管理工具
1. 更新yum缓存，安装python-pip包
```
yum update -y
# 如果没找到包，执行yum -y install epel-release 然后再次执行一次。
yum install python-pip 
```
2. 更换pip镜像源到阿里，新建`~/.pip/pip.conf`:

    ```
    mkdir ~/.pip
    tee ~/.pip/pip.conf <<-'EOF'
    [global]
    index-url = http://mirrors.aliyun.com/pypi/simple/
    [install]
    trusted-host=mirrors.aliyun.com
    EOF
    ```

3. 升级pip: `pip install --upgrade pip`
### 安装docker-compose
```
pip install docker-compose 
```

### 安装并启动docker
1. 使用阿里云提供的docker安装方式
    i. 登录阿里[docker平台-管理中心](https://account.aliyun.com/login/login.htm?oauth_callback=https%3A%2F%2Fcr.console.aliyun.com%2F%3Fspm%3D5176.1971733.0.2.h888jf&lang=zh)
    ii. 点击Docker Hub 镜像站点根据系统版本和提示进行安装和修改registry镜像源。
    ![阿里docker平台管理中心](http://upload-images.jianshu.io/upload_images/1855546-ea238070a3207731.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 直接去官网下载安装包，或者yum安装，为了下载docker镜像快一点，registry的镜像源还是要换一下的。
3. 使用阿里云的安装步骤
    i. 安装：`curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -`
    ii. 使用Docker加速器（修改registry 镜像源）
    ```
    sudo mkdir -p /etc/docker
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["获取的专属加速地址"]
    }
    EOF
    sudo systemctl daemon-reload
    ```
    iii. 启动docker：
    `sudo systemctl restart docker`
    iv. 测试：
    命令：`docker -v` 输出：`Docker version 17.05.0-ce, build 89658be`

4. 设置docker开机启动: `systemctl enable docker`

5. 容器自动重启: 
    ```
    docker run --restart=always imagename
    ```

#### 以上所有操作都是用的root用户。。。

##### 待续: *mac下 docker化开发*

## 参考
----------
* [CentOS7下安装python-pip](http://blog.csdn.net/yulei_qq/article/details/52984334)
* [centos7 修改yum源为阿里源](http://blog.csdn.net/jameshadoop/article/details/54881295)
* [Ali-OSM-CentOS](http://mirrors.aliyun.com/help/centos)
* [centos7安装docker并设置开机启动](http://www.mamicode.com/info-detail-1304628.html)
