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

- 支持的控制语句: if-else, while, do-while, for, return, break, continue, switch; 不支持goto

- Java里唯一用到逗号操作符的地方就是for循环的控制表达式

- foreach语法: 

```Java
for (float x : f)
    System.out.println(x)
```

这里f是一个浮点数数组, 在循环中float x定义了一个float临时变量, 然后循环把f中的每一个元素赋值给x进行循环

- break和continue后面可以跟一个标签, 标签就是后面跟有冒号的标识符(label1:), 标签起作用的唯一地方刚好是在迭代语句之前

```Java
label1:
outer-iteration {
    inner-iteration {
        //...
        break;   // break中断内部迭代, 回到外部迭代
        //...
        continue;   // 使执行点回到内部迭代的起始处
        //...
        continue label1;  // 同时中断内部迭代和外部迭代, 直接转到label1处; 随后, 它实际上是继续迭代过程, 但却从外部迭代开始
        //...
        break label1;   // 中断所有迭代, 回到label1处, 但并不重新进入迭代, 实际上是完全终止了两个迭代
    }
}
```

在Java里需要使用标签的唯一理由就是因为有循环嵌套存在, 而且想从多层嵌套中break或continue

- switch的选择因子必须是int或char类型; 如果将一个字符串或者浮点数作为选择因子, 那么它在switch语句里是不会工作的; 对于非整数类型, 则必须使用一系列if语句; 另外每个case后面的语句最好跟一个break, 否则后面的case语句还可能被执行到

### 初始化和清理

- 定义类的时候定义一个跟类名相同的方法, 则为构造器, Java在创建对象时会自动调用这个构造器进行初始化

- 不带任何参数的构造器叫做默认构造器(无参构造器), 构造器也可定义参数, 这样在创建对象时需要传入对应的参数; 构造器没有任何返回值

- 方法重载, 方法名字相同, 参数个数或者类型不相同的方法; 构造器方法也可以重载, 在实例化类的时候可以提供不同的参数来调用不同的构造器

- 在调用重载方法时, 如果实际参数小于方法中声明的形式参数类型, 实际数据类型就会被提升, 对于char类型会直接提升至int类型; 如果传入的实际参数大于重载方法声明的形式参数, 则需要显示的类型转换, 否则会报错 

- 如果没有定义构造器, 则实例化对象时编译器会自动创建一个默认构造器并调用; 如果定义了构造器, 则编译器就不会自动创建, 比如定义了一个带参数的构造器, 如果我们在实例化对象时没有传参数, 则系统会报错

- this关键字是对当前对象的引用, 跟Python中的self一样

- 对于方法的局部变量, 如果没有初始化则编译时会报错; 要是类的数据成员(即字段)是基本数据类型, 编译器会自动给定一个初始值, 而类的数据成员是一个对象的引用时, 如果不将其初始化, 此引用就会获得一个特殊值null

- 在类的内部, 变量定义的的先后顺序决定初始化的顺序; 即使变量定义散步于方法定义之间, 它们仍旧会在任何方法(包括构造器)被调用之前得到初始化

- 无论创建多少个对象, 静态数据都只占用一份存储区域; static关键字不能应用于局部变量; 静态对象只是在类首次加载的时候才会被初始化

- Java允许将多个静态初始化动作组织成一个特殊的"静态子句"("静态块")

```Java
class Cup {
    Cup(int marker) {
        print("Cup(" + marker + ")");
    }
    
    void f(int marker) {
        print("f(" + marker + ")");
    }
}

class Cups {
    static Cup cup1;
    static Cup cup2;
    static {
        cup1 = new Cup(1);
        cup2 = new Cup(1);
        
    }
    Cups() {
        print("Cups()")
    }
}
```

- 定义数组有以下两种方法:

```Java
int[] a1;
int a1[];
```

- 数组初始化

```Java
int[] a1 = {1, 2, 3, 4, 5};  //数组初始化方式1
int[] a2;     //现在只是一个数组的引用, 而且也没有给数组对象本身分配任何空间, 并且这里定义的时候不能指定数组的大小
a2 = a1;     //a2和a1都是数组的一个引用, 指向同一个数组, 当对a2进行修改, 则a1也会发生变化

int[] a1 = new int[10];   //定义一个数组a1, 有10个元素, 每个元素初始化为0(一般对于数字和字符, 初始化为0, 对于布尔型, 初始化为false)
a1.length;      //返回数组的元素个数

Integer[] a = {
    new Integer(1),
    new Integer(2),
    3,
}
Integer[] b = new Integer[] {
    new Integer(1),
    new Integer(2),
    3,
}
```

- 用Object数组作为方法的参数可以实现可变参数

- 枚举类型enum, 枚举对象的

### 访问权限控制

- 权限从大到小依次为: public, protected, 包访问权限(没有关键字), private; 如果没有提供任何访问权限修饰词, 则为包访问权限

- 一个Java源代码文件, 通常被称为编译单元, 该文件的后缀名必须为.java, 文件中只能有一个public类, 这个类的名称必须跟文件名相同, 包括大小写

- package定义包名, 必须放在文件的第一行(除了前面有注释)





