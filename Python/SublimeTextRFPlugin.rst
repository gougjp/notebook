Robot Framework Assistant插件安装配置
=========================================

* 手动下载ZIP包：

https://github.com/andriyko/sublime-robot-framework-assistant.git

* 打开Sublime插件目录：

Preferences -> Browse Packages ...

* 在打开的路径下创建目录 Robot Framework Assistant, 该目录跟User目录为同一级

* 将前面下载的ZIP包拷贝到刚创建的Robot Framework Assistant目录中

* 重新打开Sublime, 可以在Preferences -> Packages Settings下看到Robot Framework Assistant插件

* 点击Preferences -> Packages Settings -> Robot Framework Assistant -> Settings - Users; 打开配置文件

* 将如下配置拷贝到打开的配置文件, 并修改好python的路径和robotframework库的路径

.. code::

    /*
        Robot Framework Assistant User settings for panfeng
    */
    {
        "path_to_python": "C:\\Python36\\python.exe",

        //"robot_framework_workspace": "D:\\OneDrive\\TesterScript\\RF",

        "robot_framework_module_search_path":
            [
                "C:\\Python36\\Lib\\site-packages",
            ],
        "robot_framework_keyword_argument_format": true,
    }










