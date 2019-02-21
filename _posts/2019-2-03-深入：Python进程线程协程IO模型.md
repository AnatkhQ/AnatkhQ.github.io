---
layout:     post   				    # 使用的布局（不需要改）
title:      深入：Python进程线程协程IO模型 				# 标题 
subtitle:   理解万岁！ #副标题
date:       2019-2-3 				# 时间
author:     QQL 						# 作者
header-img: img/concert-1838412_1920.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 网络编程
---

### GIL
python中一个线程对于C语言中的一个线程  
GIL使得**同一时刻**只有一个线程在一个cpu上执行字节码，无法将多个线程映射到多个cpu上执行  
但是这并不代表有GIL线程就安全，就不用再加其他的锁  
因为GIL会根据执行的**字节码行数**以及**时间片段**，**遇到IO操作**主动释放GIL，导致一个线程的任务不一定执行完就切给另外的线程进行执行  


### 线程
对于io操作来说，多线程和多进程性能区别不大  

### condition条件变量
用于复杂的线程间同步，比如小爱和天猫精灵两个线程，天猫说一句话，小爱回答一句话，天猫回应一句话。。。。进行互相提问回答，就需要用到condition  

```
from threading import Condition
cond =  threading.Condition()
xiaoai = XiaoAi(cond)
tianmao = TianMao(cond)
```
启动顺序很重要  
在调用with cond之后才能调用wait()或notify()方法  
condition有两层锁，一把底层锁会在线程调用wait()的时候释放，上面的锁会在每次调用wait的时候分配一把并放入到队列cond的等待队列中，等待notify方法的唤醒  



### 进程池

```
pool.apply()  同步阻塞执行进程  
pool.apply_async()  异步非阻塞执行进程
```  
```
在调用pool.join()之前一定要调用pool.close(),让pool不再接受新的进程
```

### 线程池
```
from concurrent.futures import ThreadPoolExecutor,as_completed,wait,FIRST_COMPLETED
"""
线程池作用：
主线程中可以获取某一个线程的状态或者某一个任务的状态
当一个线程完成的时候我们主线程立即知道
futures可以让多线程和多进程编码接口一致
"""
import time

def get_html(times):
    time.sleep(times)
    print("get page {} success".format(times))
    return times

executor = ThreadPoolExecutor(max_workers=2)
# 通过submit函数提交执行的函数到线程池中，submit是非阻塞的
task1 = executor.submit(get_html, (3))  # 返回一个future类
task2 = executor.submit(get_html, (2))
print(task2.cancel())  # True/False 在执行中和执行完成是取消不了的，还没开始则可以结束
# done用于判定某个任务是否完成,属于future类的方法
print(task1.done())
# result可以获取task的执行结果,属于future类的方法
print(task1.result())

# 获取已经成功的task的返回
urls = [3,2,4]
all_task = [executor.submit(get_html, (url)) for url in urls]
wait(all_task)  # 等待所有线程执行完毕
wait(all_task, return_when=FIRST_COMPLETED)  # 第一个线程执行完毕后执行主线程

for future in as_completed(all_task):
    data = future.result()
    print("get {} page success".format(data))


```


### 进程和线程中的通信
共享全局变量的方法：  
多线程中适用，多个线程共享同一块内存空间  
多进程中不适用，多个进程在不同的内存空间    

整个python中有三个Queue  
```
from queue import Queue  # 用于线程中的通信
from multiprocessing import Queue  # 用于进程中的通信，但是无法用于进程中的multiprocessing进程池
from multiprocessing import Manage  # 用于multiprocessing中的Pool进行通信
Manage().Queue()  # 必须实例化才有Queue
```


进程：  
```
from multiprocessing import Process,Queue
进程中的Queue无法运用于 mutiprocessing中的Pool
from multiprocessing import Manager可用于进程池中的通信
queue = Manage().Queue()  # Manage实例化之后有一个Queue
```  

线程：  
```
import queue
```

