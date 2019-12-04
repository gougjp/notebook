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








参考:https://docs.djangoproject.com/zh-hans/2.2/
