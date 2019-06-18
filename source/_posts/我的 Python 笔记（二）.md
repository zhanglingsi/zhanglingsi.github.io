---
title: 我的 Python 笔记（二）
categories: python
copyright: true
---

# 我的 Python 笔记（二）

## 搭建 Django 开发环境 
> 学习了Python的语法和基础知识，对于老程序员来说，可直接上手学习web全能型Python框架Django

### 基础环境

- system : windows 10 
- python : anaconda3 并创建虚拟环境 conda create -n leanrn python=3.6
- Django : V2.1  Django 2.1支持Python 3.5及以后版本。
- 查看Django版本信息 : python -m django --version

> **打开Anaconda Prompt 终端**

1. 创建虚拟环境
> $> conda create -n learn python=3.6 
注：我的windows10 里面 创建的虚拟环境对应的目录是在~\.conda\envs\learn\python.exe

2. 查看当前环境
> $> conda env list

3. 激活创建的新环境 learn
> $> activate learn

4. 查看当前环境下安装的包
> $> conda list

5. 安装Django包
> $> conda install django=2.=1   或者  pip install django==2.1

6. 安装pymysql包
> $> conda install pymysql

### 配置开发环境 pycharm 
    
1. 使用Django创建项目骨架, learn_site是项目名称 
> $> django-admin startproject learn_site  
注: 
    1. 也可以使用pycharm新建项目，选择New Project，在左面选择Django，右面Location路径填写你的项目目录，我的是D:\py_code\learn_site
    2. Project Interpreter > Existing interpreter > interpreter 里面选择你新建的环境的python.exe
    3. 在More Settings > Application name: 给模块起个名字  blog
    4. django中的基本概念，一个项目中包含多个Application应用，这里建的blog是建立项目时一起新建一个应用。当然这里可以不新建应用，不是必须的 
    5. 记得选中Enable Django admin,这个是Django自带的非常强大的web后台管理平台。
<!-- more -->
2. 启动项目，验证 进入项目的根目录 
> $> python manage.py runserver 
注：
    1. 点击其中的链接，或使用浏览器访问http://127.0.0.1:8000/即可看到Django的欢迎页面
    2. 这是Django开发服务器展现的网页，官方不建议在生产环境中使用开发服务器，所以在生产环境中仍需要使用Apache、Nginx等Web服务器。生产环境必须在项目根目录的settings.py关闭DEBUG = False
    3. Django开发服务器会自动加载更新后的Python代码，无需重启。但增加文件后需要重启Django开发服务器。
    4. 指定IP和端口 python manage.py runserver 0:8000

3. 创建应用  在项目根目录（manage.py）
> $> python manage.py startapp blog

4. 修改learn_site/settings.py 文件
```
INSTALLED_APPS = [
    .....
    'blog', # 添加应用 或者使用 'blog.apps.BlogConfig',
]
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'blog',  # mysql中创建数据库 create database blog
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': '127.0.0.1',
        'PORT': '3306'
    }
}
LANGUAGE_CODE = 'zh_Hans'  # 修改语言

TIME_ZONE = 'Asia/Shanghai'  #修改时区
```

5. 打开blog/views.py，输入以下代码：
```
from django.http import HttpResponse
 
 
def index(request):
    return HttpResponse("Hello, world.")
```

6. 修改learn_site/urls.py文件
```
app_name = 'blog'  # 命名空间
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'blog/', include(('blog.urls', 'blog'), namespace='blog')),
]
```

7. 修改 learn_site/ __init__.py文件
```
import pymysql
pymysql.install_as_MySQLdb()
```

8. 修改blog\urls.py文件
```
from django.urls import path
 
from . import views
 
urlpatterns = [
    path('', views.index, name='index'),
]

```

9. 启动django  
> $> python manage.py runserver 

10. 访问 http://127.0.0.1:8000/blog 即可显示Hello world.


### Django 命令操作

> 1. 创建管理员账号
    $> **python manage.py createsuperuser --username=admin --email=admin@qq.com**
   
> 2. 模型生成sql脚本 appname 是应用名称  如果不写表示生成所有应用的sql
    $> **python manage.py makemigrations appname** 
    
> 3. 模型orm到mysql
    $> **python manage.py migrate**   
    
> 4. 清空数据库
    $> **python manage.py flush** 

> 5. 项目终端
    $> **python manage.py shell** 

> 6. 修改管理员密码
    $> **python manage.py changepassword username**     

