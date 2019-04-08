---
layout:     post   				    # 使用的布局（不需要改）
title:      python自动化测试-selenium 				# 标题 
subtitle:   自动化测试 #副标题
date:       2019-4-8 				# 时间
author:     QQL 						# 作者
header-img: img/background-2426328_1920.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - python
---




## 自动化测试

### Python自动化测试学习路线

**第一步**：先学python        -------  python不过关，别谈自动化。  
**第二步**：Selenium框架  
**第三步**：unittest框架  
**第四步**：项目  
**第五步**：Robot Framwork框架  
如果只学习Robot Framwork，通过这个来做自动化，别以为不要学习Selenium跟python了， 因为Robot Framework中的关键字可能不够用，不能满足你们的需求，那么我们需要自定义关键字，这个时候就必须自己得通过python+selenium来编写了  


> 学习路径参考https://www.jianshu.com/p/5b25b37f1556



### 软件开发流程

**需求分析----->架构模块设计----->编码----->单元测试----->集成测试----->系统测试----->验收**  

![软件开发流程.png](https://i.loli.net/2019/04/05/5ca76fc113cc9.png)   

### 测试分类

![测试分类.png](https://i.loli.net/2019/04/05/5ca76ff9f38ef.png)   

### 自动化测试优点

![自动化测试优点.png](https://i.loli.net/2019/04/05/5ca772b9e5303.png)

### 自动化测试适宜场景

1. 任务测试明确，不会频繁改动
2. 软件需求变更少
3. 项目周期长，测试脚本可以复用


### 常用测试工具

![常用测试工具.png](https://i.loli.net/2019/04/05/5ca7739dc79f8.png)

### 使用selenium
使用selenium中的webdriver模块对浏览器进行操作
`from selenium import webdriver`  
1. `b = webdriver.Firefox()   c = webdriver.Chrome()`  **打开浏览器**
2. `b.get('http://qqlblog.cc')`  **打开一个网页**
3. `b.title获取网站标题   b.current_url获取当前url  b.maximize_window()最大化窗口`  **判断访问是否有效**
4. **元素定位**
![元素定位.png](https://i.loli.net/2019/04/06/5ca8648a56688.png)  

5. **元素操作的方式**  
![元素操作.png](https://i.loli.net/2019/04/06/5ca865133bab3.png)  

6. `b.back`  **回到上一页**  
7. **选中元素后，元素类型为webelement object。 通过. 获取属性参数**  
```
ele = b.find_element_by_class_name('s_ipt')
ele.size  # 获取选中元素的width和height
ele.id  # 获取元素id
```  

8. **针对a标签可以使用**`ele = b.find_element_by_link_text('a标签字符串')`
9. **模糊匹配字符串标签**`ele = b.find_element_by_partial_link_text('a标签字符串')`
10. **通过css查找元素**
```
通过标签和属性选择器查找元素
ele = b.find_element_by_css_selector('input[id=\'search\']')  # \为转译字符
ele = b.find_eleemnt_by_css_selector('input[type="text"]')
ele = b.find_element_by_css_selecotr('img[alt="图片描述"]')
```

### xpath获取元素
![xpath定位.png](https://i.loli.net/2019/04/06/5ca86f04572a5.png)  

```
ele = b.find_element_by_xpath('//*')  全局获取所有元素
ele.tag_name  获取当前元素标签名
ele = b.find_eleemnt_by_xpath('//*[count(input)=2]')  全局获取包含2个input标签的元素
ele = b.find_eleemnt_by_xpath('//ul/li[1]')  获取ul标签下第一个li标签(下标从1开始)
```  
![xpath2.png](https://i.loli.net/2019/04/06/5ca86fe6d3d8f.png)  

```
1.查找最后匹配元素一个：[last()]
ele = b.find_element_by_xpath('//*[contains(local-name(),"i")][last()]')
2.对元素位置进行切换：[last()-1]   last返回最大索引可进行运算
ele = b.find_element_by_xpath('//*[contains(local-name(),"i")][last()-1]')

```




