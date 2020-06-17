# shell 数组

## 普通数组

用整数作为数组索引的数组, 只支持一维数组, 下标由0开始, 且必须大于等于0.

- 数组的定义: 用括号来表示数组, 数组元素用"空格"符号分割开

```Shell
array_name=(value0 value1 value2 value3)
#或者
array_name=(
    value0
    value1
    value2
    value3
)
#或者单独定义数组的各个分量
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2

#举例:
arr_number=(1 2 3 4 5)  #数字元素数组
arr_string=("abc" "edf" "sss")  #字符串元素数组
arr_string=('abc' 'edf' 'sss')  #字符串元素数组也可以用单引号
```

- 读取数组元素:${array_name\[index]}

```Shell
valuen=${array_name[2]} #读取数组的第三个元素

echo ${array_var[0]}  #读取数组的第一个元素

index=5
echo ${array_var[$index]} #读取数组的第6个元素
```

- 以清单形式打印出数组中的所有值：${array_var\[\*]} 或 ${array_var\[@]}

```Shell
echo ${array_var[*]}
test1 test2 test3 test4 test5 test6

echo ${array_var[@]}
test1 test2 test3 test4 test5 test6
```

- 获取数组的长度, 即数组中元素的个数

```Shell
#获取数组元素的个数
length=${#array_name[@]}

#或者
length=${#array_name[*]}

#取得数组单个元素的长度
lengthn=${#array_name[n]}
```

- 删除数组中的某个元素, 以及删除整个数组

```Shell
#删除数组中第二个元素
unset array[1]

#删除整个数组
unset array
```

- 对数组某个下标进行赋值:

```Shell
#要赋值的数组下标元素已经存在, 则会修改该元素
arr_number=(1 2 3 4 5)
arr_number[2]=100
echo ${arr_number[*]}
1 2 100 4 5

#要赋值的数组元素不存在,则会增加一个该下标的元素, 这里原来的下标最大为4, 加入下标13过后, 数组的长度变为6, 但是下标还是分别为0,1,2,3,4,13; 此时要取数组的最后一个值必须要用${arr_number[13]}
arr_number[13]=13
echo ${arr_number[*]}
1 2 100 4 5 13
#如果此时再对下标8进行赋值, 则会在13之前插入一个8, 会根据下标的顺序插入值, 而下标分别为0,1,2,3,4,8,13; 数组长度为7
arr_number[8]=8
echo ${arr_number[*]}
1 2 100 4 5 8 13
```

- 分片访问:${array\[@]:offset:length}或者${array\[\*]:offset:length}

```Shell
#offset可以为负数, 此时负号与冒号之间必须要有空格, length必须大于等于0
array=(0 1 2 3 4 5 6 7 8 9)
echo ${array[@]:4:2}
4 5
echo ${array[@]: -4:2}
6 7
echo ${array[@]:4:-2}
-bash: -2: substring expression < 0
```

- 模式替换:${array\[@]/old/new}或者{array\[\*]/old/new}

```Shell
array=(0 1 2 3 4 5 6 7 8 9)
echo ${array[@]/3/-3}
0 1 2 -3 4 5 6 7 8 9
echo ${array[@]}  #原数组的值不变
0 1 2 3 4 5 6 7 8 9
```

- 数组的遍历

```Shell
for v in ${arr_number[@]}; do
    echo $v;
done
```

- 获取数组所有的索引: ${!array\[\*]} 或者 ${!array\[@]}

```Shell
array=(0 1 2 3 4 5 6 7 8 9)
array[20]=123
echo ${array[@]}
0 1 2 3 4 5 6 7 8 9 123
echo ${!array[*]}
0 1 2 3 4 5 6 7 8 9 20
```

## 关联数组

以字符串作为索引, 其中键是唯一的，值可以不唯一

- 关联数组的定义: 在使用关联数组之前，需要使用命令 declare -A array 进行显示声明

```Shell
declare -A ass_array
ass_array=([index1]=val1 [index2]=val2)

#声明之后，可以用两种方法将元素添加到关联数组中
#1. 利用内嵌"索引-值"列表的方法，提供一个"索引-值"列表
ass_array=([index1]=val1 [index2]=val2)
使用独立的“索引-值”进行赋值：
ass_array[index1]=val1
ass_array[index2]=val2

declare -A fruits_value
fruits_value=([apple]='100dollars' [orange]='150 dollars')
```

- 获取关联数组的元素:

```Shell
${ass_array[key]}

#如:
echo ${fruits_value[apple]}
```

- 关联数组的其他操作

    | 语法 | 描述 |              
    | - | - | 
    | ${!array\[\*]}  | 取关联数组所有键 |        
    | ${!array\[@]}   | 取关联数组所有键 |        
    | ${array\[\*]}   | 取关联数组所有值 |        
    | ${array\[@]}    | 取关联数组所有值 |        
    | ${#array\[\*]}  | 取关联数组的长度 |        
    | ${#array\[@]}   | 取关联数组的长度 |        


