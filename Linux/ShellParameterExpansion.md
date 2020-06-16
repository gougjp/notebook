# Shell Parameter Expansion -- Shell 参数扩展

- **<font color='red'>${parameter:−word}</font>**

如果parameter没有设置或者为空，则替换为word；否则替换为parameter的值。

```Shell
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
```

- **<font color='red'>${parameter:=word}</font>**

如果parameter没有设置或者为空，则shell扩展word并将结果赋给parameter，然后替换为parameter的值。对于位置参数和特殊参数，不可以这样进行赋值。

```Shell
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
```

- **<font color='red'>${parameter:?word}</font>**

如果parameter没有设置或者为空，shell扩展word并将结果写入标准错误中(如果没有给出word,则给出一条大意相同的信息)。如果当前的shell是交互式的，退出shell。否则，替换为parameter的值。

```Shell
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
```

- **<font color='red'>${parameter:+word}</font>**

如果parameter没有设置或者为空，则不作替换。否则替换为扩展后的word。

```Shell
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
```

- **<font color='red'>${parameter:offset}</font>**
- **<font color='red'>${parameter:offset:length}</font>**

扩展为parameter中从offset开始的不超过length的字符。如果没有指定length，扩展为parameter中从offset开始的子字符串。length和offset都是算术表达式。这又叫做"子字符串扩展".
length的值必须是一个大于或等于0的数字. 如果length小于0, 它就会被当成parameter所表示的字符串中从结尾开始的偏移量. 如果parameter是"@", 结果就是从offset开始的第length个位置参数; 如果parameter是带有"@"或"*"下标的下标数组名, 则结果是该数组中从${parameter[offset]}开始的length个元素. 负的偏移量是从数组中比最大的下标大一的数字开始的。对键值数组进行子字符串扩展的结果没有定
义。注意，负数的偏移量与冒号之间至少得有一个空格，这样可以避免与":-"扩展相混淆。查找子字
符串的下标是从0 开始的；但是如果使用了位置参数，则默认从1 开始。如果使用位置参数时offset是0，则会把$@添加到结果前面.

```Shell
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
```

如果parameter是"@", 结果就是从offset开始的length个位置参数. 负的offset是相对于最大位置参数的, -1的offset是最后一个位置参数. 当length小于0时, 表达式错误:

```Shell
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
```

如果parameter是一个有索引的下标为'@'或者'*'的数组名, 则结果为从数组的${parameter[offset]}开始, length个元素. 负的offset是相对于数组最大索引的. 如果length小于0则出错.

```Shell
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
```

- **<font color='red'>${!prefix\*}</font>**
- **<font color='red'>${!prefix@}</font>**

扩展名字以prefix开头的变量,以特殊变量IFS的第一个字符分割. 如果使用了"@"，并且在双引号内扩展, 则每个变量都扩展成单独的单词.

```Shell
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
```

- **<font color='red'>${!name[\*]}</font>**
- **<font color='red'>${!name[@]}</font>**

如果name是一个数组变量, 扩展成name数组下标或者键名的列表. 如果name不是不是数组变量, 当name变量存在则返回0, 如果name变量不存在则返回空. 如果使用了"@"，并且在双引号内扩展, 则每个变量都扩展成单独的单词.

```Shell
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
```

