---
title: 更方便实用的shell：zsh
date: 2017.02.14 11:23:22
tags:
    - zsh
    - shell
categories: tutorial
---

-------
#### mac 和 centos测试。

<!-- more -->

![mac 下使用item2效果图，根据主题 smt 做了部分调整](http://upload-images.jianshu.io/upload_images/1855546-bc8ec8bf0be53316.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 1. [安装zsh](http://blog.csdn.net/lina_acm/article/details/51815095)
1. 查看是否安装zsh：`cat /etc/shells`, 如果已经安装了的话，就可以跳过2或3了。
2. **RedHat / centos**: `sudo yum install zsh`
3. **Debian / Ubuntu**: `sudo apt-get install zsh`
4. **修改默认使用的shell**：
    ```
    chsh # 命令会提示输入要使用的shell程序名, 只能修改当前用户。
    chsh -s /bin/zsh # 直接指定shell
    ```
    或者
    ```
    vi /etc/passwd #有管理员权限，修改任意用户的shell
    ```
    或者
    ```
    useradd -s /bin/bash {用户名} # 创建用户时指定使用的shell
    ```

**参考：** [Linux下安装终极Shell Zsh](http://blog.csdn.net/lina_acm/article/details/51815095) , mac下也一样

## 2. [配置](http://blog.csdn.net/yangcs2009/article/details/45720193)
#### **oh-my-zsh**
1. [github](https://github.com/robbyrussell/oh-my-zsh)
2. **安装oh-my-zsh**：
    在zsh下执行以下命令。
    ```
    # 手动安装（不推荐）。。。
    git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
    ```
    或者， 以下两个会自动初始化一些东西
    ```
    curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh
    ```
    或者
    ```
    wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O - | sh
    ```
    安装完成后重新打开shell。
3. **更新**：
    ```
    cd ~/.oh-my-zsh
    git pull --rebase
    ```
4. **配置**：在~/.zshrc 文件中配置主题和插件.

**参考：** [Shell（一）：功能、配置和插件（附iTerm 2(for mac) && Oh My Zsh教程）](http://blog.csdn.net/yangcs2009/article/details/45720193)
