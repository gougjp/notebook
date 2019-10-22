MySql新建用户
===========================

1. 登录数据库: 

.. code::

    mysql -u root -p

2. 查看现有用户：

.. code::

    select host,user,authentication_string from mysql.user;
    
.. image:: images/CheckUser.jpeg

3. 创建用户:

.. code::

    create user 'sonar'@'localhost' identified by 'sonar';
    
.. image:: images/CreateUser.jpeg























FAQ
-----

* 安装完成后, 在命令行用命令mysql -u root -p登录数据库的时候出现以下错误:

.. image:: images/LoginError.jpeg

可以进入到C:\Program Files\MySQL\MySQL Server 8.0\bin目录, 然后在当前目录下打开cmd:

.. image:: images/ChangePassword.jpeg

然后就可以了

    


