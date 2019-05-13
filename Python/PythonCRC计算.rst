Python CRC计算
================

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