- **<font color='red'>${#parameter}</font>**

被替换成parameter扩展值的字符串的长度. 如果parameter是'*'或者'@', 则替换为位置参数的个数. 如果parameter是下标为'*'或者'@'的数组名, 则替换为数组中元素的个数. 如果parameter是一个负数下标作为索引的数组名, 这个数字被解释为相对于parameter最大索引, 所以负的下标是从数组结尾倒数的, 索引-1代表最后一个元素.

```Shell
[jgou@localhost ~]$ var=0123456789abcdefg
[jgou@localhost ~]$ echo ${#var}
17
[jgou@localhost ~]$ var=(0 1 2 3 4 5 6 7 8 9 a b c d e f g)
[jgou@localhost ~]$ echo ${#var[*]}
17
[jgou@localhost ~]$ echo ${#var[@]}
17
```

- **<font color='red'>${parameter#word}</font>**
- **<font color='red'>${parameter##word}</font>**

shell像文件名扩展中那样扩展word. 如果模式匹配parameter扩展值的开始, 那么parameter扩展值扩展的结果, 在'#'情况下将删除最短的匹配, 在'##'情况下将删除最长的匹配. 如果parameter是'@'或者'\*', 则模式删除操作依次应用于每个位置参数, 并且扩展的结果是一个列表. 如果parameter是一个下标为'@'或者'\*'的数组变量, 则模式删除操作依次应用于每个数组元素,并且扩展的结果是一个列表.

```Shell
[jgou@localhost ~]$ fineName=abcdefg.0123456.bdjeng.txt
[jgou@localhost ~]$ echo ${fineName#*.}
0123456.bdjeng.txt
[jgou@localhost ~]$ echo ${fineName##*.}
txt
[jgou@localhost ~]$ echo ${fineName#[a-z]*.}
0123456.bdjeng.txt

#这里模式[a-z]匹配第一个字符a, '*'号匹配中间的所有字符，点号匹配最后一个点号
[jgou@localhost ~]$ echo ${fineName##[a-z]*.}
txt
[jgou@localhost ~]$ echo ${fineName##[a]*.}
txt
[jgou@localhost ~]$ echo ${fineName##[d]*.}
abcdefg.0123456.bdjeng.txt

#下面两个开始位置不匹配, 所有不做任何删除操作， 因为第一个字符不匹配
[jgou@localhost ~]$ echo ${fineName#[0-9]*.}
abcdefg.0123456.bdjeng.txt
[jgou@localhost ~]$ echo ${fineName##[0-9]*.}
abcdefg.0123456.bdjeng.txt
[jgou@localhost ~]$ arryVar=(acde.txt edgs.pdf adsasdf.xls roeij.csv alsdkfjoerj.py alsdfj.bat asldfjk.sh)
[jgou@localhost ~]$ echo ${arryVar[@]#*.}
txt pdf xls csv py bat sh
[jgou@localhost ~]$ echo ${arryVar[*]#*.}
txt pdf xls csv py bat sh
[jgou@localhost ~]$ echo ${arryVar[@]##*.}
txt pdf xls csv py bat sh
[jgou@localhost ~]$ echo ${arryVar[*]##*.}
txt pdf xls csv py bat sh
```

- **<font color='red'>${parameter%word}</font>**
- **<font color='red'>${parameter%%word}</font>**

shell像文件名扩展中那样扩展word.如果模式匹配parameter扩展值的结尾,那么parameter扩展值扩展的结果,在'%'情况下将删除最短的匹配,在'%%'情况下将删除最长的匹配.如果parameter是'@'或者'\*',则模式删除操作依次应用于每个位置参数,并且扩展的结果是一个列表.如果parameter是一个下标为'@'或者'\*'的数组变量,则模式删除操作依次应用于每个数组元素,并且扩展的结果是一个列表.

```Shell
[jgou@localhost ~]$ fineName=abcdefg.0123456.bdjeng.txt
[jgou@localhost ~]$ echo ${fineName%.*}
abcdefg.0123456.bdjeng
[jgou@localhost ~]$ echo ${fineName%%.*}
abcdefg
[jgou@localhost ~]$ echo ${fineName%.*[a-z]}
abcdefg.0123456.bdjeng
[jgou@localhost ~]$ echo ${fineName%%.*[a-z]}
abcdefg
[jgou@localhost ~]$ echo ${fineName%%.*[t]}
abcdefg
[jgou@localhost ~]$ echo ${fineName%%.*[a]}
abcdefg.0123456.bdjeng.txt
[jgou@localhost ~]$ echo ${fineName%%.*[0]}
abcdefg.0123456.bdjeng.txt
[jgou@localhost ~]$ echo ${fineName%.*[0-9]}
abcdefg.0123456.bdjeng.txt
[jgou@localhost ~]$ echo ${fineName%%.*[0-9]}
abcdefg.0123456.bdjeng.txt
[jgou@localhost ~]$ arryVar=(acde.txt edgs.pdf adsasdf.xls roeij.csv alsdkfjoerj.py alsdfj.bat asldfjk.sh)
[jgou@localhost ~]$ echo ${arryVar[@]%.*}
acde edgs adsasdf roeij alsdkfjoerj alsdfj asldfjk
[jgou@localhost ~]$ echo ${arryVar[*]%.*}
acde edgs adsasdf roeij alsdkfjoerj alsdfj asldfjk
[jgou@localhost ~]$ echo ${arryVar[@]%%.*}
acde edgs adsasdf roeij alsdkfjoerj alsdfj asldfjk
[jgou@localhost ~]$ echo ${arryVar[*]%%.*}
acde edgs adsasdf roeij alsdkfjoerj alsdfj asldfjk
```

- **<font color='red'>${parameter/pattern/string}</font>**

shell像文件名扩展中那样扩展pattern.parameter被扩展,并且匹配pattern最长(贪婪匹配)的值被替换成string.如果pattern以/开头,pattern匹配到的所有部分都会被替换成string(如${var//\[0-9]\/'\-'}),而正常情况下只是第一个匹配到的
被替换。如果pattern以"#"开始，则它必须匹配parameter扩展值的开始部分。如果pattern以"%"开始，则它必须匹配parameter扩展值的结尾部分。如果string为null，pattern匹配到的部分将被删掉，pattern后面的/可以省略。如果启用了shell的nocasematch选项，则匹配不区分大小写。如果parameter是@或者*,替换操作轮流应用于每个位置参数，扩展的结果是列表。如果parameter是下标为@或者*的数组变量，替换操作轮流应用于数组的每个元素，扩展的结果是列表。

```Shell
[jgou@localhost ~]$ mystr="This string is a simple test string"
[jgou@localhost ~]$ echo ${mystr/string/chars}
This chars is a simple test string
[jgou@localhost ~]$ echo ${mystr//string/chars}
This chars is a simple test chars
[jgou@localhost ~]$ echo ${mystr/string/}
This is a simple test string
[jgou@localhost ~]$ echo ${mystr/string}
This is a simple test string
[jgou@localhost ~]$ echo ${mystr//string/}
This is a simple test
[jgou@localhost ~]$ echo ${mystr//string}
This is a simple test
#string后面有一个空格，之能匹配第一个位置
[jgou@localhost ~]$ echo ${mystr//string }
This is a simple test string

[jgou@localhost ~]$ var=alsdkfj345alkjg675642aslfj.pdf
[jgou@localhost ~]$ echo ${var//[a-z]}
345675642.
[jgou@localhost ~]$ echo ${var/#[a-z]}
lsdkfj345alkjg675642aslfj.pdf
[jgou@localhost ~]$ echo ${var/#[a-z]/-}
-lsdkfj345alkjg675642aslfj.pdf
[jgou@localhost ~]$ echo ${var/%[a-z]}
alsdkfj345alkjg675642aslfj.pd
[jgou@localhost ~]$ echo ${var/%[a-z]/-}
alsdkfj345alkjg675642aslfj.pd-
[jgou@localhost ~]$ arryVar=(acde.txt edgs.pdf adsasdf.xls roeij.csv alsdkfjoerj.py alsdfj.bat asldfjk.sh)
[jgou@localhost ~]$ echo ${arryVar[@]/.*/-}
acde- edgs- adsasdf- roeij- alsdkfjoerj- alsdfj- asldfjk-
[jgou@localhost ~]$ echo ${arryVar[@]//[a-z]/-}
----.--- ----.--- -------.--- -----.--- -----------.-- ------.--- -------.--
[jgou@localhost ~]$ echo ${arryVar[@]/[a-z]/1}
1cde.txt 1dgs.pdf 1dsasdf.xls 1oeij.csv 1lsdkfjoerj.py 1lsdfj.bat 1sldfjk.sh
```

- **<font color='red'>${parameter^pattern}</font>**
- **<font color='red'>${parameter^^pattern}</font>**
- **<font color='red'>${parameter,pattern}</font>**
- **<font color='red'>${parameter,,pattern}</font>**
- **<font color='red'>${parameter~pattern}</font>**
- **<font color='red'>${parameter~~pattern}</font>**

这些扩展修改parameter中字母字符的大小写,shell像文件名扩展中那样扩展pattern。parameter扩展值的每一个字符都要对pattern进行测试,如果它匹配这个模式,就会转换这个字符的大小写.模式不应该尝试匹配多个字符.'^'操作将pattern匹配到的字母从小写转换成大写,','操作将匹配到的大写字母转换成小写,'~'将匹配到的字符转换成相反的大小写。'^^',',,'和'~~'扩展转换扩展值中的每一个匹配到的字符;而'^',','和'~'扩展只匹配和转换扩展值中的第一个字符.如果pattern被省略，则它会被当成'?',匹配任意字符。如果parameter是'@'或在'\*',大小写转换操作轮流应用于每个位置参数,扩展的结果是列表。如果parameter是下标为@或者\*的数组变量,大小写转换操作轮流应用于数组的每个元素，扩展的结果是列表。

```Shell
[jgou@localhost ~]$ var=asfsd1353asd.txt
[jgou@localhost ~]$ echo ${var^[a-z]}
Asfsd1353asd.txt
[jgou@localhost ~]$ echo ${var^[a]}
Asfsd1353asd.txt
[jgou@localhost ~]$ echo ${var^[s]}
asfsd1353asd.txt
[jgou@localhost ~]$ echo ${var^[0-9]}
asfsd1353asd.txt
[jgou@localhost ~]$ echo ${var^}
Asfsd1353asd.txt
[jgou@localhost ~]$ echo ${var^^[a-z]}
ASFSD1353ASD.TXT
[jgou@localhost ~]$ echo ${var^^[a]}
Asfsd1353Asd.txt
[jgou@localhost ~]$ echo ${var^^[s]}
aSfSd1353aSd.txt
[jgou@localhost ~]$ echo ${var^^[0-9]}
asfsd1353asd.txt
[jgou@localhost ~]$ echo ${var^^}
ASFSD1353ASD.TXT

[jgou@localhost ~]$ var=ABCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,[a-z]}
aBCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,[a]}
ABCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,[A]}
aBCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,[a-f]}
aBCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,[a-b]}
aBCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,[b-z]}
ABCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,[0-9]}
ABCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,}
aBCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,,[a-z]}
abcdef1353ghij.txt
[jgou@localhost ~]$ echo ${var,,[a]}
ABCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,,[A]}
aBCDEF1353GHIJ.TXT
#注意下面4个,下面4个大小写范围不同,导致结果不同, 根据文件名扩展, 在许多语言区域中[a-dx-z]和[abcdxyz]是不等价的, 具体见后面 文件名扩展中的字符集:
[jgou@localhost ~]$ echo ${var,,[a-f]}
abcdeF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,,[a-b]}
aBCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,,[A-B]}
abCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,,[A-F]}
abcdef1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,,[0-9]}
ABCDEF1353GHIJ.TXT
[jgou@localhost ~]$ echo ${var,,}
abcdef1353ghij.txt

[jgou@localhost ~]$ var=AbCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~[a-z]}
abCdEf1353GhIj.txT
[jgou@localhost ~]$ echo ${var~[a]}
AbCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~[A]}
abCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~[a-f]}
abCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~[A-F]}
abCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~[a-b]}
abCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~[a-B]}
abCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~[0-9]}
AbCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~}
abCdEf1353GhIj.txT
[jgou@localhost ~]$ echo ${var~~[a-z]}
aBcDeF1353gHiJ.tXt
[jgou@localhost ~]$ echo ${var~~[a]}
AbCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~~[a-f]}
aBcDeF1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~~[A]}
abCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~~[A-F]}
aBcDeF1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~~[a-b]}
aBCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~~[A-B]}
aBCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~~[0-9]}
AbCdEf1353GhIj.TxT
[jgou@localhost ~]$ echo ${var~~}
aBcDeF1353gHiJ.tXt

[jgou@localhost ~]$ arryVar=(acde.txt ELJSFDLS.PDF aaSDFsaSdf.XlS roeij.csv alSDFKfjKFrj.pY AlssFSLj.bat Asldfjk.sh nalks.cpp)
[jgou@localhost ~]$ echo ${arryVar[@]^[a-z]}
Acde.txt ELJSFDLS.PDF AaSDFsaSdf.XlS Roeij.csv AlSDFKfjKFrj.pY AlssFSLj.bat Asldfjk.sh Nalks.cpp
[jgou@localhost ~]$ echo ${arryVar[@]^[A-Z]}
acde.txt ELJSFDLS.PDF aaSDFsaSdf.XlS Roeij.csv alSDFKfjKFrj.pY AlssFSLj.bat Asldfjk.sh Nalks.cpp
[jgou@localhost ~]$ echo ${arryVar[*]^[A-Z]}
acde.txt ELJSFDLS.PDF aaSDFsaSdf.XlS Roeij.csv alSDFKfjKFrj.pY AlssFSLj.bat Asldfjk.sh Nalks.cpp
[jgou@localhost ~]$ echo ${arryVar[*]^[a-z]}
Acde.txt ELJSFDLS.PDF AaSDFsaSdf.XlS Roeij.csv AlSDFKfjKFrj.pY AlssFSLj.bat Asldfjk.sh Nalks.cpp
[jgou@localhost ~]$ echo ${arryVar[@]^^[a-z]}
ACDE.TXT ELJSFDLS.PDF AASDFSASDF.XLS ROEIJ.CSV ALSDFKFJKFRJ.PY ALSSFSLJ.BAT ASLDFJK.SH NALKS.CPP
[jgou@localhost ~]$ echo ${arryVar[@]^^}
ACDE.TXT ELJSFDLS.PDF AASDFSASDF.XLS ROEIJ.CSV ALSDFKFJKFRJ.PY ALSSFSLJ.BAT ASLDFJK.SH NALKS.CPP
```

- **<font color='red'>文件名扩展中的字符集:</font>**

字符集两端的字符均包括在匹配字符中。在C语言区域中,\[a-dx-z]和\[abcdxyz]是等价的;而在许多区域语言中,字符都是按词典顺序排列的,导致这两种通常是不等价的,如\[a-dx-z]通常等价于\[aAbBcCdxXyYz].为了方括号表达式中使用在传统意义上的范围,可以把环境变量LC_COLLATE或者LC_ALL设为"C"以强制使用C语言区域

```Shell
#下例可见:[a-z]不匹配Z,[A-Z]不匹配a,[a-z]匹配所有的小写字母,[A-Z]匹配所有的大写字母
[jgou@localhost ~]$ var=ABCDEFGHIJKLMNOPQRSTUVWXYZ
[jgou@localhost ~]$ echo ${var,,[a-z]}
abcdefghijklmnopqrstuvwxyZ
[jgou@localhost ~]$ echo ${var,,[A-Z]}
abcdefghijklmnopqrstuvwxyz

[jgou@localhost ~]$ var1=abcdefghijklmnopqrstuvwxyz
[jgou@localhost ~]$ echo ${var1^^[a-z]}
ABCDEFGHIJKLMNOPQRSTUVWXYZ
[jgou@localhost ~]$ echo ${var1^^[A-Z]}
aBCDEFGHIJKLMNOPQRSTUVWXYZ

[jgou@localhost ~]$ var=aBcDeFgHiJkLmNoPqRsTuVwXyZ
[jgou@localhost ~]$ echo ${var~~[a-z]}
AbCdEfGhIjKlMnOpQrStUvWxYZ
[jgou@localhost ~]$ echo ${var~~[A-Z]}
abCdEfGhIjKlMnOpQrStUvWxYz
[jgou@localhost ~]$ echo ${var~~[a-zZ]}
AbCdEfGhIjKlMnOpQrStUvWxYz
[jgou@localhost ~]$ echo ${var~~[aA-Z]}
AbCdEfGhIjKlMnOpQrStUvWxYz
```

可以看出此时,\[a-z]等价于\[aAbBcCdDeEfFgGhHiIjJkKlLmMnNoOpPqQrRsStTuUvVwWxXyYz],\[A-Z]等价于\[AbBcCdDeEfFgGhHiIjJkKlLmMnNoOpPqQrRsStTuUvVwWxXyYzZ],而\[a-zZ]和\[aA-Z]都等价于\[a-zA-Z]

```Shell
#查看当前LC_COLLATE和LC_ALL的值
[jgou@localhost ~]$ locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
#设置LC_ALL为"C"
[jgou@localhost ~]$ export LC_ALL="C"
[jgou@localhost ~]$ var=aBcDeFgHiJkLmNoPqRsTuVwXyZ
#此时只匹配到小写字符
[jgou@localhost ~]$ echo ${var~~[a-z]}
ABCDEFGHIJKLMNOPQRSTUVWXYZ
#此时只匹配到大写字符
[jgou@localhost ~]$ echo ${var~~[A-Z]}
abcdefghijklmnopqrstuvwxyz
#此时匹配到所有大小写字符
[jgou@localhost ~]$ echo ${var~~[a-zA-Z]}
AbCdEfGhIjKlMnOpQrStUvWxYz
#此时匹配小写字母和大写的Z
[jgou@localhost ~]$ echo ${var~~[a-zZ]}
ABCDEFGHIJKLMNOPQRSTUVWXYz
#此时匹配大写字母和小写的a
[jgou@localhost ~]$ echo ${var~~[aA-Z]}
Abcdefghijklmnopqrstuvwxyz
```

- **<font color='red'>${parameter@operator}</font>**

Bash4.4中新增
这个扩展要么是parameter值的转换，要么是parameter本身信息的转换，依赖于operator的值。每个operator是一个单独的字母。

1.Q quote 的缩写，这个 operator 的功能是把 parameter 的值加上合适的引号，从而转换成在脚本中可重用的(reused)字符串形式：

```Shell
$ foo=1
$ echo ${foo@Q}
'1' # 原本 foo 的值只有 1 这一个字符，转换后的值有三个字符 “'1'”
$ echo ${IFS@Q}
' \t\n' # 因为 IFS 中有不可打印字符，所以转换后的值会自动使用 ANSI 转义形式的引号 $'...'，并且里面的字符也会使用反斜杠转义的形式
```

2.E escape 的缩写，这个 operator 的功能是把 parameter 的值中包含的转义序列解义(unescape)，就仿佛是把 parameter 的值放在了 $'...' 中间一样：

```Shell
$ foo='\u4e00'
$ echo $foo
\u4e00 # foo 的值包含 6 个 字符，刚好是一个转义序列
$ echo ${foo@E}
一 # 识别并转换 foo 的值中的转义序列，就像是执行了 echo $'\u4e00' 一样
```

3.P prompt 的缩写，这个 operator 的功能是把 parameter 的值按照提示符变量(PS1...)的转义规则解义，就像 Bash 解义 PS1... 一样：

```Shell
$ foo=1
$ echo ${foo@A}
foo='1' # 最普通的赋值语句
$ readonly foo # 给 foo 加上 r 属性
$ echo ${foo@A}
declare -r foo='1' # declare 命令的形式
$ export foo # 给 foo 加上 x 属性
$ echo ${foo@A}
declare -rx foo='1' # 变成了两个属性 rx
```

4.a attribute 的缩写，这个 operator 的功能是获取 parameter 的所有属性：

```Shell
$ declare -irtu foo=1
$ echo ${foo@a}
irtu
```

若 parameter 是个带有 \[\*] 或者 \[@] 下标的数组，那么如果 operator 是 QEPa 中的一个，则返回的值是一个列表，列表中的值分别对应原数组中的每个元素；如果 operator 是 A，则返回一个用 declare 声明数组的形式的字符串：

```Shell
$ readonly foo=(1 "$IFS" bar)
$ echo ${foo[@]@Q}
'1' $' \t\n' 'bar'
$ echo  ${foo[@]@A}
declare -ar foo=([0]="1" [1]=$' \t\n' [2]="bar")
```

参考： GNU Bash Manual(https://www.gnu.org/software/bash/manual/bash.pdf) 3.5.3节
http://www.jianshu.com/p/c623ef6f2342
https://my.oschina.net/leejun2005/blog/368777
http://xstarcd.github.io/wiki/shell/ShellParameterExpansion.html
http://blog.csdn.net/jiankun_wang/article/details/4349013
http://www.cnblogs.com/ziyunfei/p/4918675.html

