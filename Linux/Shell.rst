Shell Note
==============

jenkins 服务器日志空间清理
------------------------------

.. code::

    set +x

    max_days=180

    while true
    do
        used=$(df -h | grep '/dev/dm-0' | awk -F " " '{print $(NF-1)}' | tr -d %)
        echo "Used:${used}"
        if [ ${used} -ge 85 ]; then
            cd ${JENKINS_HOME}/userContent
        
            echo ${max_days}
            max_days=$(($max_days-2))
            if [ ${max_days} -lt 30 ]; then
                break
            fi
        
            find project/ -maxdepth 2 -mindepth 2 -type d -mtime +${max_days} | xargs -I {} bash -c "echo '{}'; rm -rf '{}'"
            find new_project/ -maxdepth 2 -mindepth 2 -type d -mtime +${max_days} | xargs -I {} bash -c "echo '{}'; rm -rf '{}'"
            find platform/ -maxdepth 2 -mindepth 2 -type d -mtime +${max_days} | xargs -I {} bash -c "echo '{}'; rm -rf '{}'"
            echo "Clean up success"
        else
            echo "Do not need to clean up"
            break
        fi
    done

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

在Linux下免用户名密码执行git命令
--------------------------------------

1. 在用户的(home目录)~/目录下新建.git-credentials文件; 并在文件中输入如下信息

.. code::

    http://<username>:<password>@<serverip>
    
    username: git提交时用到的用户名
    password: git提交时用到的密码
    serverip: git服务器IP
    
2. 执行如下命令

.. code::

    git config --global credential.helper store
    
执行完成后在home目录下会生成一个.gitconfig文件, 内容如下:

.. code::

    jenkins@superxon33:~$ cat .gitconfig
    [credential]
            helper = store

3. 执行如下命令配置用户和邮箱

.. code::

    git config --global user.name "bryan.sun"
    git config --global user.email "bryan.sun@gmail.com"

执行完成后会在.gitconfig文件中增加如下信息, 内容如下:

.. code::

    jenkins@superxon33:~$ cat .gitconfig
    [credential]
            helper = store
    [user]
            email = bryan.sun@gmail.com
            name = bryan.sun

for, while, if else写在一行
-------------------------------------

.. code::

    #判断文件test.txt是否存在且不为空, 如果存在且不为空, 输出111, 否则输出222
    if [ -s test.txt ]; then echo 111; else echo 222; fi

    #也可以包括elif
    if [ -s test.txt ]; then echo 111; elif [ -s make.sh ]; then echo 222; else echo 333; fi

    #if在for循环内
    for i in {1..10}; do if [ ! -f git_log ]; then sleep 1; echo "git_log is not exist";fi; done

    #更复杂一点的
    for i in {1..10}; do if [ ! -f git_log ]; then sleep 1; echo "git_log is not exist"; else echo "git_log is exist"; break; fi; done

    #while循环, 这样容易导致死循环
    while [ ! -f git_log ]; do sleep 1; done
    while true;  do if [ ! -f git_log ]; then sleep 1; echo "git_log is not exist"; else echo "git_log is exist"; break; fi; done

Shell Parameter Expansion -- Shell 参数扩展
-------------------------------------------------

.. role:: red

* **:red:`${parameter:−word}`**

如果parameter没有设置或者为空，则替换为word；否则替换为parameter的值。


