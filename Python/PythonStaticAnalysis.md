# Python Static Analysis

### **PyLint**

1. 安装

	```Shell
	pip install pylint
	```

2. Checker

	https://pylint.readthedocs.io/en/latest/technical_reference/features.html

3. 文档

	https://pylint.readthedocs.io/en/latest/faq.html


### **pep8/pycodestyle**

1. 安装

	```Shell
	pip install pep8/pycodestyle
	```

2. 文档

	https://pycodestyle.pycqa.org/en/latest/
	https://www.osgeo.cn/pycodestyle/

3. Error codes

	https://pycodestyle.pycqa.org/en/latest/intro.html?highlight=E501#error-codes

### **flake8**

1. 安装

	```Shell
	pip install flake8
	```

2. 文档

	https://flake8.pycqa.org/en/latest/faq.html

3. Error / Violation Codes

	https://flake8.pycqa.org/en/latest/user/error-codes.html
    
### PEP8 代码规范常见问题及解决方法

1. **PEP 8: no newline at end of file**

    解决方法：代码末尾需要另起一行，光标移到最后回车即可
    
2. **PEP 8: indentation is not a multiple of four**

    解决方法：缩进不是4的倍数，检查缩进

3. **PEP 8: over-indented**

    解决方法：过度缩进，检查缩进
    
4. **PEP 8: missing whitespace after ','**

    解决方法：逗号后面少了空格，添加空格即可，类似还有分号或者冒号后面少了空格
    
5. **PEP 8: multiple imports on one line**

    解决方法：不要在一句 import 中引用多个库，举例：import socket, urllib.error最好写成：import socket import urllib.error
    
6. **PEP 8: blank line at end of line**

    解决方法：代码末尾行多了空格，删除空格即可

7. **PEP 8: at least two spaces before inline comment**

    解决方法：代码与注释之间至少要有两个空格

8. **PEP 8: block comment should start with '#'**

    解决方法：注释要以#加一个空格开始

9. **PEP 8: inline comment should start with '#'**

    解决方法：注释要以#加一个空格开始

10. **PEP 8: module level import not at top of file**

    解决方法：import不在文件的最上面，可能之前还有其它代码

11. **PEP 8: expected 2 blank lines，found 0**

    解决方法：需要两条空白行，添加两个空白行即可

12. **PEP 8: function name should be lowercase**

    解决方法：函数名改成小写即可

13. **PEP 8: missing whitespace around operator**

    解决方法：操作符('='、'>'、'<'等)前后缺少空格，加上即可

14. **PEP 8: unexpected spaces around keyword / parameter equals**

    解决方法：关键字/参数等号周围出现意外空格，去掉空格即可

15. **PEP 8: multiple statements on one line (colon)**

    解决方法：多行语句写到一行了，比如：if x == 2: print('OK')要分成两行写

16. **PEP 8: line too long (82 > 79 characters)**

    解决方法：超过了每行的最大长度限制79

17. **PEP 8: Simplify chained comparison**

    可简化连锁比较（例如：if a >= 0 and a <= 9: 可以简写为：if 0 <= a <= 9:）






