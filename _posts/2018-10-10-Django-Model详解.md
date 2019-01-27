---
layout:     post   				    # 使用的布局（不需要改）
title:      悟道Django--Model 				# 标题 
subtitle:   Model详解 #副标题
date:       2018-10-10 				# 时间
author:     QQL 						# 作者
header-img: img/girl-1990347_1920.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Django
---

## ORM
**映射关系**：
* 表名 <------------------>  类名  
* 字段 <------------------>  属性  
* 表记录 <------------------>  类示例对象  

## 创建表（建立模型）
**在settings中配置LOGGING可以查看翻译成的SQL语句**  
```
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console':{
            'level':'DEBUG',
            'class':'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level':'DEBUG',
        },
    }
}　　
```  
### 在创建表模型时的注意事项：  
1、 表的名称myapp_modelName，是根据 模型中的元数据自动生成的，也可以覆写为别的名称　 

2、id 字段是自动添加的  

3、对于外键字段，Django 会在字段名上添加"_id" 来创建数据库中的列名  

4、这个例子中的CREATE TABLE SQL 语句使用PostgreSQL 语法格式，要注意的是Django 会根据settings 中指定的数据库类型来使用相应的SQL 语句。  

5、定义好模型之后，你需要告诉Django _使用_这些模型。你要做的就是修改配置文件中的INSTALL_APPSZ中设置，在其中添加models.py所在应用的名称。  

6、外键字段 ForeignKey 有一个 null=True 的设置(它允许外键接受空值 NULL)，你可以赋给它空值 None 。  

