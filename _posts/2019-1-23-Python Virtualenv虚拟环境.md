---
layout:     post   				    # 使用的布局（不需要改）
title:      Python Virtualenv虚拟环境全网最细攻略 				# 标题 
subtitle:   百解python虚拟环境 #副标题
date:       2019-1-23 				# 时间
author:     QQL 						# 作者
header-img: img/computer-1209641_1920.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - python
---

# python虚拟环境Virtualenv全网最细攻略

### 前言：为什么要使用虚拟环境

在实际项目开发中，我们通常会根据自己的需求去下载各种相应的框架库，如Scrapy、Beautiful Soup等，但是可能每个项目使用的框架库并不一样，或使用框架的版本不一样，这样需要我们根据需求不断的更新或卸载相应的库。直接怼我们的Python环境操作会让我们的开发环境和项目造成很多不必要的麻烦，管理也相当混乱。如一下场景：  

场景1：项目A需要某个框架1.0版本，项目B需要这个库的2.0版本。如果没有安装虚拟环境，那么当你使用这两个项目时，你就需要 来回 的卸载安装了，这样很容易就给你的项目带来莫名的错误；  

场景2：公司之前的项目需要python2.7环境下运行，而你接手的项目需要在python3环境中运行，想想就应该知道，如果不使用虚拟环境，这这两个项目可能无法同时使用，使用python3则公司之前的项目可能无法运行，反正则新项目运行有麻烦。而如果虚拟环境可以分别为这两个项目配置不同的运行环境，这样两个项目就可以同时运行。  

Tips:其实虚拟环境好处也确实比较多，会给我们项目的开发带来许多的好处，但是初学者，建议还是不要这么折腾，我们的首要目的是更快的掌握更多的知识，研究virtualenv会花费一些额外的经历，而且意志不强的同学很容易遭受打击，但是这个优点我们还是要记下来的方便以后要用的时候能很快的想起。  

### 命令行中virtualenv的使用

#### 1.手动建立虚拟环境

**Windows cmd**：  
`pip install virtualenv`  

**创建虚拟环境目录 env**：  
通过上面的步骤安装成功之后，我们就可以创建虚拟环境了：`virtualenv 虚拟环境名`  

这个命令创建虚拟环境，会在**当前所在目录**进行创建，如`C:\Users\Smalu(电脑管理者路径)`  

建议`cd` 到一个专门创建虚拟环境的目录下，进行创建虚拟环境  
**语法**：`virtualenv envname`   
**效果**：把当前环境下的所有包都打包给新的虚拟环境(**通常不会使用此语法进行虚拟环境的创建！**)  


**语法**：`virtualenv --no-site-packages envname`  
**效果**：创建了一个全新的无任何pip包的虚拟环境(**通常使用此语法**)  



#### 2.进入虚拟环境

**windows**:  
先要进入`cd`到虚拟环境的位置（目录）的Scripts中，然后在激活（activate.bat/activate）虚拟环境，则进入新建的虚拟环境中了  
**命令**：  
```
cd envname\script
activate
```  
命令行变为`(envname) 路径>` 的格式这说明已经成功进入  

---

**linux**：  
启动虚拟环境  
`source env/bin/activate`

#### 3.退出虚拟环境
执行Scirpt目录中的deactivate.bat，则退出虚拟环境  
命令行没有(envname)  

###  pycharm中virtualenv的使用

#### 1.在pycharm中使用已经在本地创建过了的虚拟环境(例如在命令行先创建，再到pycharm中使用)  
![步骤一.png](https://i.loli.net/2019/02/09/5c5e5675a1a6a.png)  
![步骤二.png](https://i.loli.net/2019/02/09/5c5e5674e7347.png)  
完成上述操作后该项目就使用所选虚拟环境  


#### 2.直接在pycharm中创建虚拟环境并使用   
![步骤三.png](https://i.loli.net/2019/02/09/5c5e5674e8b0e.png)  
![步骤四.png](https://i.loli.net/2019/02/09/5c5e56753c5ab.png)  


### 在新的virtualenv项目中处理依赖包  
通常做项目时，会在文档中标示此项目所需要的包有哪些  
我们把需要用到的包的requirements.txt放到自己创建的虚拟目录envname下，pycharm将会自动识别，点击『Install requirements』安装相应的 package  
![步骤五.png](https://i.loli.net/2019/02/09/5c5e58b477354.png)  
**但是此方式，会导致某些包安装不上，所以我建议在命令行中进入虚拟环境，使用命令**：  
`pip install -r requirements.txt`  
进行安装，确保没有包遗漏  

---

如果我们需要把已有项目传给其他人继续完成，我们需要把该项目依赖的包列出来打包成requirements.txt  
有虚拟环境就进入虚拟环境，使用：`(venv) $ pip freeze >requirements.txt`  
没有虚拟环境，使用：`pip freeze > requirements.txt`  

### 虚拟环境升级版virtualenvwrapper-win  
通过上面的步骤其实我们就已经完成虚拟环境virtualenv的安装和使用了，但是认真的你肯定发现了上面需要记住每一个虚拟环境的目录，才能进入虚拟环境并操作，很麻烦，下面我们通过另一个配置来简化我们的使用  

#### 1.安装virtualenvwrapper-win
`pip install virtualenvwrapper-win`  

#### 2.检查是否安装成功  
在命令行中输入：`workon`  
![步骤六.png](https://i.loli.net/2019/02/09/5c5e5a98155c3.png)  
出现这个这说明安装成功！  

#### 3.配置虚拟环境安装目录
给你虚拟环境安装目录设置一个专门（你想放）的目录  

通过上面的步骤，创建虚拟环境，默认放在C:\Users\电脑用户名\Envs目录中  

这样可能有时候不满足我们的需求，比如我们想把项目放在其他盘（或其他位置），这样就需要我们自己配置一下环境  

新建一个自己专门存放虚拟环境的目录  
例如：`D:\Envs`  

按照下面操作进行环境变量的配置  
![步骤七.png](https://i.loli.net/2019/02/09/5c5e5c0cf2e47.png)  

通过设置WORKON_HOME路径，就给我们的python虚拟环境指定了一个存放位置：

再次运行workon，目录中没有虚拟环境了，因为默认目录已经改变

那么我们可以将之前的虚拟环境的项目拷贝到新建目录下

再次运行workon，就可以看到该目录下所有的虚拟空间了

![步骤吧.png](https://i.loli.net/2019/02/09/5c5e5c902b2b4.png)  


#### 使用某个虚拟环境

`workon #列出所以目录下的空间名`  

`workon py3entest #使用名为py3entest的虚拟空间`  

新建虚拟空间的方法依然是：`mkvirtualenvs 空间名`  
创建后将自动进入该虚拟环境  

退出虚拟环境`deactivate`  

#### workon常用命令

> 列出虚拟环境列表：workon

> 新建虚拟环境：mkvirtualenv [虚拟环境名称] ->应该就是make的简写方便理解  

> 启动/切换虚拟环境：workon [虚拟环境名称]

> 离开虚拟环境：deactivate






