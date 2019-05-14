.. _cookbook:

Python CookBook
====================

..

python2转Python3修改内容
------------------------------

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


Python CRC计算
-----------------

CRC-16/IBM x16 + x15 + x2 + 1

.. code::

    def crc_check(data):
        crc_temp = 0x0000
        for data_item in data:
            crc_temp ^= data_item
            for j in range(8):
                if crc_temp & 0x01:
                    crc_temp = (crc_temp >> 1) ^ 0xA001
                else:
                    crc_temp = crc_temp >> 1
        return crc_temp

参考：http://www.ip33.com/crc.html


Python向Excel中插入文件
----------------------------

1. 安装pywin32

.. code::

    pip install pywin32

2. 执行如下脚本

.. code::

    import os
    import win32com.client as win32

    xl = win32.gencache.EnsureDispatch('Excel.Application')
    xl.Visible = 1
    wb = xl.Workbooks.Open(os.path.join(os.getcwd(), 'TRT测试报告.xlsx'))
    xl.DisplayAlerts = False
    ws = wb.Worksheets('测试用例')
    dest_cell = ws.Range('A110')
    obj = ws.OLEObjects()
    obj.Add(ClassType=None,Filename=os.path.join(os.getcwd(),'log.html'),Link=False,DisplayAsIcon=True,IconIndex=0,IconLabel='log.html',Left=dest_cell.Left,Top=dest_cell.Top)
    ws.SaveAs(os.path.join(os.getcwd(), 'TRT测试报告.xlsx'))
    xl.Workbooks.Close()


Python十六进制和十进制浮点数转换
------------------------------------

python 2.7.15版本

.. code::

    import struct
    def float_to_hex(float_data):
        result = []
        hex_data = struct.pack('!f', float_data).encode('hex')
        print str(float_data), hex_data
        for i in range(0, len(hex_data), 2):
            result.append(int(hex_data[i:i+2],16))
        return result
    def hex_to_float(hex_data):
        '''
        hex_data: [0x3F,0xA0,0x00,0x00]
        '''
        hex_data_str = ''.join(['%02x'%data for data in hex_data])
        float_data = struct.unpack('!f', hex_data_str.decode('hex'))[0]
        print hex_data_str, str(float_data)
        return float_data

Python 3.6.8版本

.. code::

    def float_to_hex(float_data):
        result = []
        hex_data = struct.pack('!f', float_data).hex()
        logger.info('float:{},hex list:{}'.format(str(float_data), hex_data))
        for i in range(0, len(hex_data), 2):
            result.append(int(hex_data[i:i+2],16))
        return result
    def hex_to_float(hex_data):
        '''
        hex_data: [0x3F,0xA0,0x00,0x00]
        '''
        hex_data_str = ''.join(['%02x'%data for data in hex_data])
        float_data = struct.unpack('!f', bytes.fromhex(hex_data_str))[0]
        logger.info('hex string:{},float:{}'.format(hex_data_str, str(float_data)))
        return float_data

调用举例

.. code::

    float_to_hex(3.78)
    3.78 4071eb85
    [64, 113, 235, 133]
    hex_to_float([64, 113, 235, 133])
    4071eb85 3.77999997139
    3.7799999713897705
