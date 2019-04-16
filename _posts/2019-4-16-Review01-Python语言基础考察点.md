---
layout:     post   				    # 使用的布局（不需要改）
title:      Review01-Python语言基础考察点				# 标题 
subtitle:   Review01 #副标题
date:       2019-4-16 				# 时间
author:     QQL 						# 作者
header-img: img/conservatory-1031494_1920.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - python
---

## python语言特性

### python动态的强类型语言（不少人会误以为是弱类型语言）

动态还是静态指的是**编译期间**还是**运行期间**确定类型

强类型指的是**不会发生隐式转换**（例如JS中就会出现隐式转换所以是弱类型语言，python如果类型不一致将会报错）  

**1.动态语言的鸭子类型**

- 关注点在对象的行为，而不是类型  
- 比如file,StringIO,socket对象都支持read/write方法
- 比如定义了__iter__魔术方法的对象可以用for进行遍历

**鸭子类型更关注接口而不是类型**

**2.动态语言之monkey patch猴子补丁**

- 所谓的monkey patch就是运行时替换
- 比如gevent库需要修改内置的socket(把内置的阻塞socket替换成非阻塞socket)
- `from gevent import monkey;mokey.patch_socket()`

![monkey_patch.png](https://i.loli.net/2019/04/16/5cb56267c4056.png)  


**3.自己实现monkey patch**  
本质上就是把阻塞socket替换成非阻塞socket，实现原理：  
```
import time
print(time.time())  # 获取时间戳

def _time():
	return 123

time.time = _time
print(time.time())  # 返回123实现了对时间戳的替换
```  

### python3改进

- print成为函数，有更多参数选择例如end
- 编码问题。python3不再有Unicode对象，默认str为unicode
- 除法变化。python3除法返回浮点数，而不是python2中的地板除
- 类型注解。帮助IDE实现类型检查
- 优化super()方便直接调用父函数
- 高级解包操作。`a,b,*rest = range(10)`得到`a=0,b=1,剩下的给rest`
- Keyword only arguments。限定关键字参数
- Chained exceptions。Python3重新抛出异常不会丢失栈信息
- 一切返回迭代器range,zip,map,dict.values

**1.显示关键字参数**  
在形参中加入*让后续的参数必须制定关键字才能传参，否则会报错  
```
def add(a,b,*,c):
	return a+b+c

print(add(1,2,3))  # 报错
print(add(1,2,c=3))  # *号后的参数必须使用关键字传参
```


**2.保留栈信息抛出异常**  
方便排错  
![chain_exception.png](https://i.loli.net/2019/04/16/5cb565d804b78.png)  

### python3新增

- yield from 链接子生成器(yield加强版)
- asyncio内置库，async/await原生协程支持异步编程
- 新的内置库：enum,mock,asyncio,ipaddress,concurrent.futures等
- 生成的pyc文件统一放到__pycache__
- 一些内置库的修改。例如urllib,selector
- 性能优化

### python2/3工具
如何让Python2和Python3兼容  
需要熟悉一些兼容2/3工具  
- six模块
- 2to3等工具转换代码
- `__future__`

## Python函数

### python如何传递参数
**一个容易混淆的问题**  
- 传递值还是引用（C的想法）？都不是。唯一的传递方式是**共享传递**
- Call by sharing(共享传参)。函数形参获取实参中各个引用的副本

```
# 传递可变对象，传递的参数l指向ll，修改同一个值
def flist(l):
	l.append(0)
	print(l)
ll = []
flist(ll)  # [0]
flist(ll)  # [0,0]

# 传递不可变对象，传递的参数s指向ss，但因为是不可变对象，所以在传递后会创建一个ss的副本，s指向ss
def fstr(s):
	s+='a'
	print(s)
ss = 'hehe'
fstr(ss)  # hehea
fstr(ss)  # hehea
```  

### python可变/不可变对象
- 不可变：bool/int/float/tuple/str/frozentset
- 可变：list/set/dict

**不可变对象当做默认参数时**  
**默认参数只计算一次，即定义时已经固定**  
```
def flist(l=[1]):
	l.append(1)
	print(l)
flist()  # [1,1]
flist()  # [1,1,1]
```

## python异常
python使用异常处理错误（有些语言使用错误码）  
- BaseException 所有异常继承的父类
- SystemExit/KeyboardInterrupt/GeneratorExit  系统相关异常
- Exception 其他所有异常继承自Exception

### 使用异常的常见常见
**什么时候需要捕获处理异常？**  
- 网络请求（超时、连接错误等）
- 资源访问（权限问题、资源不存在）
- 代码逻辑（越界访问、KeyError等）

### 如何处理python异常
```
try:
	# func  # 可能会抛出异常的代码
except(Exception1,Exception2) as e  # 可以捕获多个异常并处理
	# 异常处理代码
else:
	# pass    # 异常没有发生时走的代码
finally:
	pass      # 无论异常有没有发生都走的代码，一般处理资源的管理和释放
```

### 自定义异常
- 继承Exception实现自定义异常
- 给异常加上一些附加信息
- 处理一些业务相关的特定异常(raise MyException)

```
class MyException(Exception):  # 继承Exception
	pass

try:
	raise MyException('my exception')
except MyException as e:
	print(e)  # my exception
```

## Python性能分析与优化，GIL锁
### Cpython GIL
- Cpython解释器的内存管理并不是线程安全的
- 保护多线程情况下对Python对象的访问
- Cpython使用简单的所机制避免了多个线程同时执行字节码

**GIL影响**  

限制了程序的多核执行
- 同一时间只能有一个线程执行字节码
- CPU密集程序难以利用多核优势
- IO期间会释放GIL，对IO密集程序影响不大

**规避GIL影响**  

**区分CPU和IO密集型程序**  
- CPU密集可以使用多进程+进程池
- IO密集使用多线程/协程
- cpython扩展

### 如何剖析程序性能
使用各种profile工具（内置或第三方）  
- 二八定律，大部分时间耗时在少量代码
- 内置的profile/cprofile等工具
- 使用pyflame(uber开源项目)的火焰图工具

**自行学习python优化**  

### 服务端性能优化措施

**Web应用一般语言不会成为瓶颈**  
- 数据结构与算法优化
- 数据库底层：索引优化、慢查询消除、批量操作减少IO、NoSQL
- 网络IO：批量操作，pipline操作减少IO
- 缓存：使用内存数据库redis
- 异步：asyncio，celery
- 并发：gevnet、多线程

## Python生成器与协程
### Generator生成器
- 生成器是可以生成值的函数
- 当一个函数里有了yield关键字就成了生成器
- 生成器可以挂起执行并保持当前执行的状态

### 基于生成器的协程
Python3之前没有原生协程，只有基于生成器的协程

- pep342增强了生成器功能
- 生成器可以通过yield暂停执行和产出数据
- 同时支持send()向生成器发送数据和throw()向生成器抛异常

![基于生成器的协程.png](https://i.loli.net/2019/04/16/5cb57685a8696.png)  

### 协程注意点
- 协程需要使用send(None)或者next(coroutine)来预激才能启动
- 在yield处协程会暂停执行
- 单独的yield value会产出值给调用方
- 可以通过coroutine.send(value)来给协程发送值，发送的值会赋值给yield表达式左边的变量
- 协程执行完成后(没有遇到下一个yield语句)会抛出StopIteration异常

### 协程装饰器
避免每次都要使用send预激它  
```
from functools import wraps
def coroutine(func):  # 这样就不用每次都用send()进行激活
"""
装饰器：向前执行到第一个yield表达式，预激func
"""
	@wraps(func)
	def primer(*args,**kwargs):
		gen = func(*args,**kwargs)
		next(gen)  # 预激了协程函数
		return gen  # 返回已经激活了的协程函数就不用再使用send(None)或next一次进行激活
	return primer

```  

### Python3原生协程
Python3.5引入async/await支持原生协程  
![python原生协程.png](https://i.loli.net/2019/04/16/5cb5792de132d.png)  

## Python单元测试

### 什么是单元测试
Unit Testing  
- 针对程序模块进行正确性检验
- 一个函数，一个类进行验证
- 自底向上保证程序正确性

### 单元测试相关库
- nose/pytest较为常见
- mock模块用来模拟替换网络请求等
- coverage统计测试覆盖率

**使用pytest进行模块的测试**  
**在终端输入pytest 文件名.py 会自动检测以test开头的函数测试样例**  
![pytest使用样例.png](https://i.loli.net/2019/04/16/5cb57bb119227.png)  

**在修改代码时，通常通过跑一边单元测试便可知道之前的代码有没有改坏。实现回归测试**  

## Python中创建二维数组

如果在Python中想要创建一个二维数组，我们该如何写呢？  
```
>>> A = [0]* 3 * 4
>>> B = [[0]*3] * 4
```  

是A还是B呢？当然是B了！还是先输出看一下：  

```
>>> A
[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
>>> B
[[0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0]]
```  

不出所料，我们应该按照B = [[0]*3]*4来创建二维数组。   
！！！！！！！！BUT！！！！！！！！！   
当你按照上述方式来创建二维数组的时候，如果你对二维数组中的一个数进行改变，会输出什么呢？我们来试一下，比如我们把第一行的第二个数字改为2，B[0][1] = 2，输出：

```
>>> B
[[0, 2, 0], [0, 2, 0], [0, 2, 0], [0, 2, 0]]
```  
为什么会是这样？！！   
因为list在Python中是个可变类型啊亲！按照B = [[0]*3]*4来创建二维数组只是4个指向这个空列表元素的引用,修改任何一个元素都会改变整个列表的。   
另一个栗子：  
```
>>> A = [0]*3
>>> B = A
>>> B[0] = 1
>>> A
[1, 0, 0]
```  
所以，在Python中创建二维数组应该这样写：  

```
>>> C = [[0]*3 for i in range(4)]
>>> C 
[[0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0]]
>>> C[0][1] = 2
>>> C
[[0, 2, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

所以下次在Python中创建二维数组时候记住了：  

`aList = [[0] * cols for i in range(rows)]`  

