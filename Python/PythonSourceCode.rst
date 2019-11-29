Python 源代码学习(3.7.5)
====================================

第一章: Python 总体架构
-----------------------------

**1. Python 总体架构**

| 下图是Python的总体架构：

| 右边是Python的运行环境, 包括对象/类型系统(Object/Type structures), 内存分配
| 器(Memory Allocator)和运行时状态信息(Current State of Python). 运行时状态维
| 护了解释器在执行字节码时不同的状态(比如正常状态和异常状态)之间切换的动作. 
| 内存分配器则全权负责Python中创建对象时, 对内存的申请工作, 实际上它就是
| Python运行时与C中malloc的一层接口。而对象/类型系统则包含了Python中存在
| 的各种内建对象, 比如整数, list和dict, 以及各种用户自定义的类型和对象.

| 中间部分是Python的核心 -- 解释器(interpreter), 或者称为虚拟机. 在解析器中, 箭
| 头的方向指示了Python运行过程中的数据流方向. 其中Scanner对应词法分析, 将文
| 件输入的Python源代码或从命令行输入的一行行Python代码切分为一个个的token;
| Parser对应语法分析, 在Scanner的分析结果上进行语法分析, 建立抽象语法树(AST); 
| Compiler是根据建立的AST生成指令集合 -- Python字节码(byte code), 就像Java编译
| 器和C#编译器所做的那样; 最后由Code Evaluator来执行这些字节码. 因此Code Evaluator
| 又可以被称为虚拟机

| 图中, 在解释器


.. image:: images/0-1.jpeg

**2. Python源代码的组织**












































