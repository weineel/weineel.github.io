---
title: python 中文编码问题总结 -- str 和 unicode
date: 2017.08.16 19:06:56
tags:
    - python
    - unicode
categories: python
---

### 术语 
* **unicode** 是通用码（原始码）一个字符对应一个整数, 但不是程序数据。具体的怎么编码和存储由不同版本的UTF定义。
* **UTF（Unicode Transformation Format）**：是针对unicode变长编码设计的一种前缀吗，根据前缀可判断是几个字节表示一个字符。定义了unicode 定义的数字怎么转换成程序数据(实际使用和存储的数据)。常用的用utf-8，utf-16
* **codec**: 编码解码器，根据不同的字符集执行编码解码。

<!-- more -->

### python2
1. 关于文件开头的 `# -*- coding: utf-8 -*-`。 这个是指定当前python代码文件的编码方式，python2默认是ascii。这个决定了python解释器用什么编码解释代码文件。
2. 字符串的两种形式，str(字节序列)和unicode(*用某种编码格式解码字节序列形成的字符串*)，str更底层所以unicode到str的转换是encode(编码)。
1. str 是通过wchar_t(宽字符，根据不容系统或编译方式长度不同)类型存储的，可以看作是字节流。console打印一个字符串，需要看console当前使用的编码格式。
2. `encode`: unicode对象调用，根据指定编码类型生成str类型(字节序列/宽字节序列)，str类型调用的话会先使用系统默认编码decode成unicode对象，然后再根据指定的编码encode，注意默认的一次decode. `u.encode("utf-8")`
3. `decode`: str对象调用，根据制定的编码格式解码成unicode对象。
4. 关于`'ascii' codec can't decode byte 0xe5 in position 23: ordinal not in range(128)` 经典错误。报这个错一般是因为uncode和str对象混用了，在使用默认的`ascii codec`转换时导致的错误。
5. 使用unicode的建议，主要是为了**避免混用**：
    1. 程序中出现字符串时一定要加一个前缀u。
    2. 不要用str()函数，用Unicode()代替
    3. 不要用过时的string模块。如果传给它非ASCII码，它会把一切搞砸。
    4. 不到必须时不要在你的程序里编解码Unicode字符，只在你要写入文件或者数据库或者网络时，才调用encode()函数和decode()函数。
    5. 使用什么字符编码，就要采用对应的字符集进行解码。

### python3
1. 使用byte表示字节序列
2. 使用str表示unicode。所以一般不会遇到编码问题。

#### 参考资料
* [百度百科](http://baike.baidu.com/link?url=LJNiGd7XUTRs2Qm5OYc8agh9IBBh4WWPoBAjdkSFl6f2vMLVdG7SkQOTkfcgqCt2pbIc36BA06T4L5fiNw_r0a#reference-[1]-40801-wrap)
* [python中的应用](http://www.cnblogs.com/shine-lee/p/4504559.html)
* [python decode encode](http://blog.csdn.net/trochiluses/article/details/16825269)
* [中文报错/乱码](http://www.cnblogs.com/pengei/p/6407077.html)
* [python2 和 python3的主要区别](http://www.jianshu.com/p/ae31c83ce9f5?utm_campaign=hugo&utm_medium=reader_share&utm_content=note&utm_source=qq)
