搭建ReadtheDocs托管笔记
===============================

Read the Docs是一个在线文档托管服务, 你可以从各种版本控制系统中导入文档, 如果你使用webhooks, 
那么每次提交代码后可以自动构建并上传至readthedocs网站, 非常方便.

一般来讲, 这个非常适合写软件文档以及编写一些教程, 电子书之类. 对于一些一两篇文章就能写清楚的
可以记笔记或写博客, 但是如果要写成一个系列的, 不如写成一本书的形式, 更美观, 也更系统.

Sphinx + GitHub + ReadtheDocs 作为文档写作工具, 用 Sphinx 生成文档, GitHub 托管文档, 再导入
到 ReadtheDocs.

Sphinx
---------

Sphinx是一个基于Python的文档生成项目, 最早只是用来生成 Python 官方文档, 随着工具的完善, 越来
越多的知名的项目也用他来生成文档, 甚至完全可以用他来写书采用了reStructuredText作为文档写作语
言, 不过也可以通过模块支持其他格式, 包括MarkDown格式.

* 安装Sphinx:

.. code::

    pip install sphinx sphinx-autobuild sphinx_rtd_theme
    
* 初始化:

.. code::

    # 创建文档根目录F:\cookbook, 并在命令行中进到该目录
    cd /d F:\cookbook
    # 然后执行以下命令初始化, 可以都选默认配置
    sphinx-quickstart
    
    #以下是执行结果
    F:\cookbook>sphinx-quickstart
    Welcome to the Sphinx 2.0.1 quickstart utility.

    Please enter values for the following settings (just press Enter to
    accept a default value, if one is given in brackets).

    Selected root path: .

    You have two options for placing the build directory for Sphinx output.
    Either, you use a directory "_build" within the root path, or you separate
    "source" and "build" directories within the root path.
    > Separate source and build directories (y/n) [n]: y

    The project name will occur in several places in the built documentation.
    > Project name: CookBook
    > Author name(s): GouJunping
    > Project release []: 0.1

    If the documents are to be written in a language other than English,
    you can select a language here by its language code. Sphinx will then
    translate text that it generates into that language.

    For a list of supported codes, see
    http://sphinx-doc.org/config.html#confval-language.
    > Project language [en]: zh_CN

    Creating file .\source\conf.py.
    Creating file .\source\index.rst.
    Creating file .\Makefile.
    Creating file .\make.bat.

    Finished: An initial directory structure has been created.

    You should now populate your master file .\source\index.rst and create other documentation
    source files. Use the Makefile to build the docs, like so:
       make builder
    where "builder" is one of the supported builders, e.g. html, latex or linkcheck.

* 然后运行 tree -C . 查看生成的sphinx结构:

.. code::

    .                      
    |-- Makefile           
    |-- build              
    |-- make.bat           
    `-- source             
        |-- _static        
        |-- _templates     
        |-- conf.py        
        `-- index.rst      
                           
    4 directories, 4 files 

* 添加一篇文章, 在source目录下新建hello.rst, 内容如下:

.. code::

    hello,world
    =============

* index.rst 修改如下:

.. code::

    .. toctree::
       :maxdepth: 2
       :caption: Contents:
       
       # 这里的hello是新增的,为hello.rst的文件名
       hello

* 更改主题 sphinx_rtd_theme:

更改source/conf.py:

.. code::

    # 用以下代码替换html_theme = 'alabaster'一行
    import sphinx_rtd_theme
    html_theme = "sphinx_rtd_theme"
    html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]

* 在根目录F:\cookbook执行make html命令, 输出信息如下:

.. code::

    F:\cookbook>make html
    Running Sphinx v2.0.1
    loading translations [zh_CN]... done
    making output directory... done
    building [mo]: targets for 0 po files that are out of date
    building [html]: targets for 2 source files that are out of date
    updating environment: 2 added, 0 changed, 0 removed
    reading sources... [100%] index
    looking for now-outdated files... none found
    pickling environment... done
    checking consistency... done
    preparing documents... done
    writing output... [100%] index
    generating indices... genindex
    writing additional pages... searchc:\python36\lib\site-packages\sphinx_rtd_theme\search.html:20: RemovedInSphinx30Warnin
    g: To modify script_files in the theme is deprecated. Please insert a <script> tag directly in your theme instead.
      {{ super() }}

    copying static files... done
    copying extra files... done
    dumping search index in Chinese (code: zh) ... done
    dumping object inventory... done
    build succeeded.

    The HTML pages are in build\html.

* 进入build/html目录后用浏览器打开index.html

.. image:: images/HelloWorld.jpg

* toctree 支持多级目录,比如要想将python.rst,java.rst笔记在不同的目录,toctree这样设置:

.. code::

    Contents:

    .. toctree::

       python/python
       swift/swift
       
    # 也可以使用通配符*来匹配所有的.rst文件
    Contents:

    .. toctree::

       python/*
       swift/*
    






摘自: https://www.xncoding.com/2017/01/22/fullstack/readthedoc.html
