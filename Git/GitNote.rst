Git Note
===========

Git查看某个提交的文件列表
----------------------------

* 使用git show命令

.. code::

    git show --pretty=format:"" --name-only <commit_id1>..<commit_id2>
    
    如:
    >> git show --pretty=format:""  --name-only aa282dcb8dbd03d83b3d196cad6cdb43e7e15dbf..c221841eddb41efbab0b4ab342a1c21663dde508
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

* 使用git log --name-status命令

.. code::

    git log --name-status <commit_id1>..<commit_id2>
    
    如:
    >> git log --name-status c81a665043ac406c4132d9844217f9a58592ebbe..dbca8d84acb723776c745d7fc2c213a32c389a71
    commit 2cdb267cf98a27084198ec5a010874ad4978926f (HEAD -> master, origin/master, origin/HEAD)
    Author: junping.gou <junping.gou@superxon.cn>
    Date:   Tue Mar 3 11:49:45 2020 +0800

        <E5><A2><9E><E5><8A><A0><E7><83><BD><E7><81><AB><E5><8A><A0><E5><AF><86><E6><B5><8B><E8><AF><95><E7><94><A8><E4><BE><8B>

    M       02 ci/02 firmware/hlt/Libraries/TestEncryption.py
    M       02 ci/02 firmware/hlt/TestCases/TestEncryptionCases.robot

    commit 34516865a021550389ae8b763bf119c7ab3dbe37
    Merge: e0a4f2e 19766a7
    Author: zesong.xiang <zesong.xiang@superxon.cn>
    Date:   Tue Mar 3 11:38:37 2020 +0800

        Merge branch 'develop_fh' into 'master'

        <E9><87><8D><E6><96><B0><E4><B8><8A><E4><BC><A0><E4><BB><A3><E7><A0><81><E5><88><B0><E6><96><B0><E5><88><86><E6><94><AF>

        See merge request project/sfp_plus_10g_epon_olt!296

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

Git删除本地分支的命令
---------------------------

git branch -D <branch_name>

Git查看两个版本之间差异
-----------------------------

* 只看两个版本之间提交的文件

git diff --name-only <commit_id1>..<commit_id2>

.. code::


    E:\CODE\sfp_plus_10g_epon_olt>git diff --name-only 5ad7a96665058bc8cf4589181f4e3db4ced36bb9..ae2c29bd6136e7424756b2ca61878dfb47c96d07
    02 ci/02 firmware/hlt/Libraries/Common.py
    02 ci/02 firmware/hlt/Libraries/TestBackupFlush.py
    02 ci/02 firmware/hlt/Libraries/TestBase.py
    02 ci/02 firmware/hlt/Libraries/TestCalculation.py
    02 ci/02 firmware/hlt/Libraries/TestCommittee.py
    02 ci/02 firmware/hlt/Libraries/TestEncryption.py
    02 ci/02 firmware/hlt/Libraries/TestFeatures.py
    02 ci/02 firmware/hlt/Libraries/TestI2c.py
    02 ci/02 firmware/hlt/Libraries/TestPowerFunction.py
    02 ci/02 firmware/hlt/Libraries/TestReliability.py
    02 ci/02 firmware/hlt/Libraries/TestTecDac.py
    02 ci/02 firmware/hlt/Libraries/TestTxDisableAndTrigger.py
    02 ci/02 firmware/hlt/Libraries/TestWarningFlag.py
    02 ci/02 firmware/hlt/TestCases/CommonOperation.robot
    02 ci/02 firmware/hlt/TestCases/TestBackupFlushCases.robot
    02 ci/02 firmware/hlt/TestCases/TestCalculation.robot
    02 ci/02 firmware/hlt/TestCases/TestCommittee.robot
    02 ci/02 firmware/hlt/TestCases/TestEncryptionCases.robot
    02 ci/02 firmware/hlt/TestCases/TestFeaturesCases.robot
    02 ci/02 firmware/hlt/TestCases/TestI2cCases.robot
    02 ci/02 firmware/hlt/TestCases/TestPowerFunction.robot
    02 ci/02 firmware/hlt/TestCases/TestReliabilityCases.robot
    02 ci/02 firmware/hlt/TestCases/TestTxDisableAndTrigger.robot
    02 ci/02 firmware/hlt/TestCases/TestWarningFlag.robot
    02 ci/04 common/control.sh
    02 ci/04 common/release_firmware.py
    04 src/03 software/MainPortal.exe
    04 src/03 software/common/src/OLT_SFPPLUS.c

如果不加--name-only则会显示详细差异





参考:
-------------
https://blog.csdn.net/richard_jason/article/details/53188200
https://blog.viktoradam.net/2018/07/26/githooks-auto-install-hooks/
https://www.viget.com/articles/two-ways-to-share-git-hooks-with-your-team/













