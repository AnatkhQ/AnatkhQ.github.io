---
layout:     post   				    # 使用的布局（不需要改）
title:      Django restframework第一节 				# 标题 
subtitle:   初始restframework源码 #副标题
date:       2019-1-20 				# 时间
author:     QQL 						# 作者
header-img: img/girl-3954232_1920.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Django
---


# Django restframework第一节

1. **开发模式**  
- 普通开发方式（前后端放在一起写）
- 前后端分离


2. **后端开发**  
- 为前端提供URL（API/接口的开发）
- 注：永远返回HttpResponse


3. **Django FBV、CBV**  
```
FBV,function base view

	def users(request):
		user_list = ['alex','oldboy']
		return HttpResponse(json.dumps((user_list)))
					
CBV,class base view 

	路由：
		url(r'^students/', views.StudentsView.as_view()),
	
	视图：
		from django.views import View

		class StudentsView(View):

			def get(self,request,*args,**kwargs):
				return HttpResponse('GET')

			def post(self, request, *args, **kwargs):
				return HttpResponse('POST')

			def put(self, request, *args, **kwargs):
				return HttpResponse('PUT')

			def delete(self, request, *args, **kwargs):
				return HttpResponse('DELETE')
```  

4.**面向对象**  
封装：  
对同一类方法封装到类中  
```
class File:
	文件增删改查方法
	
Class DB:
	数据库的方法
```  
					
将数据封装到对象中  
```
class File:
	def __init__(self,a1,a2):
		self.a1 = a1 
		self.xxx = a2
	def get:...
	def delete:...
	def update:...
	def add:...
	
obj1 = File(123,666)
obj2 = File(456,999)
```  
私有属性/方法  
`self.__name = name`  

### FBV、CBV
CBV，基于反射实现根据请求方式不同，执行不同的方法。  
**原理**：  

url -> view方法 -> dispatch方法（反射执行自定义的其他：GET/POST/DELETE/PUT）--->执行后再次把执行的结果返回给dipatch返回给View--->传给url-->传给客户端  

**流程**：  
```
class StudentsView(View):
	# 本质上CBV发送请求先要获取请求方式，则始终要走dispatch,通过反射getattr(self, request.method)在自定义类的对象中获取请求方法，请求方式需要自己定义def get/def post....

	def dispatch(self, request, *args, **kwargs):
		print('before')  # 可以在dispatch实现其他操作，类似于装饰器的功能
		ret = super(StudentsView,self).dispatch(request, *args, **kwargs)
		print('after')  # 视图函数get/post/put/delete...执行完后执行操作
		return ret

	def get(self,request,*args,**kwargs):
		return HttpResponse('GET')

	def post(self, request, *args, **kwargs):
		return HttpResponse('POST')

	def put(self, request, *args, **kwargs):
		return HttpResponse('PUT')

	def delete(self, request, *args, **kwargs):
		return HttpResponse('DELETE')

继承（多个类共用的功能，为了避免重复编写）：
from django.views import View


class MyBaseView(object):
	def dispatch(self, request, *args, **kwargs):
		print('before')
		# super实现以继承顺序(__mro__可以查看顺序)查找父类的.dispatch()方法
		ret = super(MyBaseView,self).dispatch(request, *args, **kwargs)
		print('after')
		return ret

class StudentsView(MyBaseView,View):  # 从左到右继承顺序

	def get(self,request,*args,**kwargs):
		print('get方法')
		return HttpResponse('GET')

	def post(self, request, *args, **kwargs):
		return HttpResponse('POST')

	def put(self, request, *args, **kwargs):
		return HttpResponse('PUT')

	def delete(self, request, *args, **kwargs):
		return HttpResponse('DELETE')

class TeachersView(MyBaseView,View):

	def get(self,request,*args,**kwargs):
		return HttpResponse('GET')

	def post(self, request, *args, **kwargs):
		return HttpResponse('POST')

	def put(self, request, *args, **kwargs):
		return HttpResponse('PUT')

	def delete(self, request, *args, **kwargs):
		return HttpResponse('DELETE')

```  

