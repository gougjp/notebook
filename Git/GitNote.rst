Git Note
===========

Git查看某个提交的文件列表
----------------------------

.. code::

    git show --pretty=format:""  --name-only <commit_id1>..<commit_id2>
    
    如:
    git show --pretty=format:""  --name-only aa282dcb8dbd03d83b3d196cad6cdb43e7e15dbf..c221841eddb41efbab0b4ab342a1c21663dde508
    04 src/02 firmware/Source/Committee/PasswordDM.c
    04 src/02 firmware/Source/Committee/PasswordDM.h
    04 src/02 firmware/Source/Common/control.c
    04 src/02 firmware/Source/Mcu/i2cs.c

    04 src/02 firmware/Source/App/CmdServ.c
    04 src/02 firmware/Source/Common/History.txt
    04 src/02 firmware/Source/Common/common.c
    04 src/02 firmware/Source/history.xlsx

    04 src/02 firmware/Source/Committee/PasswordDM.c
    04 src/02 firmware/Source/Committee/PasswordDM.h
    04 src/02 firmware/Source/Common/History.txt
    04 src/02 firmware/Source/Common/common.h
    04 src/02 firmware/Source/Common/control.c
    04 src/02 firmware/Source/Mcu/i2cs.c
    04 src/02 firmware/Source/history.xlsx

Git hooks
----------------------

* 如何使用git hooks:

.git\hooks目录下有一些现成的hooks模板, 如果要打开某个hooks, 需要将文件名后缀.sample删除, 然后编辑该文件即可

post-merge是一种客户端hooks, git merge和git pull后就会执行该hooks, 创建文件.git\hooks\post-merge

.. code::

    #! /bin/bash
    
    echo "post merge hooks"

然后执行git pull命令, 命令完成后会输出post merge hooks, 这里如果是Windows系统, 也一样, 只是下面的命令是Dos命令就好了, 第一行"#! /bin/bash"都必须这样写

* 用Python来实现hooks; 对于批处理脚本也可以用这种方式实现

post-merge文件类容:

.. code::

    #! /bin/bash
    
    python .git/hooks/post-merge.py
    
其中.git/hooks/post-merge.py文件是用Python实现的hooks, 名字可以随意

* post-checkout在git checkout和git clone后调用

* git hooks只保存在本地, 无法提交到服务器; 可以保存在其他目录, 克隆代码库过后自动拷贝到.git/hooks目录下; 或者在代码库跟目录下执行
git config core.hooksPath .githooks命令, 其中.githooks是自定义的一个目录, 里面放的是所有的hooks文件









参考:
-------------
https://blog.csdn.net/richard_jason/article/details/53188200
https://blog.viktoradam.net/2018/07/26/githooks-auto-install-hooks/
https://www.viget.com/articles/two-ways-to-share-git-hooks-with-your-team/













