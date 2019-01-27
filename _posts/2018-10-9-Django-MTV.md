---
layout:     post   				    # 使用的布局（不需要改）
title:      悟道Django--MTV 				# 标题 
subtitle:   初识框架 #副标题
date:       2018-10-09 				# 时间
author:     QQL 						# 作者
header-img: img/girl-1990347_1920.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Django
---

## http协议简介
HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于万维网（WWW:World Wide Web ）服务器与本地浏览器之间传输超文本的传送协议。    
**特点**：  
&emsp;基于TCP/IP  
&emsp;基于请求－响应模式  
&emsp;无状态保存  
&emsp;无连接  
**请求协议**  
![请求协议.png](https://i.loli.net/2018/12/13/5c120b8059c6b.png)  
请求方式: get与post请求
* GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如EditBook?name=test1&id=123456. POST方法是把提交的数据放在HTTP包的Body中.  
* GET提交的数据大小有限制（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制.  
* GET与POST请求在服务端获取请求数据方式不同。  
* GET方式提交数据，会带来安全问题，比如一个登录页面，通过GET方式提交数据时，用户名和密码将出现在URL上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码.  

**响应协议**  
![响应协议.png](https://i.loli.net/2018/12/13/5c120b805c15f.png)  
**协议格式**：请求首航\r\n请求头\r\n\请求头....r\n\r\n请求体  
其中：只有POST请求有请求体，GET请求的参数用&链接到请求首航的URL里  
**响应状态码**
```
状态代码有三位数字组成，第一个数字定义了响应的类别，共分五种类别:
1xx：指示信息--表示请求已接收，继续处理
2xx：成功--表示请求已被成功接收、理解、接受
3xx：重定向--要完成请求必须进行更进一步的操作
4xx：客户端错误--请求有语法错误或请求无法实现
5xx：服务器端错误--服务器未能实现合法的请求

常见状态码：
OK                        //客户端请求成功
Bad Request               //客户端请求有语法错误，不能被服务器所理解
Unauthorized              //请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用 
Forbidden                 //服务器收到请求，但是拒绝提供服务
Not Found                 //请求资源不存在，eg：输入了错误的URL
Internal Server Error     //服务器发生不可预期的错误
Server Unavailable        //服务器当前不能处理客户端的请求，一段时间后可能恢复正常

```
## WSGI接口协议
**封装了http请求协议解析数据的过程(切割\r\n进行拆分)和http响应协议封装数据的过程(拼接\r\n进行数据封装)，让程序员专注于web业务开发。python根据wsgi协议开发了wsgiref模块使开发更加方便**  
```
from wsgiref.simple_server import make_server

def application(environ, start_response):
    """
    回调函数，当客户端链接时调用该函数
    :param environ: 按着http协议解析数据
    :param start_response: 按着http协议组装数据
    :return:
    """
    start_response('200 OK', [('Content-Type', 'text/html')])
    return [b'<h1>Hello, World!</h1>']

if __name__ == '__main__':
	httpd = make_server('', 8080, application)  # 1:IP/2:端口/3:回调函数，当用户链接就调用回调函数
	
	print('Serving HTTP on port 8000...')
	# 等待用户链接，开始监听HTTP请求:conn,addr = socket.accept()
	httpd.serve_forever()
```  
**如何给用户返回动态内容？**  
* 自定义一套特殊的语法，进行替换  
* 使用开源工具**jinja2**，遵循其指定语法(模板语法)  

## MTV模型  
Django的MTV分别代表：  
* **Model(模型)**：负责业务对象与数据库的对象(ORM)  
* **Template(模版)**：负责如何把页面展示给用户  
* **View(视图)**：负责业务逻辑，并在适当的时候调用Model和Template  
* 此外，Django还有一个**urls分发器**，它的作用是将一个个URL的页面请求分发给不同的view处理，view再调用相应的Model和Template  

## Django基础篇  
### 基本配置及操作  
**01.查看Django版本**：`python -m django --version`  

**02.创建Django项目**：`django-admin startproject mysite`（cd到你要创建的文件下）  
 
**03.在mysite目录下创建应用**：`python manage.py startapp appname`  

**04.启动django项目**：`python manage.py runserver 8080`  
 
**05.两条数据库迁移命令即可在指定的数据库中创建表** ：  
* Django1.7**之后**：  
&emsp;`python manage.py makemigrations`  
&emsp;`python manage.py migrate`  
* 当models.py中新增了类，再次执行上面两行数据迁移命令，修改models表结构不用执行命令  

* Django1.7**之前**：  
&emsp;`pyhon manage.py syncdb`  
 
**06.清空静态服务器**：`python manage.py flush`  
* 此命令会询问是 yes 还是 no, 选择 yes 会把数据全部清空掉，只留下空表  
 
**07.创建超级用户**：`python manage.py createsuperuser`  
 
**08.修改用户密码可以用**： `python manage.py changepassword username`  

**09.导入导出数据**：  
&emsp;`python manage.py dumpdata > proname.json`  
&emsp;`python manage.py loaddata proname.json`  

**10.Django 项目环境终端**:`python manage.py shell`  
* 这个命令和 直接运行 python 进入 shell 的区别是：你可以在这个 shell 里面调用当前项目的 models.py 中的 API，对于操作数据的测试非常方便。  

**11.Django终端直接操作SQL语句**：`python mange.py dbshell`  
* Django 会自动进入在settings.py中设置的数据库，如果是 MySQL 或 postgreSQL,会要求输入数据库用户密码。在这个终端可以执行数据库的SQL语句。  

**12.查看所有命令**：`python manage.py `  

### 配置文件
**1.APPS注册**
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',  # 注册自己的应用名到installed_apps里
]
```  
**2.数据库**  
Django默认使用sqlite进行存储数据，如果要使用其他数据库需要在settings中进行配置  
```
DATABASES = {
    'default': {  # default为默认使用的数据库，可以在DATABASES中配置多个不同数据库
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'cnblog',
        'USER': 'root',
        'PASSWORD': 'xxxx',
        'HOST': '127.0.0.1',  # IP
        'POST': 3306  # 端口
    }
}
```  
由于Django内部连接MySQL时使用的是MySQLdb模块，而python3中还无此模块，所以需要使用pymysql来代替  
如下设置放置的与**project同名的配置**的` __init__.py`文件中  
```
import pymysql
pymysql.install_as_MySQLdb()
```  
**3.静态文件**：  
```
静态文件交由Web服务器处理，Django本身不处理静态文件。简单的处理逻辑如下(以nginx为例)：

              URI请求-----> 按照Web服务器里面的配置规则先处理，以nginx为例，主要求配置在nginx.
                             conf里的location

                         |---------->如果是静态文件，则由nginx直接处理

                         |---------->如果不是则交由Django处理，Django根据urls.py里面的规则进行匹配
```  
**以上是部署到Web服务器后的处理方式(生产环境)，为了便于开发，Django提供了在开发环境的对静态文件的处理机制，方法是这样**：  

**STATIC主要指的是如css,js,images这样文件**：  
```
STATIC_URL = '/static/'      # 别名/引用名
STATICFILES_DIRS = (
            os.path.join(BASE_DIR,"static"),  #实际名 ,即实际文件夹的名字
        )

'''

注意点1:
 django对引用名和实际名进行映射,引用时,只能按照引用名来,不能按实际名去找
        <script src="/statics/jquery-3.1.1.js"></script>
        ------error－－－－－不能直接用，必须用STATIC_URL = '/static/':
        <script src="/static/jquery-3.1.1.js"></script>

注意点2:
 STATICFILES_DIRS = (
    ("app01",os.path.join(BASE_DIR, "app01/statics")),
        )

 <script src="/static/app01/jquery.js"></script>

'''
```  
**media配置**  
```
# in settings:

MEDIA_URL="/media/"  # 让用户能够访问到media文件夹的路径
MEDIA_ROOT=os.path.join(BASE_DIR,"app01","media","upload")  # 好像是Django2.0之前？
MEDIA_ROOT = os.path.join(BASE_DIR, "media")  # 2.0后配置？
# MEDIA_ROOT效果：一旦配置了media，Django将把文件下载到media文件夹里

# in urls:
from django.views.static import serve
re_path(r'^media/(?P<path>.*)$', serve, {'document_root': settings.MEDIA_ROOT}),
```  
```
        静态文件的处理又包括STATIC和MEDIA两类，这往往容易混淆，在Django里面是这样定义的：
        
        MEDIA:指用户上传的文件，比如在Model里面的FileFIeld，ImageField上传的文件。如果你定义
        
        MEDIA_ROOT=c:\temp\media，那么File=models.FileField(upload_to="abc/")＃，上传的文件就会被保存到c:\temp\media\abc

        eg：
            class blog(models.Model):
                   Title=models.charField(max_length=64)
                   Photo=models.ImageField(upload_to="photo")
          上传的图片就上传到c:\temp\media\photo，而在模板中要显示该文件，则在这样写
          在settings里面设置的MEDIA_ROOT必须是本地路径的绝对路径，一般是这样写:
                 BASE_DIR= os.path.abspath(os.path.dirname(__file__))
                 MEDIA_ROOT=os.path.join(BASE_DIR,'media/').replace('\\','/')

        MEDIA_URL是指从浏览器访问时的地址前缀，举个例子：
            MEDIA_ROOT=c:\temp\media\photo
            MEDIA_URL="/data/"
        在开发阶段,media的处理由django处理：

           访问http://localhost/data/abc/a.png就是访问c:\temp\media\photo\abc\a.png

           在模板里面这样写<img src="/media/abc/a.png">

           在部署阶段最大的不同在于你必须让web服务器来处理media文件，因此你必须在web服务器中配置，
           以便能让web服务器能访问media文件
           以nginx为例，可以在nginx.conf里面这样：

                 location ~/media/{
                       root/temp/
                       break;
                    }

           具体可以参考如何在nginx部署django的资料。
```  
### 路由系统  
### 视图函数views  
### 模板层template
上述详情均看本人[简书文章](https://www.jianshu.com/p/0fad8665cb8e)