### 面试题  
1. **django中间件**  
- process_request
- process_view
- process_response
- process_exception
- process_render_template

**中间键流程完整流程**：  
用户-->WSGI-->每个中间键(可以有多个)的process_request--->走完所有中间键的process_request将倒回来--->走每个中间键的process_view--->视图函数中处理业务逻辑-->template/ORM交互数据库--->如果视图函数中有render，走中间键process_render_template返回给用户/如果视图函数中抛出异常，走中间键process_exception返回给用户错误信息  


2. **使用中间件做过什么？**  
- 权限  
- 用户登录验证
- django的csrf是如何实现？
csrf放在中间键process_view中而不是precess_request，因为csrf要先在视图函数中查看访问的视图是否加了装饰器csrf_exempt
process_view方法
- 检查视图是否被 @csrf_exempt （免除csrf认证）
	- 装饰器需要导入： from django.views.decorators.csrf import csrf_exempt
- 去请求体或cookie中获取token


3. **不使用csrf认证**  
情况一：  
```
settings.py

MIDDLEWARE = [
	'django.middleware.security.SecurityMiddleware',
	'django.contrib.sessions.middleware.SessionMiddleware',
	'django.middleware.common.CommonMiddleware',
	'django.middleware.csrf.CsrfViewMiddleware', # 全站使用csrf认证
	'django.contrib.auth.middleware.AuthenticationMiddleware',
	'django.contrib.messages.middleware.MessageMiddleware',
	'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

views.py

from django.views.decorators.csrf import csrf_exempt
@csrf_exempt # 该函数无需认证
def users(request):
	user_list = ['alex','oldboy']
	return HttpResponse(json.dumps((user_list)))
```  

情况二：  
```
settings.py

MIDDLEWARE = [
	'django.middleware.security.SecurityMiddleware',
	'django.contrib.sessions.middleware.SessionMiddleware',
	'django.middleware.common.CommonMiddleware',
	#'django.middleware.csrf.CsrfViewMiddleware', # 全站不使用csrf认证
	'django.contrib.auth.middleware.AuthenticationMiddleware',
	'django.contrib.messages.middleware.MessageMiddleware',
	'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

views.py

from django.views.decorators.csrf import csrf_exempt,csrf_protect
@csrf_protect # 该函数需认证
def users(request):
	user_list = ['alex','oldboy']
	return HttpResponse(json.dumps((user_list)))
```  


### CBV小知识  
**CBV无法给方法(get/post/delete...)添加装饰器！！！**  
**解决办法**：  
csrf时需要使用  
`from django.utils.decorators import method_decorator`  
`@method_decorator(csrf_exempt)`  
装饰器给dispatch方法中（装饰单独方法(get/post/put...)无效）  


方式一：  
```
from django.views.decorators.csrf import csrf_exempt,csrf_protect
from django.utils.decorators import method_decorator
class StudentsView(View):
	
	@method_decorator(csrf_exempt)
	def dispatch(self, request, *args, **kwargs):
		return super(StudentsView,self).dispatch(request, *args, **kwargs)

	def get(self,request,*args,**kwargs):
		print('get方法')
		return HttpResponse('GET')

	def post(self, request, *args, **kwargs):
		return HttpResponse('POST')

	def put(self, request, *args, **kwargs):
		return HttpResponse('PUT')

	def delete(self, request, *args, **kwargs):
		return HttpResponse('DELETE')
```  
方式二：    
```
from django.views.decorators.csrf import csrf_exempt,csrf_protect
from django.utils.decorators import method_decorator

@method_decorator(csrf_exempt,name='dispatch')  # 装饰给类
class StudentsView(View):

	def get(self,request,*args,**kwargs):
		print('get方法')
		return HttpResponse('GET')

	def post(self, request, *args, **kwargs):
		return HttpResponse('POST')

	def put(self, request, *args, **kwargs):
		return HttpResponse('PUT')

	def delete(self, request, *args, **kwargs):
		return HttpResponse('DELETE')
```  

