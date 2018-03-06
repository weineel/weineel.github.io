---
title: chrome 和 微信开发者工具 跨域
date: 2017.10.17 13:12:21
tags: 跨域
categories: tutorial
---

mac 和 windos平台下，chrome和微信开发者工具的跨域，跨域主要是为了在开发阶段调用接口。

<!-- more -->

![image.png](http://upload-images.jianshu.io/upload_images/1855546-b2ba87fc80bbd9d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## Mac 平台

要先完全退出chrome或微信开发者工具后执行。有可能需要尝试多次才能起作用。一般会提示 `你使用的是不受支持的命令标记 --disable-web-security`

#### chrome
```
open -a /Applications/Google\ Chrome.app --args --disable-web-security --user-data-dir
```

#### 微信开发者工具
不同版本的微信开发者工具在/Applications下的名字可能不同，根据实际情况修改命令行。

```
open -a /Applications/wechatwebdevtools.app --args --disable-web-security --user-data-dir
```

![微信开发者工具跨域](http://upload-images.jianshu.io/upload_images/1855546-3fd0efc77a2fb503.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Windows
#### chrome
1. 新建一个目录，例如: `C:\path\to\ChromeUserData`，可以根据自己的习惯创建。
2. 新建一个能打开chrome的快捷方式，右击快捷方式，选择`属性`，弹出属性窗口。
3. 在`快捷方式`选项卡下的`目标`输入框中追加上` --disable-web-security --user-data-dir=C:\path\to\ChromeUserData`

![image.png](http://upload-images.jianshu.io/upload_images/1855546-33e75ae59dcd07a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 点击应用和确定按钮关闭属性窗口，双击这个快捷方式，并打开chrome，查看是否有`你使用的是不受支持的命令标记 --disable-web-security`的提示。

#### 微信开发者工具
对比chrom的操作执行。

## 警告
所有操作要在完全退出chrome或微信开发者工具后操作。

## 参考
* [微信开发者工具 跨域问题](http://www.cnblogs.com/lhyforfront/p/6867683.html)
* [chrome浏览器的跨域设置——包括版本49前后两种设置](http://www.cnblogs.com/laden666666/p/5544572.html)
