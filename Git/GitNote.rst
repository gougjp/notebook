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


