---
title: 使用ssl自签证书搭建 Nodejs https 服务器, 并采用 AFNetworking 调用示例接口
date: 2016.09.28 21:07:35
tags:
    - nodejs
    - ios
    - https
categories: https
---

************************************************
使用 openssl 生成证书
--------------------------------------------
openssl 说明: 使用 openssl 命令会把生成的证书输出到当前目录.

<!-- more -->

1. 创建 CA 证书: 此证书是自签证书的根本, 创建了此证书相当于你自己就是第三方证书签发机构 (CA)了.
生成步骤:  
``` 
  1. 生成 CA 秘钥   
    openssl genrsa -out ca-key.pem -des3 2048
  2. 通过CA私钥生成CSR(证书请求文件)
    openssl req -new -key ca-key.pem -out ca-csr.pem
  3. 通过CSR文件和私钥生成CA证书(我理解为公钥, 包含了CA的信息)
    openssl x509 -req -in ca-csr.pem -signkey ca-key.pem -out ca-cert.pem
  ```
#### 如果遇到权限问题
  你需要root或者admin的权限 Unable to load config info from /user/local/ssl/openssl.cnf 对于这个问题，你可以从网上下载一份正确的openssl.cnf文件， 然后set OPENSSL_CONF=openssl.cnf文件的本地路径

2. 创建服务器端证书: 此证书主要是在创建服务器时使用的, 在不同的认证方式中使用的证书不同, 下面会给出一些区别例子.
在当前目录新建 openssl.cnf 文件
模板:
```
[req]  
    distinguished_name = req_distinguished_name  
    req_extensions = v3_req  
  
    [req_distinguished_name]  
    countryName = Country Name (2 letter code)  
    countryName_default = CN  
    stateOrProvinceName = State or Province Name (full name)  
    stateOrProvinceName_default = BeiJing  
    localityName = Locality Name (eg, city)  
    localityName_default = YaYunCun  
    organizationalUnitName  = Organizational Unit Name (eg, section)  
    organizationalUnitName_default  = Domain Control Validated  
    commonName = Internet Widgits Ltd  
    commonName_max  = 64  
  
    [ v3_req ]  
    # Extensions to add to a certificate request  
    basicConstraints = CA:FALSE  
    keyUsage = nonRepudiation, digitalSignature, keyEncipherment  
    subjectAltName = @alt_names  
  
    [alt_names]  
	#注意这个IP.1的设置，IP地址需要和你的服务器的监听地址一样
    IP.1 = 127.0.0.1
```
生成步骤:
``` 
  1. 生成服务器秘钥   
    openssl genrsa -out server-key.pem 2048
  2. 通过服务器私钥生成CSR(证书请求文件)
    openssl req -new -key server-key.pem -config openssl.cnf -out server-csr.pem
  3. 通过CSR文件和私钥以及CA证书生成服务器证书(我理解为公钥, 包含了 CA 证书,服务器信息(域名等..), 和服务器公钥)
    openssl x509 -req -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -in server-csr.pem -out server-cert.pem -extensions v3_req -extfile openssl.cnf
  ```
