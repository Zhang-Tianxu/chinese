---
title: Django的基本用法及数据库连接
tags:
  - python
  - Django
  - web
  - 数据库
categories:
  - web
  - Django
date: 2018-11-21 20:32:22
---


# 创建Django项目
&emsp;安装 Django 之后，我们就有了管理工具 django-admin.py。我们可以使用 django-admin.py 来创建一个项目：
```shell
django-admin startproject HelloWorld
```
&emsp;对于旧版本的Django要使用：
```shell
django-admin.py startproject HelloWorld
```
&emsp;进入项目目录，执行：
```shell
python manage.py runserver 0.0.0.0:8000
```
来启动项目，然后在浏览器中访问*127.0.0.1:8000*可知项目是否创建成功。
<!--more-->
# URL配置
&emsp;创建项目后我们必然要添加自己的页面，同时我们也需要配置该页面的URL。在讲Django模板之前我们没办法直接返回*html*文件，只能直接返回内容，当然你也可以把这些内容组织为*html*格式。
1. **添加页面处理**。在*…/HelloWorld/HelloWorld/*目录下新建*newpage.py*文件，文件代码如下：
```python
from django.http import HttpResponse

#这是视图函数，配置url时要用到
def HelloWorld(request):
    return HttpResponse("<h1>Hello World</h1>")
```
2. **配置URL**。编辑*…/HelloWorld/HelloWorld/*目录下的*urls.py*，添加两行代码
```python
#******原有import********
from . import newpage

urlpatterns = [
    #*****原有url****
    url(r'^HelloWorld$',newpage.HelloWorld),
]
```
访问*127.0.0.1:8000/HelloWorld*即可得到新建的页面。
>url() 函数:
Django url() 可以接收四个参数，分别是两个必选参数：regex、view 和两个可选参数：kwargs、name，接下来详细介绍这四个参数。
*regex: 正则表达式，与之匹配的 URL 会执行对应的第二个参数 view。
view: 用于执行与正则表达式匹配的 URL 请求。
kwargs: 视图使用的字典类型的参数。
name: 用来反向获取 URL。*
# Django模板
&emsp;若想返回*html*文件，需要先指定*…/HelloWorld/HelloWorld/setting.py*中的*TEMPLATES*，将其*DIRS*指定为某个文件夹（一般为templates），然后讲html文件放入该文件夹调用。
&emsp;我们可以在讲*TEMPLATES*的*DIRS*指定为`'DIRS': [BASE_DIR+"/templates",],`，然后在*…/HelloWorld/templates/*目录下新建*HelloWorld.html*：
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>HelloWorld</title>
</head>
<body>
    <h1>Hello World From html</h1>
</body>
</html>
```
&emsp;最后将*newpage.py*的内容改为：
```python
# -*- coding:utf-8 -*-

# from django.http import render
from django.shortcuts import render_to_response
def HelloWorld(request):
	return render_to_response('HelloWorld.html')
```
&emsp;因为url已经配置过了，所以直接访问*127.0.0.1:8000/HelloWorld*即可得到新的页面内容。  
***
&emsp;如果只是这样使用Django模板的话，就真的是浪费了。**Django模板的方便之处在于可以将文档的表现形式和内容分开**。我们在*…/HelloWorld/templates/*目录下建立*pattern.html*用于存储文档的表现形式：
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>pattern name</title>
</head>
<body>
    <h1>{{ title }}</h1>
    <p>Django 模板</p>
    {% block mainbody %}
       <p>default contents</p>
    {% endblock %}
</body>
</html>
```
&emsp;然后在用以目录下建立*content.html*用于存储要展示的内容：
```html
{%extends "pattern.html" %}
 
{% block mainbody %}
<p>继承了pattern.html的排版，展示我自己的内容</p>
{% endblock %}
```
&emsp;接下来配置URL，在*…/HelloWorld/HelloWorld/*目录下新建*template.py*文件，文件代码如下：
```python
# -*- coding:utf-8 -*-

# from django.http import render
from django.shortcuts import render

def template(request):
    title = {}
    title['title'] = 'I am title'
    return render(request, 'content.html', title)
```
在*urls.py*中添加`url(r'^template$',template.template),`就能访问*127.0.0.1:8000/template*看到效果了。
# Django之App
&emsp;Django是一个比较重型的Web框架，一个比较大型的网站下可能有许多个分支，为了实现隔离可以创建多个用`django-admin startapp appName`命令来创建多个APP。有了APP我们就可以使用**Django模型**_（注意不是模板）_，进而使用数据库。

# Django的数据库连接
&emsp;Django 对各种数据库提供了很好的支持，包括：PostgreSQL、MySQL、SQLite、Oracle。Django 为这些数据库提供了统一的调用API。 我们可以根据自己业务需求选择不同的数据库。
&emsp;Django的默认数据库是*SQLite3*，想要转换为其他数据库需要调整一些参数。我们以Mysql为例做讲解数据库的连接。
&emsp;Django规定，如果要使用*模型*进而对数据库进行操作，就必须创建一个app。连接并操作Mysql数据库的步骤如下：
1. 在Mysql中建立一个database
2. 创建一个新APP
3. 在*setting.py*的*INSTALLED_APPS*中加入新的APP
4. 填写*setting.py*的*DATABASES*：
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',  # 或者使用 mysql.connector.django
        'NAME': 'test',#数据库名
        'USER': 'test',#用户名
        'PASSWORD': 'test123',#用户密码
        'HOST':'localhost',#数据库所在主机地址
        'PORT':'3306',#mysql服务端口
    }
}
```
5. 填写APP目录下*\_\_int\_\_.py*文件
```python
import pymysql
pymysql.install_as_MySQLdb()
```
&emsp;这一步非常重要，我因为不知道要修改*\_\_int\_\_.py*文件，导致Django不能运行，最后才知道是数据库连接失败的原因。
具体的连接细节和操作方法网上有很多教程，可以参见[菜鸟教程](http://www.runoob.com/django/django-model.html)