7、默认使用的数据库是sqlite3  
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```  
**如果使用其他数据库**：  
1.在settings文件中配置对应的数据库：如mysql  
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',#对应的数据库
        'NAME': "newblog",#自己数据名称
        "USER":"root",#用户名
        "PASSWORD":"root",#密码
        "HOST":"",#主机好 默认本机
        "PORT":"3306"#端口
    }
}
```  
2.models对应文件中的__init__.py中添加如下代码  
```
import pymysql
pymysql.install_as_MySQLdb()
```  
### models属性
```
    AutoField(Field)
        - int自增列，必须填入参数 primary_key=True

    BigAutoField(AutoField)
        - bigint自增列，必须填入参数 primary_key=True

        注：当model中如果没有自增列，则自动会创建一个列名为id的列
        from django.db import models

        class UserInfo(models.Model):
            # 自动创建一个列名为id的且为自增的整数列
            username = models.CharField(max_length=32)

        class Group(models.Model):
            # 自定义自增列
            nid = models.AutoField(primary_key=True)
            name = models.CharField(max_length=32)

    SmallIntegerField(IntegerField):
        - 小整数 -32768 ～ 32767

    PositiveSmallIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
        - 正小整数 0 ～ 32767
    IntegerField(Field)
        - 整数列(有符号的) -2147483648 ～ 2147483647

    PositiveIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
        - 正整数 0 ～ 2147483647

    BigIntegerField(IntegerField):
        - 长整型(有符号的) -9223372036854775808 ～ 9223372036854775807

    自定义无符号整数字段

        class UnsignedIntegerField(models.IntegerField):
            def db_type(self, connection):
                return 'integer UNSIGNED'

        PS: 返回值为字段在数据库中的属性，Django字段默认的值为：
            'AutoField': 'integer AUTO_INCREMENT',
            'BigAutoField': 'bigint AUTO_INCREMENT',
            'BinaryField': 'longblob',
            'BooleanField': 'bool',
            'CharField': 'varchar(%(max_length)s)',
            'CommaSeparatedIntegerField': 'varchar(%(max_length)s)',
            'DateField': 'date',
            'DateTimeField': 'datetime',
            'DecimalField': 'numeric(%(max_digits)s, %(decimal_places)s)',
            'DurationField': 'bigint',
            'FileField': 'varchar(%(max_length)s)',
            'FilePathField': 'varchar(%(max_length)s)',
            'FloatField': 'double precision',
            'IntegerField': 'integer',
            'BigIntegerField': 'bigint',
            'IPAddressField': 'char(15)',
            'GenericIPAddressField': 'char(39)',
            'NullBooleanField': 'bool',
            'OneToOneField': 'integer',
            'PositiveIntegerField': 'integer UNSIGNED',
            'PositiveSmallIntegerField': 'smallint UNSIGNED',
            'SlugField': 'varchar(%(max_length)s)',
            'SmallIntegerField': 'smallint',
            'TextField': 'longtext',
            'TimeField': 'time',
            'UUIDField': 'char(32)',

    BooleanField(Field)
        - 布尔值类型

    NullBooleanField(Field):
        - 可以为空的布尔值

    CharField(Field)
        - 字符类型
        - 必须提供max_length参数， max_length表示字符长度

    TextField(Field)
        - 文本类型

    EmailField(CharField)：
        - 字符串类型，Django Admin以及ModelForm中提供验证机制

    IPAddressField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供验证 IPV4 机制

    GenericIPAddressField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供验证 Ipv4和Ipv6
        - 参数：
            protocol，用于指定Ipv4或Ipv6， 'both',"ipv4","ipv6"
            unpack_ipv4， 如果指定为True，则输入::ffff:192.0.2.1时候，可解析为192.0.2.1，开启刺功能，需要protocol="both"

    URLField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证 URL

    SlugField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证支持 字母、数字、下划线、连接符（减号）

    CommaSeparatedIntegerField(CharField)
        - 字符串类型，格式必须为逗号分割的数字

    UUIDField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供对UUID格式的验证

    FilePathField(Field)
        - 字符串，Django Admin以及ModelForm中提供读取文件夹下文件的功能
        - 参数：
                path,                      文件夹路径
                match=None,                正则匹配
                recursive=False,           递归下面的文件夹
                allow_files=True,          允许文件
                allow_folders=False,       允许文件夹

    FileField(Field)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage

    ImageField(FileField)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage
            width_field=None,   上传图片的高度保存的数据库字段名（字符串）
            height_field=None   上传图片的宽度保存的数据库字段名（字符串）

    DateTimeField(DateField)
        - 支持的格式
        ['%b %d %Y',      # 'Oct 25 2006'
		 '%b %d, %Y',     # 'Oct 25, 2006'
		 '%d %b %Y',      # '25 Oct 2006'
		 '%d %b, %Y',     # '25 Oct, 2006'
		 '%B %d %Y',      # 'October 25 2006'
		 '%B %d, %Y',     # 'October 25, 2006'
		 '%d %B %Y',      # '25 October 2006'
		 '%d %B, %Y']     # '25 October, 2006'


    DateField(DateTimeCheckMixin, Field)
        - 支持的输入格式  
		['%Y-%m-%d',      # '2006-10-25'
		 '%m/%d/%Y',      # '10/25/2006'
		 '%m/%d/%y']      # '10/25/06'

    TimeField(DateTimeCheckMixin, Field)
        - 时间格式      HH:MM[:ss[.uuuuuu]]

    DurationField(Field)
        - 长整数，时间间隔，数据库中按照bigint存储，ORM中获取的值为datetime.timedelta类型

    FloatField(Field)
        - 浮点型

    DecimalField(Field)
        - 10进制小数
        - 参数：
            max_digits，小数总长度
            decimal_places，小数位长度

    BinaryField(Field)
        - 二进制类型
```  
### 常用字段参数
```
(1)null

如果为True，Django 将用NULL 来在数据库中存储空值。 默认值是 False.

(1)blank

如果为True，该字段允许不填。默认为False。
要注意，这与 null 不同。null纯粹是数据库范畴的，而 blank 是数据验证范畴的。
如果一个字段的blank=True，表单的验证将允许该字段是空值。如果字段的blank=False，该字段就是必填的。

(2)default

字段的默认值。可以是一个值或者可调用对象。如果可调用 ，每有新对象被创建它都会被调用。

(3)primary_key

如果为True，那么这个字段就是模型的主键。如果你没有指定任何一个字段的primary_key=True，
Django 就会自动添加一个IntegerField字段做为主键，所以除非你想覆盖默认的主键行为，
否则没必要设置任何一个字段的primary_key=True。

(4)unique

如果该值设置为 True, 这个数据字段的值在整张表中必须是唯一的

(5)choices
由二元组组成的一个可迭代对象（例如，列表或元组），用来给字段提供选择项。 如果设置了choices ，默认的表单将是一个选择框而不是标准的文本框，而且这个选择框的选项就是choices 中的选项。

这是一个关于 choices 列表的例子：

YEAR_IN_SCHOOL_CHOICES = (
    ('FR', 'Freshman'),
    ('SO', 'Sophomore'),
    ('JR', 'Junior'),
    ('SR', 'Senior'),
    ('GR', 'Graduate'),
)
每个元组中的第一个元素，是存储在数据库中的值；第二个元素是在管理界面或 ModelChoiceField 中用作显示的内容。 在一个给定的 model 类的实例中，想得到某个 choices 字段的显示值，就调用 get_FOO_display 方法(这里的 FOO 就是 choices 字段的名称 )。例如：

from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)


>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'

(6)verbose_name
Admin中显示的字段名称

(7)help_text
Admin中该字段的提示信息

(8)on_delete=models.CASCADE
在foreignkey和onetoone的字段必写：on_delete=models.CASCADE:删除关联数据,与之关联也删除

(9)auto_now_add=True
uto_now_add=True不用赋值，自动拿到当前时间进行赋值

(10)related_name
在models.py使用Foreign定义外键的时候也可以传入一个参数related_name，用于反向生成，效果和orm语句 `表名小写_set` 作用一样，如果觉得orm反向跨表麻烦，可以添加related_name，直接使用 `.related_name名称` 进行反向跨表

(11)error_messages
自定义错误信息（字典类型），从而定制想要显示的错误信息；字典健：null, blank, invalid,invalid_choice, unique, and unique_for_date
如：{'null': "不能为空.", 'invalid': '格式错误'}
```  
### 模型的元数据Meta  
模型的元数据，指的是“除了字段外的所有内容”，例如排序方式、数据库表名、人类可读的单数或者复数名等等。所有的这些都是非必须的，甚至元数据本身对模型也是非必须的。但是，我要说但是，有些元数据选项能给予你极大的帮助，在实际使用中具有重要的作用，是实际应用的‘必须’。  
```
from django.db import models

class Ox(models.Model):
    horn_length = models.IntegerField()

    class Meta:         # 注意，是模型的子类，要缩进！
        ordering = ["horn_length"]
        verbose_name_plural = "oxen"
```  
上面的例子中，我们为模型Ox增加了两个元数据‘ordering’和‘verbose_name_plural’，分别表示排序和复数名，下面我们会详细介绍有哪些可用的元数据选项。  
强调：每个模型都可以有自己的元数据类，每个元数据类也只对自己所在模型起作用。  
******
**重要的元数据**  
**abstract**  
如果`abstract=True`，那么模型会被认为是一个抽象模型。抽象模型本身不实际生成数据库表，而是作为其它模型的父类，被继承使用。具体内容可以参考Django模型的继承。  
***
**app_label**  
如果定义了模型的app没有在INSTALLED_APPS中注册，则必须通过此元选项声明它属于哪个app，例如：`app_label = 'myapp'`  
***
**db_table**  
指定在数据库中，当前模型生成的数据表的表名。比如：`db_table = 'my_freinds'`  
***
**get_latest_by**  
Django管理器给我们提供有`latest()`和`earliest()`方法，分别表示获取最近一个和最前一个数据对象。但是，如何来判断最近一个和最前面一个呢？也就是根据什么来排序呢？  

