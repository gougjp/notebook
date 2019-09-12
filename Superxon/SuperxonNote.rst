Superxon Note
==================

数据回传工具打包exe方法
----------------------------

1. 安装对应的包

.. code::

    # 这两个包为打包工具
    pip install pyinstaller
    pip install pywin32
    # 这个包为Python文件中引用的
    pip install cx-Oracle
    
2. 进入到对应的目录,执行如下命令

.. code::

    pyinstaller -F main.py

3. 如果出现以下错误，则可能是中文路径导致的

.. code::

    Traceback (most recent call last):
      File "c:\python27\lib\runpy.py", line 174, in _run_module_as_main
        "__main__", fname, loader, pkg_name)
      File "c:\python27\lib\runpy.py", line 72, in _run_code
        exec code in run_globals
      File "C:\Python27\Scripts\pyinstaller.exe\__main__.py", line 5, in <module>
      File "c:\python27\lib\site-packages\PyInstaller\__init__.py", line 72, in <module>
        DEFAULT_SPECPATH = compat.getcwd()
      File "c:\python27\lib\site-packages\PyInstaller\compat.py", line 613, in getcwd
        cwd = win32api.GetShortPathName(cwd)
    AttributeError: 'module' object has no attribute 'GetShortPathName'

Jenkins 服务器维护
----------------------------

1. 重启服务：

配置文件：
/etc/init.d/jenkins
/etc/default/jenkins

.. code::

    service jenkins status    #查看Jenkins服务的状态
    service jenkins restart   #重启Jenkins服务
    service jenkins stop      #停止Jenkins服务
    service jenkins start     #启动Jenkins服务
    
2. 版本更新：

将下载的新版jenkins.war放到/usr/share/jenkins/目录下，然后重启Jenkins服务
