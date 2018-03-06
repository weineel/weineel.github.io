---
title: javascript 变量提升
date: 2017.07.04 15:36:33
tags:
    - javascript
    - scope
categories: javascript
---

-----

<!-- more -->

#### 源代码
``` javascript
console.log(x); // undefined
// console.log(y); // 这个会报为定义的错误
f(); // f 是个函数，函数定义也提升，函数定义就是定义实现，所以f是可以调用的(f不是通过赋值获得的引用)。
// f1(); // 会报 undefined 不是一个函数的错误，因为f1是一个函数类型的变量(不管什么类型他也是个变量)，定义提升了，所以f1的值是undefined

var x = 3;

function f(){
	console.log("ffff");
}

var f1 = function() {
	console.log("f111");	
};
```
#### 效果等同
```
var x;
function f(){
    console.log("ffff");
}
var f1;

console.log(x); // undefined
// console.log(y);
f(); 
// f1();

x = 3;

f1 = function() {
    console.log("f111");    
};
```

### 总结
1. 变量定义提升，只提升定义，不会初始化值
2. 函数定义时，直接进行函数定义函数可以调用，变量赋值的方式定义函数会报错。