`get_latest_by`元数据选项帮你解决这个问题，它可以指定一个类似 `DateField`、`DateTimeField`或者`IntegerField`这种可以排序的字段，作为`latest()`和`earliest()`方法的排序依据，从而得出最近一个或最前面一个对象。例如：`get_latest_by = "order_date"`  
***
**ordering**
最常用的元数据之一了！  
用于指定该模型生成的所有对象的排序方式，接收一个字段名组成的元组或列表。默认按升序排列，如果在字段名前加上字符“-”则表示按降序排列，如果使用字符问号“？”表示随机排列。请看下面的例子：  
```
ordering = ['pub_date']             # 表示按'pub_date'字段进行升序排列
ordering = ['-pub_date']            # 表示按'pub_date'字段进行降序排列
ordering = ['-pub_date', 'author']  # 表示先按'pub_date'字段进行降序排列，再按`author`字段进行升序排列。
```  
***
**index_together联合索引即将被indexes代替**  
接收一个应用在当前模型上的索引列表，如下例所示：  
```
from django.db import models

class Customer(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    class Meta:
        indexes = [
            models.Index(fields=['last_name', 'first_name']),  # 联合索引
            models.Index(fields=['first_name'], name='first_name_idx'),  #给联合索引命名
        ]
```  
***
**unique_together**  
举个例子，假设有一张用户表，保存有用户的姓名、出生日期、性别和籍贯等等信息。要求是所有的用户唯一不重复，可现在有好几个叫“张伟”的，如何区别它们呢？（不要和我说主键唯一，这里讨论的不是这个问题）  

