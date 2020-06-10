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

* **${parameter:−word}**

如果parameter没有设置或者为空，则替换为word；否则替换为parameter的值。

.. code::

    #varB没有设置
    [jgou@localhost ~]$ varA=123
    [jgou@localhost ~]$ unset varB; echo $varB

    [jgou@localhost ~]$ echo ${varB:-${varA}}
    123
    [jgou@localhost ~]$ echo $varB

    [jgou@localhost ~]$

    #varB设置为空
    [jgou@localhost ~]$ varA=123
    [jgou@localhost ~]$ unset varB; varB=""
    [jgou@localhost ~]$ echo ${varB:-${varA}}
    123
    [jgou@localhost ~]$ echo ${varB}

    [jgou@localhost ~]$
    
    #varB为456
    [jgou@localhost ~]$ varA=123
    [jgou@localhost ~]$ varB=456
    [jgou@localhost ~]$ echo ${varB:-${varA}}
    456
    [jgou@localhost ~]$ echo ${varB}
    456
    
    # 举例
    [jgou@localhost ~]$ unset x;y="abc def"; echo "/${x:-'XYZ'}/${y:-'XYZ'}/$x/$y/"
    /'XYZ'/abc def//abc def/

* **${parameter:=word}**

如果parameter没有设置或者为空，则shell扩展word并将结果赋给parameter，然后替换为parameter的值。对于位置参数和特殊参数，不可以这样进行赋值。

.. code::

    #varB没有设置
    [jgou@localhost ~]$ varA=123
    [jgou@localhost ~]$ unset varB; echo $varB

    [jgou@localhost ~]$ echo ${varB:=${varA}}
    123
    [jgou@localhost ~]$ echo $varB
    123
    [jgou@localhost ~]$

    #varB设置为空
    [jgou@localhost ~]$ varA=123
    [jgou@localhost ~]$ unset varB; varB=""
    [jgou@localhost ~]$ echo ${varB:-${varA}}
    123
    [jgou@localhost ~]$ echo ${varB}
    123
    [jgou@localhost ~]$

    #varB为456
    [jgou@localhost ~]$ varA=123
    [jgou@localhost ~]$ varB=456
    [jgou@localhost ~]$ echo ${varB:-${varA}}
    456
    [jgou@localhost ~]$ echo ${varB}
    456

    # 举例
    [jgou@localhost ~]$ unset x;y="abc def"; echo "/${x:='XYZ'}/${y:='XYZ'}/$x/$y/"
    /'XYZ'/abc def/'XYZ'/abc def/

* **${parameter:?word}**

如果parameter没有设置或者为空，shell扩展word并将结果写入标准错误中(如果没有给出word,则给出一条大意相同的信息)。如果当前的shell是交互式的，退出shell。否则，替换为parameter的值。

.. code::

    [jgou@localhost ~]$ varA=123
    [jgou@localhost ~]$ unset varB; echo ${varB}

    [jgou@localhost ~]$ echo ${varB:?${varA}}
    bash: varB: 123
    [jgou@localhost ~]$ echo ${varB:?456}
    bash: varB: 456
    [jgou@localhost ~]$ echo ${varB:?${varC}}
    bash: varB: 

    [ian@pinguino ~]$ ( unset x;y="abc def"; echo "/${x:?'XYZ'}/${y:?'XYZ'}/$x/$y/" ) >so.txt 2>se.txt
    [ian@pinguino ~]$ cat so.txt
    [ian@pinguino ~]$ cat se.txt
    -bash: x: XYZ

* **${parameter:+word}**

如果parameter没有设置或者为空，则不作替换。否则替换为扩展后的word。

.. code::

    [jgou@localhost ~]$ varA=123
    [jgou@localhost ~]$ unset varB; echo ${varB}

    [jgou@localhost ~]$ echo ${varB:+${varA}}

    [jgou@localhost ~]$ varB=456
    [jgou@localhost ~]$ echo ${varB:+${varA}}
    123
    [jgou@localhost ~]$ echo $varB
    456

    # 举例
    [jgou@localhost ~]$ unset x;y="abc def"; echo "/${x:+'XYZ'}/${y:+'XYZ'}/$x/$y/"
    //'XYZ'//abc def/