### 并发、并行

并发指的是一个**时间段内**， 有几个程序在**同一cpu**运行，但是任意时刻只有一个程序在cpu上运行。因为cpu执行速度快。  

### 同步、异步
指消息间的通讯机制

同步指的是调用IO操作时，必须等待IO操作完成才返回的调用

异步是指代码调用IO操作时，不必等待IO操作就返回调用

### 阻塞、非阻塞
函数调用的机制

阻塞是指调用函数时当前线程被挂起

非阻塞指的是调用函数时当前线程不会被挂起，而是立即返回


### Unix下五中io模型
阻塞式I/O
阻塞不会消耗CPU

非阻塞式I/O

I/O多路复用（最常使用）
select,pool.epoll都是IO多路复用的机制。IO多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦一个描述符就绪，能够通知程序进行相应的读写操作。（相当于一个中介，把所有socket任务放到中介那里，中介进行打包发布，一旦有一个结果就返回给那个用户）但select，poll，epoll本质上都是同步I/O

信号驱动式I/O（可忽略）

异步I/O（POSIX的aio_系列函数）不是很成熟


### select、poll、epoll
**select**的缺点有以下三点，会导致效率下降：

1. 每次调用select都要将所有的fd（文件描述符），copy到你的内核空间
2. 遍历所有的fd，是否有数据访问
3. 最大连接数（1024），超出链接不再监听


**poll**：

与select一样，只是最大连接数没有限制



**epoll**不同于select和poll只有一个**回调函数**，epoll通过三个函数实现实现轮询socket：

1. 第一个函数：创建epoll句柄：将所有的fd（文件描述符），copy到你的内核空间，只copy一次
2. 回调函数：为所有fd绑定一个回调函数，一旦有数据访问，触发回调函数，回调函数将fd放入一个链表中（回调函数：某一个函数或者某一个动作，成功完成之后，会触发的函数）
3. 第三个函数：判断链表是否为空

epoll最大连接数没有上线

```
1.epoll并不代表一定比select好
在并发高的情况下，连接活跃度不是很高，epoll比select好（比如web开发，用户很容易关闭页面）
并发不高，同时连接活跃度高，select比epoll好（比如游戏开发，进了游戏就一直在游戏里）
```  


### 改变server为非阻塞
```
from socket import *
server = socket(AF_INET, SOCK_STREAM)
server.bind('127.0.0', 8080)
server.listen(5)
server.setblocking(False)  # 设置server为非阻塞
```

### 高并发三要素
事件循环 + 回调(驱动生成器/协程) + select/poll/epoll(I/O多路复用) 

### 协程就是一个可以暂停的函数
出现的原因：  
1. 回调模式编码复杂度高(select/poll/epoll)
2. 同步编程的并发性不高
3. 多线程编程需要线程间的数据同步(需要用到Lock)

解决的方法：  
1. 采用同步的方式去编写异步的代码
2. 使用单线程去切换任务：  
1.线程是由操作系统切换的，单线程切换意味着我们需要程序员自己去调度任务  
2.不再需要锁，并发性高，如果单线程内切换函数，性能远高于线程切换，并发性更高  

协程的作用：单线程实现并发，线程和进程都是内核级别的调度，必须由操作系统进行管理切换，都会消耗大量资源。而协程是函数级别的调度，由程序员自己进行管理切换   

我们需要一个可以暂停的函数，并且可以在适当的时候恢复该函数继续执行  

协程---->有多个入口的函数，可以暂停的函数，可以向暂停的地方传入值的函数


### socket编程中需要注意和优化的地方
服务端和客户端通信主要经过的IO请求有三个部分：  
```
conn.connect()  # 等待客户端的链接花费大量时间
conn.send()  # send的速度很快，但是发送大文件依然会比较慢
conn.recv()  # recv接收数据，要从操作系统的缓存中copy一份出来，也需要花费大量时间
```  
这三个部分都可进行解耦拆分，使用`select/poll/epoll`进行优化操作，实现多路I/O复用模型  

