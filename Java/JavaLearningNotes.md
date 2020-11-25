# Java Learning Notes

## 第一个Java程序

HelloDate.java, 主类名必须跟文件名相同, 这里都为HelloDate; 主函数名必须为main

```Java
import java.util.*;

public class HelloDate {
    public static void main(String[] args) {
        System.out.println("Hello, it's: ");
        System.out.println(new Date());
    }
}
```

- 编译

在命令行执行javac HelloDate.java, 则会编译生成一个HelloDate.class文件

- 执行

在命令行执行java HelloDate.class会出现如下错误

"**错误: 找不到或无法加载主类 HelloDate.class**"

配置如下环境变量, 然后删除class后缀则能成功执行: java HelloDate

```Shell
set JAVA_HOME="C:\Program Files (x86)\Java\jre1.8.0_251"
set CLASSPATH=".;%JAVA_HOME%\\lib:%JAVA_HOME%\\lib\\tools.jar"
```

## 基本语法

### 注释

多行注释: /\* ... \*/

单行注释: // ...

注释文档: /\*\* ... \*/

### 编码风格

类名大写字母开头, 后面驼峰风格
方法, 变量, 对象引用等名字以小写字母开头, 后面驼峰风格

### 操作符

- 单个字符用单引号, 字符串用双引号

- \>>> 无符号右移操作符, 它使用0扩展, 无论正负, 都在高位插入0; >>>= 为无符号右移和等号的组合, i >>>= 2即i = i >>> 2

- 字符串+, 如果+表达式中有字符串, 那么表达式中所有操作数都必须是字符串型, 其他类型编译器也会自动转换成它们对应的字符串形式

```Java
int x = 0, y = 1, z = 2;
String s = "x, y, z ";
print(s + x + y + z);   // x, y, z 012
print(x + " " + s);     // 0 x, y, z
print(s + (x + y + z)); // x, y, z 3
print("" + x);          // 0; 这里会调用函数Integer.toString()把整数0转成字符串0
```

- java 不会自动的将int数值转换成布尔值

- 类型转换: 在Java中, 执行窄化转换(将能容纳更多信息的数据类型转换成无法容纳那么多信息的类型), 编译器会强制要求进行显示类型转换; 扩展转换则不必显示的进行类型转换; 布尔类型不允许进行任何类型转换; "类"类型不允许进行类型转换

```Java
int i = 200;
long lng = (long)i;  // 可以显示转换
lng = i;             // 也可以不显示转换
long lng2 = (long)200;   // 可以显示转换
lng2 = 200;              // 也可以不显示转换
i = (int)lng2;           // 必须进行显示转换
```

浮点数转换成整数总是截断小数部分而不会进行四舍五入, 如果要进行四舍五入的转换, 可以使用函数Math.round(0.7), 结果为1

表达式中出现的最大数据类型决定了表达式最终结果的数据类型

- Java中没有sizeof()操作符

### 控制执行流程

- Java不允许将一个非布尔值, 比如数字作为布尔值使用

- 支持的控制语句: if-else, while, do-while, for, return, break, countinue, switch; 不支持goto

- Java里唯一用到逗号操作符的地方就是for循环的控制表达式

- foreach语法: 

```Java
for (float x : f)
    System.out.println(x)
```

这里f是一个浮点数数组, 在循环中float x定义了一个float临时变量, 然后循环把f中的每一个元素赋值给x进行循环