3. 创建客户端证书: 此证书,在某些情况下可以不使用. 比如我们下面的 iOS 端使用 AFNetworking 调用接口的例子.
生成步骤:
```
生成客户端私钥
  openssl genrsa -out client-key.pem
生成CSR
  openssl req -new -key client-key.pem -out client-csr.pem
生成客户端证书(我理解为公钥)
openssl x509 -req -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -in client-csr.pem -out client-cert.pem
```
###  参考:
[HTTPS单向认证和双向认证](http://my.oschina.net/nearzk/blog/485652)

************************************************
使用express框架创建 nodejs 服务器
--------------------------------------------
1. 安装express generator , 我使用的是4.13.4版本, nodejs 的版本为v5.11.1, npm 的版本为3.8.6
`` npm install -g express-generator #必要时可能需要 sudo ``
2. 安装完成后使用 express 命令,生成 express app
`` express httpsapp #执行结束后会在当前目录下生成 nodejs 项目 httpsapp``
3. 安装依赖,测试项目创建是否成功
```
cd httpsapp
npm install
//執行
npm start
```
  在浏览器中打开[http://localhost:3000](http://localhost:3000/)查看是否启动成功.
![启动成功返回的页面](http://upload-images.jianshu.io/upload_images/1855546-74f8590a2a6970e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 在项目根目录下创建 ssl 文件夹, 并把刚刚生成的证书放到其中.
5. 修改bin/www的代码, 把 http服务器改成 https 服务器
 以下是修改后的代码: 保证核心代码一样就行.
```
#!/usr/bin/env node
var app = require('../app');
var debug = require('debug')('httpsApp:server');
// var http = require('http');
var https = require('https');
var fs = require('fs');
var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);
/**
 * 创建 https 服务器
*/
// 获取证书
var options = {
  key: fs.readFileSync('./ssl/server-key.pem'),
  ca: [fs.readFileSync('./ssl/ca-cert.pem')],
  cert: fs.readFileSync('./ssl/server-cert.pem')
};
var server = https.createServer(options, app).listen(app.get('port'),'127.0.0.1');
server.listen(port);
server.on('error', onError);
server.on('listening', onListening);
function normalizePort(val) {
  var port = parseInt(val, 10);
  if (isNaN(port)) {
    // named pipe
    return val;
  }
  if (port >= 0) {
    // port number
    return port;
  }
  return false;
}
function onError(error) {
  if (error.syscall !== 'listen') {
    throw error;
  }
  var bind = typeof port === 'string'
    ? 'Pipe ' + port
    : 'Port ' + port;
  switch (error.code) {
    case 'EACCES':
      console.error(bind + ' requires elevated privileges');
      process.exit(1);
      break;
    case 'EADDRINUSE':
      console.error(bind + ' is already in use');
      process.exit(1);
      break;
    default:
      throw error;
  }
}
function onListening() {
  var addr = server.address();
  var bind = typeof addr === 'string'
    ? 'pipe ' + addr
    : 'port ' + addr.port;
  debug('Listening on ' + bind);
}
```
6. 再次打开浏览器查看[https://localhost:3000](http://localhost:3000/).如下图则服务器创建成功.

![修改为 https 服务器后的页面](http://upload-images.jianshu.io/upload_images/1855546-e6b12047f1e49cf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7.  打开routs/users.js 文件(或者自己根据路由创建自己的测试接口)
  修改为:
``` 
var express = require('express');
var router = express.Router();
/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send({
  	"code" : "1100",
  	"message" : "test success"
  });
});
module.exports = router;
```
这样就完成了一个简单的测试接口.

8. 编写客户端代码测试接口是否可用.
在任意目录创建文件 client.js
内容为:
```
var https = require('https');
var fs = require('fs');
var options = {
	hostname:'127.0.0.1',
	port:3000,
	path:'/users',
	method:'GET',
	key:fs.readFileSync('../ssl/client-key.pem'),
	cert:fs.readFileSync('../ssl/client-cert.pem'),
	ca: [fs.readFileSync('../ssl/ca-cert.pem')],
	agent:false
};
options.agent = new https.Agent(options);
var req = https.request(options,function(res){
console.log("statusCode: ", res.statusCode);
  console.log("headers: ", res.headers);
	res.setEncoding('utf-8');
	res.on('data',function(d){
		console.log(d);
	})
});
req.end();
req.on('error',function(e){
	console.log(e);
})
```
最后执行命令
``` node client.js``` 
返回结果为:
![返回结果](http://upload-images.jianshu.io/upload_images/1855546-85c1a7cd5f7d6859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
说明接口没有问题.

## 以上过程, 可以参考 :
[用Node.js创建自签名的HTTPS服务器](https://cnodejs.org/topic/54745ac22804a0997d38b32d)
[Node.js - Express 4.x 使用 HTTPS/SSL](http://jade.logdown.com/posts/233332-nodejs-express-4x-using-https-ssl)

************************************************
在 iOS 端使用 AFNWorking 调用接口
---------------------------------------------

1. ** 在 iOS 中使用 https 需要注意 **
首先必须要基于TLS 1.2版本协议。再来就是连接的加密方式要提供Forward Secrecy (这个不太懂)。最后就是证书至少要使用一个SHA256的指纹与任一个2048位或者更高位的RSA密钥，或者是256位或者更高位的ECC密钥。如果不符合其中一项，请求将被中断并返回nil。从我们生成证书的过程中也看到秘钥的长度是2048.

2. ** 转换证书格式**
此部分参考[使用openssl进行证书格式转换](http://blog.csdn.net/linda1000/article/details/8676330)
其中 cer 和 der 是基本通用的,由于 AFNWorking 推荐使用 cer 类型的证书,所以我们可以使用一下命令把我们生成的 pem 证书转换成 cer.
``` 
openssl x509 -outform der -in server-cert.pem -out server-cert.cer
```
基于AFNWorking 的验证方式我们只需要 server 的证书(公钥)就可以了.
3. 把生成的 cer 证书拖到项目中.
4. 使用一下代码条用接口查看返回数据
```
//把服务端证书(需要转换成cer格式)放到APP项目资源里，AFSecurityPolicy会自动寻找根目录下所有cer文件
	NSString *cerPath = [[NSBundle mainBundle] pathForResource:@"server-cert" ofType:@"cer"];
	NSData *cerData = [NSData dataWithContentsOfFile:cerPath];
	NSSet *cerSet = [[NSSet alloc] initWithObjects:cerData, nil];
	AFSecurityPolicy *securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModePublicKey withPinnedCertificates:cerSet];
	securityPolicy.allowInvalidCertificates = YES;
    securityPolicy.validatesDomainName = NO;
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.requestSerializer = [AFHTTPRequestSerializer serializer];
    manager.responseSerializer = [AFJSONResponseSerializer serializer]; // 这一步需要注意否者返回的数据可能是 NSData 需要自行转换成 json.
    manager.securityPolicy = securityPolicy;
    manager.responseSerializer.acceptableContentTypes=[NSSet setWithObjects:@"application/json", @"text/json", @"text/javascript", @"text/plain", nil];
//关闭缓存避免干扰测试
    manager.requestSerializer.cachePolicy = NSURLRequestReloadIgnoringLocalCacheData;	[manager GET:@"https://127.0.0.1:3000/users" parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
		DDLogDebug(@"%@", responseObject);
	} failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
		DDLogError(@"%@", error);
	}];
```
 返回结果为

![iOS 控制台返回结果](http://upload-images.jianshu.io/upload_images/1855546-21d4033994d565a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上内容很多地方引用自[iOS中AFNetworking HTTPS的使用](http://www.jianshu.com/p/20d5fb4cd76d)
