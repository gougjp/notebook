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

或者在命令执行时指定参数"-cp ."或"-classpath ."

```Shell
java -cp . HelloDate
java -classpath . HelloDate
```

## 基本语法

### 注释

多行注释: /\* ... \*/

单行注释: // ...

注释文档: /\*\* ... \*/

### 声明常量

常量在程序运行中只赋值一次, 并且在整个运行过程中都不会改变; 在声明常量时, 通常使用大写字母的格式

**final 数据类型 常量名称\[=值\]**

### 类成员变量和局部变量

```Java
class var {
    int x = 45;           //定义实例变量
    static int y = 90;    //定义静态变量
}
```

类成员变量在整个类中都可以使用; 静态变量的有效范围可以跨类, 甚至可以达到整个应用程序内, 在定义的类中可以直接使用, 在其他类中可以通过"类名.静态变量"的方式使用

```Java
public class Val {

	static int times = 3;                                 //定义类成员变量times
	
	public static void main(String[] args) {
		int times = 4;                                    //定义局部变量times
		
		System.out.println("times: " + times);            //打印局部变量的值
		System.out.println("Val times: " + Val.times);    //打印类成员变量的值
	}
}
```

### 数组

- 一维数组

```Java
//方法一
int arr[];            //声明整数数组
arr = new int[3];     //为数组分配内存
arr[0] = 1;           //给数组赋值
arr[1] = 2;
arr[2] = 3;

//方法二
int arr[] = new int[3];   //声明数组并分配内存
arr[0] = 1;               //给数组赋值
arr[1] = 2;
arr[2] = 3;

//方法三
int arr[] = new int[]{1, 2, 3};   //定义并初始化
int arr[] = {1, 2, 3};            //定义并初始化
```

- 二维数组

```Java
//方法一
int arr[][];             //声明整数数组
arr = new int[2][3];     //为数组分配内存
arr[0][0] = 1;           //给数组赋值

//方法二
int arr[][] = new int[2][3];   //声明数组并分配内存
arr[0][0] = 1;                 //给数组赋值

//方法三
int arr[][] = new int[]{{1, 2, 3}, {4, 5, 6}};   //定义并初始化
int arr[][] = {{1, 2, 3}, {4, 5, 6}};            //定义并初始化
```

- Arrays类中的静态方法fill()可对数组中的元素进行替换

**Arrays.fill(int\[\] a, int value)**

```Java
import java.util.Arrays;

public class Swap {

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		int arr[] = new int[5];
		Arrays.fill(arr, 8);        //填充过后, 数组的所有元素都为8
		for (int i = 0; i < arr.length; i++) {
			System.out.println("arr[" + i + "] = " + arr[i]);
		}
	}

}
```

**Arrays.fill(int\[\] a, int fromIndex, int toIndex, int value)**

填充数组中指定索引范围中的元素, fromIndex表示起始索引(包括), toIndex表示结束索引(不包括), 如果fromIndex == toIndex, 则填充范围为空

```Java
import java.util.Arrays;

public class Displace {

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		int arr[] = new int[] {45, 12, 2, 10, 1};
		
		Arrays.fill(arr, 1, 3, 8);
		
		for (int i = 0; i < arr.length; i++) {
			System.out.println("arr[" + i + "] = " + arr[i]);
		}
	}

}
```

- 对数组进行排序

**Arrays.sort(object)**

- 复制数组

**Arrays.copyOff(arr, int newlength)**

如果newlength大于arr的长度, 则新数组的后面用0填充; 如果小于则截断数组后半部分

**Arrays.copyOfRange(arr, int fromIndex, int toIndex)**

fromIndex: 起始索引, 必须为0到数组长度之间, 新数组包括该索引的元素
toIndex: 结束索引, 可以大于数组的长度, 新数组不包括该索引的元素

### 字符串

- 创建字符串

```Java
//方法一
String s = new String();

//方法二
car a[] = {'g', 'o', 'o', 'd'}
String s = new String(a);             //这种方式等价于String s = new String("good")

//方法三
car a[] = {'s', 't', 'u', 'd', 'e', 'n', 't'}
String s = new String(a, 2, 4);       //第二个参数为offset, 第三个参数为length; 这种方式等价于String s = new String("uden")
```

- 字符串连接都是用"+", 包括字符串和字符串连接, 字符串和其他数据类型连接

- 获取字符串的信息

**str.length():** 获取字符串的长度

**str.indexOf(substr):** 获取字符串中指定字符的索引, 没有检索到则返回-1

**str.lastIndexOf(substr):** 获取字符串中指定字符最后出现的索引, 没有检索到则返回-1

**str.charAt(int index):** 获取指定索引的字符

**str.trim():** 去掉字符串前后的空格

**StringTokenizer(String str, String delim):** 分割字符串, 不支持正则表达式

