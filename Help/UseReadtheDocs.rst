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








































摘自: https://www.xncoding.com/2017/01/22/fullstack/readthedoc.html
