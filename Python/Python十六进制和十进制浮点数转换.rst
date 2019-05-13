Python十六进制和十进制浮点数转换
===================================

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


