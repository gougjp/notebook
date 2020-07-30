# Django Note

## 安装

* 安装Python

* 安装django

```Shell
pip install Django
```

## 创建项目

```Shell
django-admin startproject mysite
```

这行代码将会在当前目录下创建一个mysite目录, mysite目录下文件结构如下:

```Shell
mysite/
|-- manage.py
`-- mysite
    |   `-- __init__.py
    |   `-- asgi.py
    |   `-- settings.py
    |   `-- urls.py
        `-- wsgi.py

1 directory, 6 files
```

这些目录和文件的用处是：

- 最外层的 mysite/ 根目录只是你项目的容器, Django 不关心它的名字, 你可以将它重命名为任何你喜欢的名字.
- manage.py: 一个让你用各种方式管理 Django 项目的命令行工具: https://docs.djangoproject.com/zh-hans/2.2/ref/django-admin/
- 里面一层的 mysite/ 目录包含你的项目, 它是一个纯 Python 包. 它的名字就是当你引用它内部任何东西时需要用到的 Python 包名. (比如 mysite.urls).
- mysite/__init__.py：一个空文件, 告诉 Python 这个目录应该被认为是一个 Python 包.
- mysite/settings.py：Django 项目的配置文件: https://docs.djangoproject.com/zh-hans/2.2/topics/settings/
- mysite/urls.py：Django 项目的 URL 声明, 就像你网站的"目录": https://docs.djangoproject.com/zh-hans/2.2/topics/http/urls/
- mysite/wsgi.py：作为你的项目的运行在 WSGI 兼容的Web服务器上的入口: https://docs.djangoproject.com/zh-hans/2.2/howto/deployment/wsgi/

进入到mysite目录下, 执行如下命令来启动服务器:

```Shell
$ python manage.py runserver
```

输出如下:

```Shell
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
December 04, 2019 - 11:22:44
Django version 3.0, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

然后在浏览器中用http://127.0.0.1:8000/即可访问服务器

默认情况下，runserver 命令会将服务器设置为监听本机内部IP的8000端口
如果你想更换服务器的监听端口,请使用命令行参数.举个例子,下面的命令会使服务器监听8080端口：

```Shell
$ python manage.py runserver 8080
```
    
如果你想要修改服务器监听的IP,在端口之前输入新的. 比如,为了监听所有服务器的公开IP(这你运行Vagrant或想要向网络上的其它电脑展示你的成果时很有用),使用:

```Shell
$ python manage.py runserver 0:8000
```
    
\0 是0.0.0.0的简写

## 创建一个新的项目

在manage.py同一级目录, 运行如下命令来创建一个应用:

```Shell
$ python manage.py startapp polls
```

这将会创建一个 polls 目录, 它的目录结构大致如下：

```Shell
polls/                                             
|-- __init__.py                                    
|-- admin.py                                       
|-- apps.py                                        
|-- migrations                                     
|   `-- __init__.py
|-- models.py                                      
|-- tests.py                                       
`-- views.py                                       
                                                   
1 directory, 7 files
```      

## 配置已有数据库

```Python
DATABASES = {
    'default': {
        # 'ENGINE': 'django.db.backends.sqlite3',
        'ENGINE': 'django.db.backends.oracle',
        # 'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        'NAME': '<server_ip>:1521/Product_Test_DB',
        'USER':'<username>',
        'PASSWORD':'<password>',
    }
}
```

然后在项目中执行命令 **python manage.py inspectdb**

## 编写第一个试图

打开 polls/views.py，把下面这些 Python 代码输入进去:

```Python
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

然后将一个 URL 映射到这个view——这就是我们需要 URLconf 的原因了

为了创建 URLconf，请在 polls 目录里新建一个 urls.py 文件。你的应用目录现在看起来应该是这样：

```Shell
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py
```

在 polls/urls.py 中，输入如下代码：

```Python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

下一步是要在根 URLconf 文件中指定我们创建的 polls.urls 模块. 在 mysite/urls.py 文件的 urlpatterns 列表里插入一个 include(),如下:

```Python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

函数 include() 允许引用其它 URLconfs. 每当 Django 遇到 include() 时, 它会截断与此项匹配的 URL 的部分，并将剩余的字符串发送到 URLconf 以供进一步处理.

我们设计 include() 的理念是使其可以即插即用. 因为投票应用有它自己的 URLconf( polls/urls.py ), 他们能够被放在 "/polls/", "/fun_polls/", "/content/polls/", 或者其他任何路径下, 这个应用都能够正常工作.

然后运行服务器即可.

函数 path() 具有四个参数，两个必须参数：route 和 view，两个可选参数：kwargs 和 name

**route**

route 是一个匹配 URL 的准则(类似正则表达式). 当 Django 响应一个请求时, 它会从 urlpatterns 的第一项开始, 按顺序依次匹配列表中的项, 直到找到匹配的项.

这些准则不会匹配 GET 和 POST 参数或域名. 例如, URLconf 在处理请求 https://www.example.com/myapp/ 时, 它会尝试匹配 myapp/. 处理请求 https://www.example.com/myapp/?page=3 时, 也只会尝试匹配 myapp/

**view**

当 Django 找到了一个匹配的准则, 就会调用这个特定的视图函数, 并传入一个 HttpRequest 对象作为第一个参数, 被"捕获"的参数以关键字参数的形式传入

**kwargs**

任意个关键字参数可以作为一个字典传递给目标视图函数

**name**

为你的 URL 取名能使你在 Django 的任意地方唯一地引用它, 尤其是在模板中. 这个有用的特性允许你只改一个文件就能全局地修改某个 URL 模式










参考:https://docs.djangoproject.com/zh-hans/2.2/

https://zhuanlan.zhihu.com/p/157776581