* **${parameter:offset}**
* **${parameter:offset:length}**

扩展为parameter中从offset开始的不超过length的字符。如果没有指定length，扩展为parameter中从offset开始的子字符串。length和offset都是算术表达式。这又叫做"子字符串扩展".
length的值必须是一个大于或等于0的数字. 如果length小于0, 它就会被当成parameter所表示的字符串中从结尾开始的偏移量. 如果parameter是"@", 结果就是从offset开始的第length个位置参数; 如果parameter是带有"@"或"*"下标的下标数组名, 则结果是该数组中从${parameter[offset]}开始的length个元素. 负的偏移量是从数组中比最大的下标大一的数字开始的。对键值数组进行子字符串扩展的结果没有定
义。注意，负数的偏移量与冒号之间至少得有一个空格，这样可以避免与":-"扩展相混淆。查找子字
符串的下标是从0 开始的；但是如果使用了位置参数，则默认从1 开始。如果使用位置参数时offset是0，则会把$@添加到结果前面.

.. code::

    [jgou@localhost ~]$ string=01234567890abcdefgh
    [jgou@localhost ~]$ echo ${string:7}
    7890abcdefgh
    [jgou@localhost ~]$ echo ${string:7:0}

    [jgou@localhost ~]$ echo ${string:7:2}
    78
    [jgou@localhost ~]$ echo ${string:7:-2}
    7890abcdef
    [jgou@localhost ~]$ echo ${string: -7}
    bcdefgh
    [jgou@localhost ~]$ echo ${string: -7:0}

    [jgou@localhost ~]$ echo ${string: -7:2}
    bc
    [jgou@localhost ~]$ echo ${string: -7:-2}
    bcdef
    [jgou@localhost ~]$ 


    [jgou@localhost ~]$ set -- 01234567890abcdefgh
    [jgou@localhost ~]$ echo ${1:7}
    7890abcdefgh
    [jgou@localhost ~]$ echo ${1:7:0}

    [jgou@localhost ~]$ echo ${1:7:2}
    78
    [jgou@localhost ~]$ echo ${1:7:-2}
    7890abcdef
    [jgou@localhost ~]$ echo ${1: -7}
    bcdefgh
    [jgou@localhost ~]$ echo ${1: -7:0}

    [jgou@localhost ~]$ echo ${1: -7:2}
    bc
    [jgou@localhost ~]$ echo ${1: -7:-2}
    bcdef
    [jgou@localhost ~]$


    [jgou@localhost ~]$ array[0]=01234567890abcdefgh
    [jgou@localhost ~]$ echo ${array[0]:7}
    7890abcdefgh
    [jgou@localhost ~]$ echo ${array[0]:7:0}

    [jgou@localhost ~]$ echo ${array[0]:7:2}
    78
    [jgou@localhost ~]$ echo ${array[0]:7:-2}
    7890abcdef
    [jgou@localhost ~]$ echo ${array[0]: -7}
    bcdefgh
    [jgou@localhost ~]$ echo ${array[0]: -7:0}

    [jgou@localhost ~]$ echo ${array[0]: -7:2}
    bc
    [jgou@localhost ~]$ echo ${array[0]: -7:-2}
    bcdef
    [jgou@localhost ~]$ echo ${array}
    01234567890abcdefgh
    [jgou@localhost ~]$ echo ${#array[@]}   #数组中元素个数
    1

如果parameter是"@", 结果就是从offset开始的length个位置参数. 负的offset是相对于最大位置参数的, -1的offset是最后一个位置参数. 当length小于0时, 表达式错误:

.. code::

    [jgou@localhost ~]$ echo $@
    1 2 3 4 5 6 7 8 9 0 a b c d e f g h
    [jgou@localhost ~]$ echo ${@:7}
    7 8 9 0 a b c d e f g h
    [jgou@localhost ~]$ echo ${@:7:0}

    [jgou@localhost ~]$ echo ${@:7:2}
    7 8
    [jgou@localhost ~]$ echo ${@:7:-2}
    -bash: -2: substring expression < 0
    [jgou@localhost ~]$ echo ${@: -7:2}
    b c
    [jgou@localhost ~]$ echo ${@:0}
    -bash 1 2 3 4 5 6 7 8 9 0 a b c d e f g h
    [jgou@localhost ~]$ echo ${@:0:2}
    -bash 1
    [jgou@localhost ~]$ echo ${@: -7:0}

如果parameter是一个有索引的下标为'@'或者'*'的数组名, 则结果为从数组的${parameter[offset]}开始, length个元素. 负的offset是相对于数组最大索引的. 如果length小于0则出错.

.. code::

    [jgou@localhost ~]$ array=(0 1 2 3 4 5 6 7 8 9 0 a b c d e f g h)
    [jgou@localhost ~]$ echo ${array[@]}
    0 1 2 3 4 5 6 7 8 9 0 a b c d e f g h
    [jgou@localhost ~]$ echo ${array[@]:7}
    7 8 9 0 a b c d e f g h
    [jgou@localhost ~]$ echo ${array[@]:7:2}
    7 8
    [jgou@localhost ~]$ echo ${array[@]: -7:2}
    b c
    [jgou@localhost ~]$ echo ${array[@]: -7:-2}
    -bash: -2: substring expression < 0
    [jgou@localhost ~]$ echo ${array[@]:0}
    0 1 2 3 4 5 6 7 8 9 0 a b c d e f g h
    [jgou@localhost ~]$ echo ${array[@]:0:2}
    0 1
    [jgou@localhost ~]$ echo ${array[@]: -7:0}

    [jgou@localhost ~]$

* **${!prefix*}**
* **${!prefix@}**

扩展名字以prefix开头的变量,以特殊变量IFS的第一个字符分割. 如果使用了"@"，并且在双引号内扩展, 则每个变量都扩展成单独的单词.

.. code::

    [jgou@localhost ~]$ IFS="|"
    [jgou@localhost ~]$ varA=123
    [jgou@localhost ~]$ varB=456
    [jgou@localhost ~]$ varC=789
    [jgou@localhost ~]$ echo ${!var*}
    varA varB varC
    [jgou@localhost ~]$ echo "${!var*}"
    varA|varB|varC
    [jgou@localhost ~]$ echo ${!var@}
    varA varB varC
    [jgou@localhost ~]$ echo "${!var@}"
    varA varB varC

* **${!name[*]}**
* **${!name[@]}**

如果name是一个数组变量, 扩展成name数组下标或者键名的列表. 如果name不是不是数组变量, 当name变量存在则返回0, 如果name变量不存在则返回空. 如果使用了"@"，并且在双引号内扩展, 则每个变量都扩展成单独的单词.

.. code::

    [jgou@localhost ~]$ var=(a b c d e f g)
    [jgou@localhost ~]$ echo ${!var[*]}
    0 1 2 3 4 5 6
    [jgou@localhost ~]$ echo ${!var[@]}
    0 1 2 3 4 5 6
    [jgou@localhost ~]$ unset var
    [jgou@localhost ~]$ echo ${!var[*]}

    [jgou@localhost ~]$ echo ${!var[@]}

    [jgou@localhost ~]$ var=123
    [jgou@localhost ~]$ echo ${!var[*]}
    0
    [jgou@localhost ~]$ echo ${!var[@]}
    0
    [jgou@localhost ~]$ var=(a b c d e f g)
    [jgou@localhost ~]$ echo "${!var[@]}"
    0 1 2 3 4 5 6
    [jgou@localhost ~]$ echo "${!var[*]}"
    0 1 2 3 4 5 6

* **${#parameter}**

被替换成parameter扩展值的字符串的长度. 如果parameter是'*'或者'@', 则替换为位置参数的个数. 如果parameter是下标为'*'或者'@'的数组名, 则替换为数组中元素的个数. 如果parameter是一个负数下标作为索引的数组名, 这个数字被解释为相对于parameter最大索引, 所以负的下标是从数组结尾倒数的, 索引-1代表最后一个元素.

.. code::

    [jgou@localhost ~]$ var=0123456789abcdefg
    [jgou@localhost ~]$ echo ${#var}
    17
    [jgou@localhost ~]$ var=(0 1 2 3 4 5 6 7 8 9 a b c d e f g)
    [jgou@localhost ~]$ echo ${#var[*]}
    17
    [jgou@localhost ~]$ echo ${#var[@]}
    17

* **${parameter#word}**
* **${parameter##word}**