### 总结  
- 本质，基于反射来实现  
- 流程：路由，view，dispatch(反射)  
- 取消csrf认证（装饰器要加到dispatch方法上且method_decorator装饰）  


扩展：  
- csrf 
	- 基于中间件的process_view方法
	- 装饰器给单独函数进行设置（认证或无需认证）


### django rest framework框架
```
pip3 install djangorestframework
from rest_framework.views import APIView
```  
01.进入APIView源码，发现class APIView(View):pass  继承了View只是在原有的基础上，添加了新的功能    

02.找到APIView中的dispatch()方法，重写了原View的dispatch，给request对象，封装了新的属性    
```
def dispatch(self, request, *args, **kwargs):
	self.args = args
	self.kwargs = kwargs
	# 将原有的request 加工分装了新的request
	request = self.initialize_request(request, *args, **kwargs)
	self.request = request
	self.headers = self.default_response_headers  # deprecate?

	try:
		self.initial(request, *args, **kwargs)

		# Get the appropriate handler method
		if request.method.lower() in self.http_method_names:
			handler = getattr(self, request.method.lower(),
							  self.http_method_not_allowed)
		else:
			handler = self.http_method_not_allowed

		response = handler(request, *args, **kwargs)

	except Exception as exc:
		response = self.handle_exception(exc)

	self.response = self.finalize_response(request, response, *args, **kwargs)
	return self.response
```  


```
def initialize_request(self, request, *args, **kwargs):
	"""
	Returns the initial request object.
	"""
	parser_context = self.get_parser_context(request)

	return Request(
		request,
		parsers=self.get_parsers(),  # 拿到一个列表生成式，生成一个对象列表[obj1,obj2....]
		authenticators=self.get_authenticators(),  # 拿到一个列表生成式，生成一个对象列表[obj1,obj2....]
		negotiator=self.get_content_negotiator(),
		parser_context=parser_context
	)
```  

04.parsers和authenticators两个列表生成式  
找self.parser_classes，self为自定义的class对象  
当自己的class没有定义parser_classes找继承的父类APIView是否有parser_classes  
```
def get_parsers(self):
	return [parser() for parser in self.parser_classes]

def get_authenticators(self):
	return [auth() for auth in self.authentication_classes]
```  

05.在继承的父类APIView中找到parser_classes和authentication_classes  
```
class APIView(View):
	...
	renderer_classes = api_settings.DEFAULT_RENDERER_CLASSES
	parser_classes = api_settings.DEFAULT_PARSER_CLASSES
	authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES
	...
```  

06.在initialize_request返回的对象Request中  
```
class Request(object):
	def __init__(self, request, parsers=None, authenticators=None,
				 negotiator=None, parser_context=None):
		assert isinstance(request, HttpRequest), (
			'The `request` argument must be an instance of '
			'`django.http.HttpRequest`, not `{}.{}`.'
			.format(request.__class__.__module__, request.__class__.__name__)
		)

		self._request = request  # 原生的request分装成了_request
		self.parsers = parsers or ()
		self.authenticators = authenticators or ()  # .authenticators可进行验证
		........
	......
```  

authenticator认证仅使用：  
```
from django.views import View
from rest_framework.views import APIView
from rest_framework.authentication import BasicAuthentication
from rest_framework import exceptions
from rest_framework.request import Request

class MyAuthentication(object):
	def authenticate(self,request):
		token = request._request.GET.get('token')
		# 获取用户名和密码，去数据校验
		if not token:
			raise exceptions.AuthenticationFailed('用户认证失败')
		return ("alex",None)

	def authenticate_header(self,val):
		pass

class DogView(APIView):
	authentication_classes = [MyAuthentication,]

	def get(self,request,*args,**kwargs):
		print(request)
		print(request.user)
		ret  = {
			'code':1000,
			'msg':'xxx'
		}
		return HttpResponse(json.dumps(ret),status=201)

	def post(self,request,*args,**kwargs):
		return HttpResponse('创建Dog')

	def put(self,request,*args,**kwargs):
		return HttpResponse('更新Dog')

	def delete(self,request,*args,**kwargs):
		return HttpResponse('删除Dog')

```  
