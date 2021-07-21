# Shell Note

## xargs后面跟多条命令

```Shell
#这里执行了grep和echo两条命令
find ./ -iname "*pusch*.csv" -print0 | xargs -0 -I {} bash -c "grep 'numOfDlHarqSubframesPacked' {}; echo {}"
#其中bash可以改成sh:
find ./ -iname "*pusch*.csv" -print0 | xargs -0 -I {} sh -c "grep 'numOfDlHarqSubframesPacked' {}; echo {}"
```

## find 命令

- <font color='red'>**-prune选项**</font>

该选项用于排除目录树中的某个目录, 它的功能就是: "如果文件是目录, 就不要向下遍历这个目录"

```Shell
#查找当前路径下的所有文件和目录
$ find .
.
./area.pm
./C
./C/temp
./C/f2.c
./test.c
./temp
./temp/a.c

#这里find查找当前目录下的所有文件, 但是如果它是一个目录,
#-prune选项将不允许进一步进入这个路径下
$ find . -prune
.

#找出只出现在当前目录下的文件和目录, '! -name .'表示除了当前目录以外的所有文件
#该命令的结果为, 除了当前目录以外的所有文件及目录, 目录不会被遍历, 只有目录名
$ find . ! -name .
./area.pm
./C
./C/temp
./C/f2.c
./test.c
./temp
./temp/a.c

$ find . ! -name . -prune
./area.pm
./C
./test.c
./temp

#跟上例结果相同的另一种方法
$ find ./* -prune
./area.pm
./CPP
./temp
./test.c
```

<font color="red">记住: find命令可以接收一个路径或者一系列路径来搜索, ./\*会得到当前目录下所有的文件和目录, 因此从这个列表中, prune将阻止对子目录的遍历; 但是这种方法不推荐, 因为 1.我们应该只给出查找的路径中的目录, 2.如果文件和目录的列表巨大, 这个命令将会出现性能问题</font>

```Shell
#'-name temp'找出所有名字为temp的文件, prune表示不要遍历名字为temp的目录; 因此结果为所有名字为temp的文件和目录
$ find . -name temp -prune
./C/temp
./temp

#-o是OR操作符, 查找将修剪掉名字为temp的目录.由于OR条件,
#所有其他文件会被打印出来(除了出现在temp目录下的文)
#默认情况下, find命令打印所有匹配到规则的文件; 但是一旦指定了-print选项,
#它就只打印有明确打印指令的文件, 在这个命令中, -print被关联到条件操作符OR的另一边, 因此第一部分得到的将不会有任何打印
$ find . -name temp -prune -o -print
.
./area.pm
./C
./C/f2.c
./test.c

#另一个类似的例子, 第一部分-type f查找当前目录下所有类型为文件的文件, 这部分不会被打印出来, -o -print会打印其他类型的文件, 比如当前目录下的所有目录
find . -type f -o -print

#找出除了temp目录下面的所有文件, 如果存在temp目录, 也会被打印出来的; 这里第一部分和第二部分都显示指定了print指令, 所以都会被打印出来
$ find . -name temp -prune -print -o -print
.
./area.pm
./C
./C/temp
./C/f2.c
./test.c
./temp

#找出除了C目录以外的的所有.c结尾的文件
#第一部分修剪掉C目录, 第二步分打印除了C目录以外的所有.c文件
$ find . -type d -name C -prune -o -name "*.c" -print
./test.c
./temp/a.c

#找出所有除了目录C和tempy以外的所有.c文件
#要为-name选项指定多个目录, 应该使用-o作为OR条件, 如括号中的-o;
#括号两边必须要有空格, 括号需要被转义
$ find . -type d \( -name C -o -name temp \) -prune -o -name "*.c" -print
./test.c

#查找一天之内修改的文件, 除了temp目录下的所有文件
$ find . -type d  -name temp -prune -o -mtime -1 -print
.
./area.pm
./C
./C/temp
./C/f2.c
./test.c

#查找一天之内修改的普通文件, 除了temp目录下的所有文件
$ find . -type d  -name temp -prune -o  -type f -mtime -1 -print
./area.pm
./C/temp
./C/f2.c
./test.c

#找出所有权限为644的文件, 除了文件名中包括temp的文件
$ find . -name "temp" -prune -o -perm 644 -print
./area.pm
./C/f2.c

#找出所有权限为644的文件, 除了全名为./temp的文件
$ find . -wholename "./temp" -prune -o -perm 644 -print
./area.pm
./C/temp
./C/f2.c

$ find . -name temp -exec test '{}' = "./temp" \; -prune -o -perm 644 -print
./area.pm
./C/temp
./C/f2.c

#-inum指定文件的iNode编号
$ find .  -inum 17662059 -prune -o -perm 644 -print
./area.pm
./C/temp
./C/f2.c

#找出除了C目录以外的所有.c文件
$ find . ! -path "./C/*" -name "*.c"
./test.c
./temp/a.c
```

参考文档:
http://blog.csdn.net/mrtitan/article/details/11068879
http://www.cnblogs.com/skynet/archive/2010/12/25/1916873.html
http://www.cnblogs.com/wangkangluo1/archive/2012/09/06/2673030.html


