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



