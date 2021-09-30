Django Note   
======================

配置已有数据库
----------------------

.. code-block:: python

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

然后在项目中执行命令 **python manage.py inspectdb**

错误**ORA-00904: "IDENTITY_COLUMN": invalid identifier**
----------------------

在执行python manage.py inspectdb命令从已有数据库表格生成models.py的时候会出现如下错误:

::

    [root@localhost mysite]# python manage.py inspectdb
    # This is an auto-generated Django model module.
    # You'll have to do the following manually to clean this up:
    #   * Rearrange models' order
    #   * Make sure each model has one field with primary_key=True
    #   * Make sure each ForeignKey has `on_delete` set to the desired behavior.
    #   * Remove `managed = False` lines if you wish to allow Django to create, modify, and delete the table
    # Feel free to rename the models, but don't rename db_table values or field names.
    from django.db import models
    # Unable to inspect table 'autodt_login'
    # The error was: ORA-00904: "IDENTITY_COLUMN": invalid identifier
    # Unable to inspect table 'autodt_spec_tco'
    # The error was: ORA-00904: "IDENTITY_COLUMN": invalid identifier
    [root@localhost mysite]#


目前没有找到更好的解决办法, 在网上看到可以通过降低django的版本来规避这个问题, 经测试1.11.22这个版本可以

::

    pip uninstall django
    pip install Django==1.11.22
    python manage.py inspectdb > fqc/models.py
    pip uninstall django
    pip install django==2.2

POST请求提交表单时出现以下错误: "**Forbidden (CSRF token missing or incorrect.): /fqc/inserttco**"
----------------------

::

    [11/Aug/2020 07:20:57] "GET /fqc/inserttco HTTP/1.1" 200 4057
    [11/Aug/2020 07:20:58] "GET /static/css/inserttco.css HTTP/1.1" 304 0
    Forbidden (CSRF token missing or incorrect.): /fqc/inserttco
    [11/Aug/2020 07:21:13] "POST /fqc/inserttco HTTP/1.1" 403 2513

只需要在views.py中增加以下内容即可

from django.views.decorators.csrf import csrf_exempt

然后在post处理函数前面增加"@csrf_exempt"

.. code-block:: python

    from django.shortcuts import render, render_to_response
    from django.views.decorators.csrf import csrf_exempt

    # Create your views here.
    from django.http import HttpResponse, HttpResponseRedirect
    from fqc.models import AutodtSpecTco, AutodtLogin
    import json

    ...

    @csrf_exempt
    def inserttco(request):
        if request.method == 'GET':
            return render_to_response("inserttco.html", {})
        else:
            print(request.method)
            tcocode = request.POST.get('tcocode', '')
            return HttpResponse(json.dumps({'code': 201, 'message': 'success', 'data': None}, ensure_ascii=False))

参考:
https://docs.djangoproject.com/zh-hans/2.2/
https://zhuanlan.zhihu.com/p/157776581
https://www.cnblogs.com/alex3174/p/11608374.html
