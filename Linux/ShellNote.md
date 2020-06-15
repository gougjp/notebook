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