## jenkins 服务器日志空间清理

```Shell
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
```

find的-mtime +150参数表示找出150天前的文件

## Ubuntu 服务查看命令

- 查看所有服务

```Shell
service --status-all
```

- 启动服务

```Shell
service 服务名 start
```

- 停止服务

```Shell
service 服务名 stop
```

- 重启服务

```Shell
service 服务名 restart
```

- 查看服务状态

```Shell
service 服务名 status
```

## 在Linux下免用户名密码执行git命令

- 在用户的(home目录)~/目录下新建.git-credentials文件; 并在文件中输入如下信息

```
http://<username>:<password>@<serverip>

username: git提交时用到的用户名
password: git提交时用到的密码
serverip: git服务器IP
```

- 执行如下命令

```
git config --global credential.helper store
```

执行完成后在home目录下会生成一个.gitconfig文件, 内容如下:

```Shell
jenkins@superxon33:~$ cat .gitconfig
[credential]
        helper = store
```

- 执行如下命令配置用户和邮箱

```Shell
git config --global user.name "bryan.sun"
git config --global user.email "bryan.sun@gmail.com"
```

执行完成后会在.gitconfig文件中增加如下信息, 内容如下:

```Shell
jenkins@superxon33:~$ cat .gitconfig
[credential]
        helper = store
[user]
        email = bryan.sun@gmail.com
        name = bryan.sun
```

## for, while, if else写在一行

```Shell
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
```

## Sed 常用命令

- 插入行

```Shell
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
```

- 替换

```Shell
#将文件file中的text替换为replace, 替换每一行的第一个text, 如果某一行有多个text, 后面的几个都不会被替换
sed -i 's/text/replace/' file

#将文件file中的text替换为replace, 替换每一行的所有text
sed -i 's/text/replace/g' file

#只替换第20行
sed -i '20s/text/replace/g' file
```

- 删除行

```Shell
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
```

- 在行首行位加HTML标签

```Shell
# 这里分号用于分割命令; q退出命令, 这样程序只处理第一行文本
sed -n 's/^/<h1>/;s/$/<\/h1>p;q' rime.txt
```

## Centos 下卸载包

rpm -qa | grep <packagename>
rpm -e --nodeps <fullpackagename>

## Centos 下安装Maven最新版本

- 下载: http://maven.apache.org/download.cgi, 选择二进制包(Binary tar.gz archive)下载

- 解压到/opt下, 其他目录下也可以

- 配置/etc/profile

```Shell
export MAVEN_HOME=/opt/apache-maven-3.6.3
export PATH=$PATH:$MAVEN_HOME/bin

source /etc/profile
```

## Linux下挂载网盘

- 在/etc/fstab文件中增加如下内容

```Shell
//<ip>/dll    /home/releasedll    cifs    rw,dir_mode=0777,file_mode=0777,username=<user>,password=<password>  0 0
```

- 然后执行如下命令

```Shell
mount -a
```

## Centos 7 升级 sqlite3

在 centos 7 上面运行 django 3.1 开发服务器时出现以下错误:

```Shell
django.core.exceptions.ImproperlyConfigured: SQLite 3.8.3 or later is required (found 3.7.17).
```

原因时系统自带 sqlite3 版本太低，解决方法是升级就可以了。

```Shell
# 下载源码
wget https://www.sqlite.org/2019/sqlite-autoconf-3290000.tar.gz
# 编译
tar zxvf sqlite-autoconf-3290000.tar.gz 
cd sqlite-autoconf-3290000/
./configure --prefix=/usr/local
make && make install

# 替换系统低版本 sqlite3
mv /usr/bin/sqlite3  /usr/bin/sqlite3_old
ln -s /usr/local/bin/sqlite3   /usr/bin/sqlite3
echo "/usr/local/lib" > /etc/ld.so.conf.d/sqlite3.conf
ldconfig
sqlite3 -version
```

## 终端混乱解决

有时候会出现在终端输入命令时, 命令不会显示出来, 但是敲完回车后命令仍然能够执行成功; 如果没有命令一直敲回车, 则一直显示命令提示符, 且不能回车, 只有输入Ctrl+C过后才能回车; 
此时在终端输入**stty sane**命令则可恢复终端.

## centos7使用系统自带Java配置JAVA环境变量

如果执行echo $JAVA_HOME发现返回为空, 则说明该环境变量没有配置

1. 执行以下命令查询openjdk安装位置:

```Shell
[root@bogon ~]# which java
/usr/bin/java
[root@bogon ~]# ls -lrt /usr/bin/java
lrwxrwxrwx. 1 root root 22 Jul  9 16:01 /usr/bin/java -> /etc/alternatives/java
[root@bogon ~]# ls -lrt /etc/alternatives/java
lrwxrwxrwx. 1 root root 71 Jul  9 16:01 /etc/alternatives/java -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-1.el7.x86_64/jre/bin/java
```

这里找到的Java安装路径为: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-1.el7.x86_64

2. 将环境变量配置到 /etc/profile 文件中

```Shell
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-1.el7.x86_64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```

3. 执行source /etc/profile 命令后环境变量生效, 此时再执行echo $JAVA_HOME则能看到该环境变量的值

