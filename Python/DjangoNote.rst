Django Note
===========================

安装
--------------

* 安装Python

* 安装django

.. code::

    pip install Django

创建项目
-----------------

.. code::

    django-admin startproject mysite

这行代码将会在当前目录下创建一个mysite目录, mysite目录下文件结构如下:

.. code::

    mysite/
    |-- manage.py
    `-- mysite
        |   `-- __init__.py
        |   `-- asgi.py
        |   `-- settings.py
        |   `-- urls.py
            `-- wsgi.py

    1 directory, 6 files

这些目录和文件的用处是：

* 最外层的 mysite/ 根目录只是你项目的容器, Django 不关心它的名字, 你可以将它重命名为任何你喜欢的名字.
* manage.py: 一个让你用各种方式管理 Django 项目的命令行工具: https://docs.djangoproject.com/zh-hans/2.2/ref/django-admin/
* 里面一层的 mysite/ 目录包含你的项目, 它是一个纯 Python 包. 它的名字就是当你引用它内部任何东西时需要用到的 Python 包名. (比如 mysite.urls).
* mysite/__init__.py：一个空文件, 告诉 Python 这个目录应该被认为是一个 Python 包.
* mysite/settings.py：Django 项目的配置文件: https://docs.djangoproject.com/zh-hans/2.2/topics/settings/
* mysite/urls.py：Django 项目的 URL 声明, 就像你网站的"目录": https://docs.djangoproject.com/zh-hans/2.2/topics/http/urls/
* mysite/wsgi.py：作为你的项目的运行在 WSGI 兼容的Web服务器上的入口: https://docs.djangoproject.com/zh-hans/2.2/howto/deployment/wsgi/

进入到mysite目录下, 执行如下命令来启动服务器:

.. code::

    $ python manage.py runserver

输出如下:

.. code::

    Watching for file changes with StatReloader
    Performing system checks...

    System check identified no issues (0 silenced).

    You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
    Run 'python manage.py migrate' to apply them.
    December 04, 2019 - 11:22:44
    Django version 3.0, using settings 'mysite.settings'
    Starting development server at http://127.0.0.1:8000/
    Quit the server with CTRL-BREAK.

然后在浏览器中用http://127.0.0.1:8000/即可访问服务器

默认情况下，runserver 命令会将服务器设置为监听本机内部IP的8000端口
如果你想更换服务器的监听端口,请使用命令行参数.举个例子,下面的命令会使服务器监听8080端口：

.. code::

    $ python manage.py runserver 8080
    
如果你想要修改服务器监听的IP,在端口之前输入新的. 比如,为了监听所有服务器的公开IP(这你运行Vagrant或想要向网络上的其它电脑展示你的成果时很有用),使用:

.. code::

    $ python manage.py runserver 0:8000
    
0 是0.0.0.0的简写

创建一个新的项目
----------------------

在manage.py同一级目录, 运行如下命令来创建一个应用:

.. code::

    $ python manage.py startapp polls

这将会创建一个 polls 目录, 它的目录结构大致如下：

.. code::

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



















参考:https://docs.djangoproject.com/zh-hans/2.2/                         