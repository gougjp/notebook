Java Learning Notes
========================

..

第一个Java程序
-------------------

HelloDate.java, 主类名必须跟文件名相同, 这里都为HelloDate; 主函数名必须为main

.. code::

    import java.util.*;

    public class HelloDate {
        public static void main(String[] args) {
            System.out.println("Hello, it's: ");
            System.out.println(new Date());
        }
    }

* 编译

在命令行执行javac HelloDate.java, 则会编译生成一个HelloDate.class文件

* 执行

在命令行执行java HelloDate.class会出现如下错误

"**错误: 找不到或无法加载主类 HelloDate.class**"

删除class后缀则能成功执行: java HelloDate


基本语法
-------------------

* 注释

多行注释: /\* ... \*/
单行注释: // ...
注释文档: /\*\* ... \*/

* 编码风格

类名大写字母开头, 后面驼峰风格
方法, 变量, 对象引用等名字以小写字母开头, 后面驼峰风格


