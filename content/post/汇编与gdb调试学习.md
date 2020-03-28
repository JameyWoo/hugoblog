---
title: "汇编与gdb调试学习"
date: 2019-03-09
draft: false
category: ["计算机系统"]
---


#### 1、在gdb中如何列出汇编代码
应该是不可以用list 命令列出汇编代码的。
但可以使用`display /i $pc` 命令在调试的时候出了列出一行源码，也列出相应的汇编代码
同时，s和si等的区别还是比较大的：si按汇编一行一行执行，有的源码一行会有很多条汇编；
我认为这是个学习汇编的好方法：**使用gdb一步一步调试，对比汇编和源码**

#### 2、如何将一个可执行文件或者是.o文件得到它的汇编码或者是源码？
可以使用`objdump -d test.out` 获取汇编代码（右侧）以及机器码（左侧）；要查看但里面有很多除我写的东西之外的东西，要具体定位到自己写的东西，可根据函数名查看。

如果编译时**使用了-g参数**的话，使用`objdump -dS test` 就可以得到机器码，源码，汇编码一一对应了！但如果没用-g的话，可执行文件是没有源码信息的，这时需要通过特殊手段得到。

![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpggy1g0wz27sv1rj30s60gwjy7.jpg)
#### 3、将c源码变成.o文件，会不会很干净，和变成可执行文件的区别？体量？
编译过程图，来源：https://blog.csdn.net/misskissC/article/details/38020151
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpggy1g0wz3cnv22j30dk07874k.jpg)
和期待的相符，没有目标文件的链接过程，.o 文件果然很干净，使用-d命令查看的话，可以只看我自己写的代码部分！但是没有 -g 的话没有源码。
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpggy1g0wz3qgssaj30uu0t3dpz.jpg)
同样和期待的相符，加了-g之后，成功出现源码
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpgly1g0wz49e2xnj30r30oyq8r.jpg)
#### 4、c代码变成.s文件，如何精确捕捉到我写的函数的内容？
额，我发现.s 文件还是非常干净的，没有什么特别多的其他文件，想要找哪个函数，前面都有名字的。 尝试是由-g会有什么区别码？
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpgly1g0wz4p1qm5j30hd0kt41y.jpg)
加了-g 参数后，生成的.s 文件果然多了很多不认识的东西，仔细找了下后，发现并没有看到源码的字符串，可能是以某种特殊的方式编码了？如图是对比，左侧是加了 -g的，而右侧是没有加的。
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpggy1g0wz509prej31el0k5wmu.jpg)
我们来验证以下，这个加了-g的.s文件，是否真的是包含了我源码的信息？

验证通过，确实有，哈哈
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpgly1g0wz5i91ybj30nn0scq9h.jpg)
#### 5、各种情况的编译失败是在编译的过程是哪一步？
在编译c语言的时候，通常是一步全编译，我们来尝试分部编译，探究不同错误的编译失败地点。
1、如果我只是写一个函数而没有main函数，可以进行到哪一步？
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpgly1g0wz5u0tlsj30er05s3yw.jpg)
编译成汇编代码居然就报错了！预处理的话还是可以的
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpggy1g0wzcl23cbj30vx05176q.jpg)

2、不小心没写分号 ;
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpgly1g0wz6h56eaj30pi09kq3z.jpg)
额，看来还是这个源码变成汇编的过程过程
![在这里插入图片描述](https://ws3.sinaimg.cn/large/005BYqpgly1g0wz6uqm52j30mu03kabl.jpg)
我突然想到，整个编译的大过程分为`预处理->编译->汇编->链接`，那么可能语法问题之类的都是在`编译`这个小过程被发现的吧。
#### 6、list命令用法
默认显示10行，可使用`list 1,1000` 来获取更多行的代码
使用`list +/-` 用以继续，和查看更前的源码
`set listsize 20` 设置显示行数
`show listsize` 查看显示行数

#### 7、删除断点：`d b`
查看断点：
`info b`
`info watch`

---
查表
![image](https://ws3.sinaimg.cn/large/005BYqpgly1g0wz788t0zj30u30gtjsb.jpg)