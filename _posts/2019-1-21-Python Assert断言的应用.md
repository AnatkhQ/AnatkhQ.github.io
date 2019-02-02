---
layout:     post   				    # 使用的布局（不需要改）
title:      Python Assert断言的应用 				# 标题 
subtitle:   断言的使用语法及使用场景 #副标题
date:       2019-1-21 				# 时间
author:     QQL 						# 作者
header-img: img/computer-1209641_1920.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - python
---

## python中断言assert的使用

### 断言的使用场景

在没完善一个程序之前，我们不知道程序在哪里会出错，与其让它在运行时崩溃(特别是python这种解释型语言，把整个项目翻译完后才进行代码运行)，不如在出现错误条件时就崩溃（返回错误）。这时候断言assert 就显得非常有用。  

### 断言的作用

python assert断言是声明布尔值必须为真的判定，当这个关键字后边的条件为假的时候，程序自动崩溃并抛出AssertionError的异常  
可以理解assert断言语句为`raise-if-not`，用来测试表示式，其返回值为假，就会触发异常。  
**一般来说我们可以用assert在程序中置入检查点，当需要确保程序中的某个条件一定为真才能让程序正常工作的话，assert关键字就非常有用了**。  

### assert的语法格式

> assert expression

它的等价语句为：  
> if not expression:  
  &emsp;raise AssertionError


1. assert语句用来声明某个条件是真的。
2. 如果你非常确信某个你使用的列表中至少有一个元素，而你想要检验这一点，并且在它非真的时候引发一个错误，那么assert语句是应用在这种情形下的理想语句。
3. 当assert语句失败的时候，会引发一AssertionError。


下面做一些assert用法的语句供参考：  
```
>>assert 1==1
>> assert 1 == 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError

>>assert 2+2==2*2
>>assert len(['my boy',12])<10
>>assert range(4)==[0,1,2,3]
>>> mylist = ['item']
>>assert len(mylist) >= 1
>>mylist.pop()
'item'
>>assert len(mylist) >= 1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AssertionError
```  

### 如何为assert断言语句添加异常参数

assert的异常参数，其实就是在断言表达式后添加字符串信息，用来解释断言并更好的知道是哪里出了问题。格式如下：  

1. `assert 表达式 [, arguments]`
2. `assert 表达式 [, 参数]`

python源码里用到了很多assert