我们可以设置不能有两个用户在同一个地方同一时刻出生并且都叫“张伟”，使用这种联合约束，保证数据库能不能重复添加用户（也不要和我谈小概率问题）。在Django的模型中，如何实现这种约束呢？  

使用`unique_together`，也就是联合唯一！  

比如：`unique_together = (('name', 'birth_day', 'address'),)`  
这样，哪怕有两个在同一天出生的张伟，但他们的籍贯不同，也就是两个不同的用户。一旦三者都相同，则会被Django拒绝创建。这一元数据经常被用在admin后台，并且强制应用于数据库层面。  

`unique_together`接收一个二维的元组((xx,xx,xx,...),(),(),()...)，每一个元素都是一个元组，表示一组联合唯一约束，可以同时设置多组约束。为了方便，对于只有一组约束的情况下，可以简单地使用一维元素，例如：`unique_together = ('name', 'birth_day', 'address')`
联合唯一无法作用于普通的多对多字段。  
***
**verbose_name**  
最常用的元数据之一！用于设置模型对象的直观、人类可读的名称。可以用中文。例如：  
```
verbose_name = "userinfo"
verbose_name = "用户表"
```  
如果你不指定它，那么Django会使用小写的模型名作为默认值。  
***
**verbose_name_plural**  
英语有单数和复数形式。这个就是模型对象的复数名，比如“apples”。因为我们中文通常不区分单复数，所以保持和`verbose_name`一致也可以。  
```
verbose_name_plural = "userinfos"
verbose_name_plural = "用户表"
```  
如果不指定该选项，那么默认的复数名字是`verbose_name`加上‘s’   
### Model扩展了解知识  
```
1.触发Model中的验证和错误提示有两种方式：
        a. Django Admin中的错误信息会优先根据Admiin内部的ModelForm错误信息提示，如果都成     功，才来检查Model的字段并显示指定错误信息
        b. 调用Model对象的 clean_fields 方法，如：
            # models.py
            class UserInfo(models.Model):
                nid = models.AutoField(primary_key=True)
                username = models.CharField(max_length=32)

                email = models.EmailField(error_messages={'invalid': '格式错了.'})

            # views.py
            def index(request):
                obj = models.UserInfo(username='11234', email='uu')
                try:
                    print(obj.clean_fields())
                except Exception as e:
                    print(e)
                return HttpResponse('ok')

           # Model的clean方法是一个钩子，可用于定制操作，如：上述的异常处理。

    2.Admin中修改错误提示
        # admin.py
        from django.contrib import admin
        from model_club import models
        from django import forms


        class UserInfoForm(forms.ModelForm):
            username = forms.CharField(error_messages={'required': '用户名不能为空.'})
            email = forms.EmailField(error_messages={'invalid': '邮箱格式错误.'})
            age = forms.IntegerField(initial=1, error_messages={'required': '请输入数值.', 'invalid': '年龄必须为数值.'})

            class Meta:
                model = models.UserInfo
                # fields = ('username',)
                fields = "__all__"

        class UserInfoAdmin(admin.ModelAdmin):
            form = UserInfoForm


        admin.site.register(models.UserInfo, UserInfoAdmin) # 注册admin
```  




