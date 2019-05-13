python2转Python3修改内容
=================================

1. struct.pack

.. code::

    struct.pack('!f', float_data).encode('hex')
    改为
    struct.pack('!f', float_data).hex()
    
2. struct.unpack

.. code::

    struct.unpack('!f', hex_data_str.decode('hex'))
    改为
    struct.unpack('!f', bytes.fromhex(hex_data_str))
    
3. str转bytes

.. code::

    字符串指针作为动态库函数的参数需要转换为bytes类型
    bytes(sn, 'ascii')
    bytes转字符串，str_output为b'abcdef'
    str_output.decode('utf-8')

    python3有两种表示字符序列的类型：bytes和str。前者的实例包含原始的8位值；后者的实例包含Unicode字符。
    python2中也有两种表示字符序列的类型，分别叫做str和unicode。与python3不同的是，str的实例包含原始的8位值，而unicode的实例，则包含Unicode字符。

4. 文件打开

.. code::

    打开文件不支持file(firmwareName)函数，需要改成open
    open(firmwareName)

5. 除法运算

.. code::

    在Python2中，除法的取值结果取整数
    在Python3中，除法/的结果包含小数，如果只想取整数需要使用//