**str.replaceAll(String regex, String replacement):** 字符串替换, 默认为正则表达式匹配, 将regex替换为replacement, 替换所有匹配到的字符串

**str.replace(CharSequence target, CharSequence replacement):** 字符串替换, 全部替换, 不支持正则表达式匹配

**str.replaceFirst(String regex, String replacement):** 只替换第一个匹配的字符串, 默认为正则表达式匹配

```Java
public class ReplaceString {

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		String str1 = "Aoc.Iop.Aoc.Iop.Aoc";
		String str2 = "Aoc.Iop.Aoc.Iop.Aoc";
		String str3 = "Aoc.Iop.Aoc.Iop.Aoc";
		
		String str11 = str1.replace(".", "#");		// str11 = "Aoc#Iop#Aoc#Iop#Aoc"
		String str22 = str2.replaceAll(".", "#");	// str22 = "###################"
		String str33 = str3.replaceFirst(".", "#");	// str33 = "#oc.Iop.Aoc.Iop.Aoc"

		System.out.println(str11);
		System.out.println(str22);
		System.out.println(str33);
	}

}
```

- 字符串判断

**str.equals(String otherstr):** 比较字符串str和otherstr的是否相等, 长度是否一样; 且严格区分大小写; 返回true或false

**str.equalsIgnoreCase(String otherstr):** 比较字符串str和otherstr的是否相等, 长度是否一样; 不区分大小写; 返回true或false

**str.startsWith(String prefix):** 判断字符串是否以prefix开始

**str.endsWith(String suffix):** 判断字符串是否以suffix开始

**str.toLowerCase():** 将字符串转换成小写

**str.toUpperCase():** 将字符串转换成大写

**str.split(String sign):** 分割字符串, sign可以是正则表达式

**str.split(String sign, int limit):** 分割字符串, sign可以是正则表达式, limit指定拆分的份数

- 格式化字符串

**str.format(String format, Object...args):** 包括日期, 时间, 常规类型格式化

- 正则表达式

**str.matches(String regex):** 判断str是否匹配正则表达式regex, 返回true或false

- 字符串生成器StringBuilder

新创建的StringBuilder对象初始容量是16个字符, 可以自行指定长度, 也可以动态的执行添加, 删除和插入等操作, 大大的提高了频繁增加字符串的效率; 如果附加的字符超过可容纳的长度, 则StringBuilder对象将自动增加长度以容纳被附加的字符

```Java
public class Jerque {

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		String str = "";
		long startTime = System.currentTimeMillis();
		for (long i = 0; i < 100000; i++) {
			str = str + i;
		}
		long endTime = System.currentTimeMillis();
		long time = endTime - startTime;
		System.out.println("Used time: " + time);         //Used time: 5851
		StringBuilder builder = new StringBuilder("");
		startTime = System.currentTimeMillis();
		for (long j = 0; j < 100000; j++) {
			builder.append(j);
		}
		endTime = System.currentTimeMillis();
		time = endTime - startTime;
		System.out.println("StringBuilder used time:" + time);    //StringBuilder used time:2
	}

}
```

**StringBuilder append(String str):** 将字符串str追加到字符串生成器中

**StringBuilder append(StringBuffer sb):** 将字符串缓存sb的值追加到字符串生成器中

**StringBuilder insert(int offset, String str):** 将指定的字符串str添加到offset指定的位置

**StringBuilder delete(int start, int end):** 移除字符串生成器中的子字符串, start和end分别为起始位置和结束位置, 返回删除后的字符串; start位置的字符会被删除, end位置的字符不会被删除; 如果start等于end, 则不会删除任何字符

**StringBuilder toString():** 将字符串生成器转换成字符串, 转换后字符串生成器的值不变

### 编码风格

类名大写字母开头, 后面驼峰风格
方法, 变量, 对象引用等名字以小写字母开头, 后面驼峰风格

### 操作符

- 单个字符用单引号, 字符串用双引号

- 逻辑运算符的操作元必须为布尔类型值, 返回值也为布尔类型值; &&和&都是逻辑与, \|\|和\|都是逻辑或, 只是&&和\|\|是短路运算符, 而&和\|是非短路运算符; 同时, &和\|也是位运算符, 当两边的操作元为布尔类型时则为逻辑运算符, 否则为位运算符

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

- 复合语句就是一个语句块, 里面定义的变量只在语句块中可见

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

### Java交换两个变量的值

```Java
import java.util.Scanner;

public class VariableExchange {
	
	public static void main(String[] args) {
		Scanner scan = new Scanner(System.in);
		
		System.out.println("Input the value of A");
		long A = scan.nextLong();
		
		System.out.println("Input the value of B");
		long B = scan.nextLong();
		
		System.out.println("A = " + A);
		System.out.println("B = " + B);
		
		A = A ^ B;
		B = B ^ A;
		A = A ^ B;
		
		System.out.println("A = " + A);
		System.out.println("B = " + B);
	}
}
```



