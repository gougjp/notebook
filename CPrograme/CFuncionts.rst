C Functions
====================
.. raw:: html

    <b><font color="red">char \*strrchr(const char \*str, int c)</font></b><br/>

在参数str所指向的字符串中搜索最后一次出现字符c(一个无符号字符)的位置; 该函数返回str中最后一次出现字符c的位置指针. 如果未找到该值, 则函数返回一个空指针.

``char *strstr(const char *haystack, const char *needle)``

在字符串haystack中查找第一次出现字符串needle的位置, 不包含终止符'\0'; 该函数返回在haystack中第一次出现needle字符串的位置指针, 如果未找到则返回null.

``char * __strcat_chk(char * dest, const char * src, size_t destlen)``

该接口与strcat()接口的工作方式相同, 只是\_\_strcat\_chk()接口在执行之前会先检查缓冲区是否溢出, 如果发现溢出, 该函数应终止, 调用该函数的程序将终止. destlen参数指定dest缓冲区的大小.

https://refspecs.linuxfoundation.org/LSB_5.0.0/LSB-Core-generic/LSB-Core-generic/baselib---strcat-chk-1.html

``char *strtok(char s[], const char *delim)``

当strtok()在参数s的字符串中发现参数delim中包含的分割字符时, 则会将该字符改为\0字符. 在第一次调用时, strtok()必需给予参数s字符串, 往后的调用则将参数s设置成NULL. 每次调用成功则返回指向被分割出片段的指针.

从s开头开始的一个个被分割的串. 当s中的字符查找到末尾时, 返回NULL. 如果查找不到delim中的字符时, 返回当前strtok的字符串的指针. 所有delim中包含的字符都会被滤掉, 并将被滤掉的地方设为一处分割的节点.

需要注意的是, 使用该函数进行字符串分割时, 会破坏被分解字符串的完整, 调用前和调用后的s已经不一样了. 第一次分割之后, 原字符串str是分割完成之后的第一个字符串, 剩余的字符串存储在一个静态变量中, 因此多线程同时访问该静态变量时, 则会出现错误.

strtok函数会破坏被分解字符串的完整, 调用前和调用后的s已经不一样了. 如果要保持原字符串的完整, 可以使用strchr和sscanf的组合等.

.. code-block:: c

    #include<string.h>
    #include<stdio.h>

    int main(void)
    {
        char input[16]="abc,d";
        char*p;

        /*strtok places a NULL terminator
        infront of the token,if found*/
        p=strtok(input,",");
        if(p)
            printf("%s\n",p);
            /*Asecond call to strtok using a NULL
            as the first parameter returns a pointer
            to the character following the token*/

        p=strtok(NULL,",");
        if(p)
            printf("%s\n",p);

        return 0;
    }

``size_t getpagesize(void)``

返回一分页的大小, 单位为字节(byte). 此为系统的分页大小, 不一定会和硬件分页大小相同. 在Intel x86上其返回值应为4096 bytes.

``int sscanf(const char *str, const char *format, ...)``

从字符串读取格式化输入, str是C字符串, 是函数检索数据的源; format是C字符串, 包含一个或多个格式说明符号; 其后的多个参数都为指针, 跟格式字符串中的每一个%标签一一对应, 包括类型和数量. 如果成功, 该函数返回成功匹配和赋值的个数. 如果到达文件末尾或发生读错误, 则返回EOF.