操作系统在用户态和内核态之间，大部分在内核态维护内存  
服务端和客户端交换数据主要在两部分中经过：  
操作系统的缓存和应用程序的内存  
请求到服务器，先把数据copy到操作系统的缓存中，再由应用程序获取操作系统的缓存从而拿到传输的数据  


### 生成器

生成器不止可以产生值，也可以接收值

```
def gen_func():
    # 1. 可以产出值 2.可以接受值（调用方传递过来的值）
    html = yield "http://baidu.com"
    print(html)  # 拿到send的值
    yield 2
    yield 3
    return "body"



if __name__ == "__main__":
    gen = gen_func()
    # 1.启动生成器的方式有两种：next()  和  send
    # url = next(gen)
    # 在调用send发送飞none之前，必须启动一次生成器：1.send(None) 2.next(func)
    url = gen.send(None)
    html = "bitch"
    # send方法可以传递值到生成器内部，同时还可以重启生成器，执行到下一个yield
    print(gen.send(html))  # 拿到下一个yield值=2
    print(next(gen))
    gen.throw(Exception, "download error")  # 给生成器抛进异常
    gen.close()  # 使用close方法后，再调用生成器则会抛出异常，异常指向当前yield，而不会指向下一个yield
    print(next(gen))
    print(next(gen))  # 生成器拿不到生成对象，则会抛出StopIteration的异常
```  

生成器的真实案例：  

```
# python3.3中添加了yield from语法
# 语法：yield from iterable  iterable指：list/dict/tuple/range。。。。
# 作用：简化iterable的yield操作

from itertools import chain
# chain用于多个数据进行链接

my_list = [1,2,3]
my_dict = {
    "n1":"http://baidu.com",
    "n2":"http://taobao.com"

}

for value in chain(my_list,my_dict, range(5,10)):
    print(value)

"""
输出value：把list的值和dict的key，还有生成器range的值进行链接
1
2
3
n1
n2
5
6
7
8
9
"""
# chain内部的实现机制
def my_chain(*args, **kwargs):
    for my_iterable in args:
        yield from my_iterable  # 等于循环可迭代对象，并把每一个元素进行yield
        # for value in my_iterable:
        #     yield value
for value in my_chain(my_list,my_dict, range(5,10)):
    print(value)
# chain内部使用生成器yield达到链接目的


# yield from 核心
def g1(gen):
    yield from gen


def main():
    g = g1(range(100))
    g.send(None)  # 启动生成器


# 1.main:调用方   g1委托生成器    gen子生成器
# yield from 会在调用方与子生成器之间建立一个双向通道
# 即g1中gen的返回值不再先给g1再给main，而是gen直接给main

```  
只要遇到I/O操作,就把I/O操作封装到一个函数或方法中，使用yield 进行抛出  



### python3.5后原生的协程
原生的协程是为了区分生成器和协程。否则容易混淆

```
# python3.5前定义协程本质上是定义一个生成器
# python3.5后有了原生的协程
# python为了将语义变得更加明确，就引入了async和await两个关键词用于定义原生的协程
# 如果用生成器定义协程，则会混乱，既可以用于当生成器，又可以用来当协程
import types

@types.coroutine  # 使用此装饰器装饰生成器，就可以用await进行调用
def download(url):
    yield url


async def downloader(url):
    return "boby"


async def download_url(url):
    # do somethings
    # yield 1  # 在async中不能使用yield和yield from 用于区分生成器

    # await=yield from  await只能出现在async中，且生成器不能由await抛出
    html = await downloader(url)
    # html = await download(url)  # 不能await生成器，使用装饰器@types.coroutine则可以await
    return html

if __name__ == "__main__":
    coro = download_url("http://www.imooc.com")
    # next(None)  # 原生的协程不能用next调用
    coro.send(None)

```  


### asyncio并发编程
解决异步IO高并发的核心模块





