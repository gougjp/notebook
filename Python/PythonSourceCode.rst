Python 源代码学习(3.7.5)
====================================

第一章: Python 总体架构
-----------------------------

**1. Python 总体架构**

|     下图是Python的总体架构：

|     右边是Python的运行环境, 包括对象/类型系统(Object/Type structures), 内存
| 分配器(Memory Allocator)和运行时状态信息(Current State of Python). 运行时状
| 态维护了解释器在执行字节码时不同的状态(比如正常状态和异常状态)之间切换的动
| 作. 内存分配器则全权负责Python中创建对象时, 对内存的申请工作, 实际上它就是
| Python运行时与C中malloc的一层接口。而对象/类型系统则包含了Python中存在的各种
| 内建对象, 比如整数, list和dict, 以及各种用户自定义的类型和对象.


.. image:: images/0-1.jpeg














































