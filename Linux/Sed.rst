Sed 命令
======================

插入行
--------------

.. code::

    # 在文件test.conf的第5行之前插入this is a test line
    sed -i '5i\this is a test line' test.conf

    #在文件test.conf的第5行之后插入this is a test line
    sed -i '5a\this is a test line' test.conf
    
    #在指定行后面插入一行,这里是在"#   StrictHostKeyChecking ask"这一行后面加上"StrictHostKeyChecking no";这里的/a是指在匹配到的行后面加入
    sed -i '/#   StrictHostKeyChecking ask/aStrictHostKeyChecking no' /etc/ssh/ssh_config
    
    #在指定行前面插入一行,这里是在"#   StrictHostKeyChecking ask"这一行前面加上"StrictHostKeyChecking no";这里的/i是指在匹配到的行前面加入
    sed -i '/#   StrictHostKeyChecking ask/iStrictHostKeyChecking no' /etc/ssh/ssh_config
    
    #在文件最后插入一行,$匹配文件最后,a\表示之后插入
    sed -i '$a\somestring' file.txt
    
替换
---------

.. code::

    #将文件file中的text替换为replace, 替换每一行的第一个text, 如果某一行有多个text, 后面的几个都不会被替换
    sed -i 's/text/replace/' file
    
    #将文件file中的text替换为replace, 替换每一行的所有text
    sed -i 's/text/replace/g' file

    #只替换第20行
    sed -i '20s/text/replace/g' file

删除行
-----------

.. code::

    #删除匹配到的行, 匹配到多行的话就删除匹配到的所有行
    sed -i '/userChannelParametersIndex/d' TswUserSetupReq.txt

    #删除第n行
    sed -i nd filename

    #从第m行开始,包括第m行,每隔n-1行删除
    sed -i m~nd filename

    #删除第m到n行, 包括m和n行
    sed -i m,nd filename
    sed -i 'm,n'd filename
    sed -i 'm,nd' filename

    #删除最后一行
    sed -i '$'d filename

    #删除从匹配行到文件结尾,删除从匹配到的somestring行及以后的所有行
    sed -i '/somestring/,$d' filename

    #删除匹配行及之后的n行
    sed '/somestring/,+nd' filename
    sed '/somestring/,+n'd filename

    #删除空行
    sed '/^$/d' filename

在行首行位加HTML标签
--------------------------

.. code::

    # 这里分号用于分割命令; q退出命令, 这样程序只处理第一行文本
    sed -n 's/^/<h1>/;s/$/<\/h1>p;q' rime.txt
    


