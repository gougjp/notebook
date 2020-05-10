Shell Note
==============

jenkins 服务器日志空间清理
------------------------------

.. code::

    set +x

    used=$(df -h | grep '/dev/dm-0' | awk -F " " '{print $(NF-1)}' | tr -d %)
    echo "Used:${used}"
    if [ ${used} -ge 95 ]; then
        cd ${JENKINS_HOME}/userContent

        find project/ -maxdepth 2 -mindepth 2 -type d -mtime +150 | xargs -I {} bash -c "echo '{}'; rm -rf '{}'"
        find new_project/ -maxdepth 2 -mindepth 2 -type d -mtime +150 | xargs -I {} bash -c "echo '{}'; rm -rf '{}'"
        find platform/ -maxdepth 2 -mindepth 2 -type d -mtime +150 | xargs -I {} bash -c "echo '{}'; rm -rf '{}'"
        echo "Clean up success"
    else
        echo "Do not need to clean up"
    fi

find的-mtime +150参数表示找出150天前的文件

Ubuntu 服务查看命令
--------------------------------

**service --status-all**

查看所有服务

**service 服务名 start**

启动服务

**service 服务名 stop**

停止服务

**service 服务名 restart**

重启服务

**service 服务名 status**

查看服务状态





