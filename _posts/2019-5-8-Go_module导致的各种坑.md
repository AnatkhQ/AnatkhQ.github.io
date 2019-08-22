---
layout:     post   				    # 使用的布局（不需要改）
title:      Go_module导致的各种坑 				# 标题 
subtitle:   排坑 #副标题
date:       2019-5-8 				# 时间
author:     QQL 						# 作者
header-img: img/music-3888684_1920.png 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - go
---


### go module 介绍

自Go1.1.1版本发布(2018-08-24发布)，从官方的博客中看到，其中有个比较突出的特色就是module，模块概念。

module是一个相关Go包的集合，它是源代码更替和版本控制的单元。模块由源文件形成的go.mod文件的根目录定义，包含go.mod文件的目录也被称为模块根。moudles取代旧的的基于GOPATH方法来指定在工程中使用哪些源文件或导入包。模块路径是导入包的路径前缀，go.mod文件定义模块路径，并且列出了在项目构建过程中使用的特定版本。


### 具体使用步骤

1. 首先将你的Go更新到1.11以上版本
2. 在当前项目下，系统环境变量GO111MODULE=on

> mac/linux：export GO111MODULE=on

> windows：set GO111MODULE=on

> 也可以直接在全局设置系统环境变量：

![gomodule环境变量.png](https://i.loli.net/2019/08/22/mYZaMu2qd83swRb.png)


3. 执行命令`go mod init [模块名]` 在当前目录下生成一个go.mod文件
4. 如果你工程中存在一些不能确定版本的包，那么生成的go.mod文件可能就不完整，因此继续执行下面的命令
5. 执行`go mod tidy`命令，它会添加缺失的模块以及移除不需要的模块。执行后会生成go.sum文件(模块下载条目)。添加参数-v，例如go mod tidy -v可以将执行的信息，即删除和添加的包打印到命令行；

6. 执行命令`go mod verify`来检查当前模块的依赖是否全部下载下来，是否下载下来被修改过。如果所有的模块都没有被修改过，那么执行这条命令之后，会打印all modules verified。

7. 执行命令`go mod vendor`生成vendor文件夹，该文件夹下将会放置你go.mod文件描述的依赖包，文件夹下同时还有一个文件modules.txt，它是你整个工程的所有模块。在执行这条命令之前，如果你工程之前有vendor目录，应该先进行删除。同理go mod vendor -v会将添加到vendor中的模块打印出来；

### 总结
1. **设置了go&nbsp;module之后，每创建一个项目都必须使用`go mod init`，否则`package main`该行将会报错**
2. **必须使用`go mod tidy`生成go.sum**，使用后IDE才会有代码提示，第三方包才能引入进来

### 排坑
#### 1.Vscode对`go module`的支持非常不好

目前使用`go module`会导致代码没有提示补全，扩展程序出错等等问题！！  
不建议用了go module后使用vscode，建议使用goland(网上有很多激活码这里就不提供了)。

#### 2.gocode出错

![gocode解决方法.png](https://i.loli.net/2019/08/22/AKHJ7pjxlXcNMno.png)

#### 3.goland出现代码无法提示，包无法引入，go module出错等

打开上方菜单：File>>Settings

> goland代码无法提示or包无法引入

![filewatchers配置.png](https://i.loli.net/2019/08/22/gxr52q1DWabt9Pm.png)

点击上述按钮，选择默认配置即可。有时不添加任何扩展也能出现代码提示等。。。总之非常坑人

> go module出错

![gomodule设置.png](https://i.loli.net/2019/08/22/IiWpsKQgrbLudU7.png)

只需要选择check就行  
代理可以自行配置  

#### 4.待续....



 



