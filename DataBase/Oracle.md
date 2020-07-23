# Oracle存储过程

## Oracle存储过程简介

　　存储过程是事先经过编译并存储在数据库中的一段SQL语句的集合，调用存储过程可以简化应用开发人员的很多工作，
减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。

优点：

- 允许模块化程序设计，就是说只需要创建一次过程，以后在程序中就可以调用该过程任意次。
- 允许更快执行，如果某操作需要执行大量SQL语句或重复执行，存储过程比SQL语句执行的要快。
- 减少网络流量，例如一个需要数百行的SQL代码的操作有一条执行语句完成，不需要在网络中发送数百行代码。
- 更好的安全机制，对于没有权限执行存储过程的用户，也可授权他们执行存储过程。  

## 创建存储过程的语法

```Sql
create [or replace] procedure 存储过程名（param1 in type，param2 out type）
as
   变量1 类型（值范围）;
   变量2 类型（值范围）;
begin
   select count(*) into 变量1 from 表A where列名=param1；
   if (判断条件) then
        select 列名 into 变量2 from 表A where列名=param1；
        dbms_output.Put_line('打印信息');
   elsif (判断条件) then
        dbms_output.Put_line('打印信息');
   else
        raise 异常名（NO_DATA_FOUND）;
   end if;
exception
   when others then
        rollback;
end;
```

参数的几种类型:

- in 是参数的默认模式，这种模式就是在程序运行的时候已经具有值，在程序体中值不会改变。
- out 模式定义的参数只能在过程体内部赋值，表示该参数可以将某个值传递回调用他的过程
- in out 表示该参数可以向该过程中传递值，也可以将某个值传出去

### 举例

不带参数的存储过程:

```Sql
CREATE OR REPLACE PROCEDURE MYDEMO02 
AS 
        name VARCHAR(10);
        age NUMBER(10);
BEGIN
        name := 'xiaoming';--:=则是对属性进行赋值
        age := 18;
        dbms_output.put_line ( 'name=' || name || ', age=' || age );--这条是输出语句
END;
--存储过程调用(下面只是调用存储过程语法)
BEGIN 
    MYDEMO02();
END;
```

带参数的存储过程:

```Sql
CREATE OR REPLACE procedure MYDEMO03(name in varchar,age in int)
AS
BEGIN
      dbms_output.put_line('name='||name||', age='||age);
END;

--存储过程调用
BEGIN 
    MYDEMO03('姜煜',18);
END;
```

出现异常的输出存储过程:

```Sql
CREATE OR REPLACE PROCEDURE MYDEMO04
AS
    age INT;
BEGIN
    age:=10/0;
    dbms_output.put_line(age);
EXCEPTION when others then   --处理异常
    dbms_output.put_line('error');
END;
--调用存储过程
BEGIN
     MYDEMO04;
END;
```

Oracle常见的三大异常分类

- 预定义异常：由PL/SQL定义的异常。由于它们已在standard包中预定义了，因此，这些预定义异常可以直接在程序中使用，而不必再定义部分声明。
- 非预定义异常：用于处理预定义异常所不能处理的Oracle错误。
- 自定义异常：用户自定义的异常，需要在定义部分声明后才能在可执行部分使用。用户自定义异常对应的错误不一定是Oracle错误，例如它可能是一个数据错误。

获取当前时间和总人数:

```Sql
CREATE OR REPLACE PROCEDURE TEST_COUNT01
IS 
    v_total int;
    v_date varchar(20);
BEGIN
    select count(*) into v_total from EMP_TEST WHERE ENAME ='燕小六';  --into是赋值的关键字
    select to_char(sysdate,'yyyy-mm-dd')into v_date FROM EMP_TEST WHERE ENAME ='郭芙蓉';
    DBMS_OUTPUT.put_line('总人数:'||v_total);
    DBMS_OUTPUT.put_line('当前日期'||v_date);
END;

--调用存储过程
BEGIN
    TEST_COUNT01();
END;
```
