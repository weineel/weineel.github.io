# sprint

## 个人主页

### 自定义域名

* 有一个自己的域名
* 在云解析中配置CNAME类型的解析规则，记录值为weineel.github.io
* 然后在本项目setting中配置自定义域名为，云解析配置的记录sprint.weinee.top

## 自动部署
使用 trivis 拉取src分支代码编译结果 push 到master分支。
