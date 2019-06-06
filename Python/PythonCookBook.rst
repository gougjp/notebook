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


python 各进制转换
------------------------------------

* 10进制转为2进制

.. code::

    bin(10)   # 0b1010
    
* 2进制转为10进制

.. code::
    
    int('1001',2)  # 9
    int('0b1101', 2) # 13
    
* 10进制转为16进制

.. code::

    hex(10)  # 0xa
    
* 16进制到10进制

.. code::

    int('ff', 16)  # 255
    int('0xab', 16)   # 171
    
* 10进制转为8进制

.. code::

    print("%o" % 10)  # 12
    oct(8)  # 10

* 16进制到2进制

.. code::

    bin(0xa)  # 0b1010
    
* 2进制到16进制

.. code::

    hex(0b1001)  # 0x9


python 捕获键盘和鼠标输入
------------------------------------
    
https://github.com/ethanhs/pyhooked/blob/master/pyhooked/__init__.py

.. code::
   
    """
    This file is part of pyhooked, an LGPL licensed pure Python hotkey module for Windows
    Copyright (C) 2016 Ethan Smith
    """
    import ctypes
    from ctypes import wintypes
    from ctypes import CFUNCTYPE, POINTER, c_int, c_uint, c_void_p, windll
    from ctypes import byref
    from warnings import warn
    from traceback import format_exc
    import atexit

    __version__ = '0.8.1'

    cmp_func = CFUNCTYPE(c_int, c_int, wintypes.HINSTANCE, POINTER(c_void_p))

    # redefine names to avoid needless clutter
    GetModuleHandleA = ctypes.windll.kernel32.GetModuleHandleA
    SetWindowsHookExA = ctypes.windll.user32.SetWindowsHookExA
    GetMessageW = ctypes.windll.user32.GetMessageW
    DispatchMessageW = ctypes.windll.user32.DispatchMessageW
    TranslateMessage = ctypes.windll.user32.TranslateMessage
    CallNextHookEx = ctypes.windll.user32.CallNextHookEx
    UnhookWindowsHookEx = ctypes.windll.user32.UnhookWindowsHookEx

    # specify the argument and return types of functions
    GetModuleHandleA.restype = wintypes.HMODULE
    GetModuleHandleA.argtypes = [wintypes.LPCWSTR]
    SetWindowsHookExA.restype = c_int
    SetWindowsHookExA.argtypes = [c_int, cmp_func, wintypes.HINSTANCE, wintypes.DWORD]
    GetMessageW.argtypes = [POINTER(wintypes.MSG), wintypes.HWND, c_uint, c_uint]
    TranslateMessage.argtypes = [POINTER(wintypes.MSG)]
    DispatchMessageW.argtypes = [POINTER(wintypes.MSG)]


    def _callback_pointer(handler):
        """Create and return C-pointer"""
        return cmp_func(handler)


    class BaseEvent(object):
        """A keyboard or mouse event."""
        pass

    class KeyboardEvent(BaseEvent):
        """Class to describe an event triggered by the keyboard"""

        def __init__(self, current_key=None, event_type=None, pressed_key=None, key_code=None):
            self.current_key = current_key
            self.event_type = event_type
            self.pressed_key = pressed_key
            self.key_code = key_code


    class MouseEvent(BaseEvent):
        """Class to describe an event triggered by the mouse"""

        def __init__(self, current_key=None, event_type=None, mouse_x=None, mouse_y=None):
            self.current_key = current_key
            self.event_type = event_type
            self.mouse_x = mouse_x
            self.mouse_y = mouse_y


    # The following section contains dictionaries that map key codes and other event codes to the event type (e.g. key up)
    # and the key or button doing the action (e.g. Tab)
    MOUSE_ID_TO_KEY = {512: 'Move',
                       513: 'LButton',
                       514: 'LButton',
                       516: 'RButton',
                       517: 'RButton',
                       519: 'WheelButton',
                       520: 'WheelButton',
                       522: 'Wheel'}

    MOUSE_ID_TO_EVENT_TYPE = {512: None,
                              513: 'key down',
                              514: 'key up',
                              516: 'key down',
                              517: 'key up',
                              519: 'key down',
                              520: 'key up',
                              522: None}

    # stores the relation between keyboard event codes and the key pressed. Reference:
    # https://msdn.microsoft.com/en-us/library/windows/desktop/dd375731(v=vs.85).aspx
    # seems to only work on 32 bits
    ID_TO_KEY = {8: 'Back',
                 9: 'Tab',
                 13: 'Return',
                 20: 'Capital',
                 27: 'Escape',
                 32: 'Space',
                 33: 'Prior',
                 34: 'Next',
                 35: 'End',
                 36: 'Home',
                 37: 'Left',
                 38: 'Up',
                 39: 'Right',
                 40: 'Down',
                 44: 'PrtScr',
                 46: 'Delete',
                 48: '0',
                 49: '1',
                 50: '2',
                 51: '3',
                 52: '4',
                 53: '5',
                 54: '6',
                 55: '7',
                 56: '8',
                 57: '9',
                 65: 'A',
                 66: 'B',
                 67: 'C',
                 68: 'D',
                 69: 'E',
                 70: 'F',
                 71: 'G',
                 72: 'H',
                 73: 'I',
                 74: 'J',
                 75: 'K',
                 76: 'L',
                 77: 'M',
                 78: 'N',
                 79: 'O',
                 80: 'P',
                 81: 'Q',
                 82: 'R',
                 83: 'S',
                 84: 'T',
                 85: 'U',
                 86: 'V',
                 87: 'W',
                 88: 'X',
                 89: 'Y',
                 90: 'Z',
                 91: 'Lwin',
                 92: 'Rwin',
                 93: 'App',
                 95: 'Sleep',
                 96: 'Numpad0',
                 97: 'Numpad1',
                 98: 'Numpad2',
                 99: 'Numpad3',
                 100: 'Numpad4',
                 101: 'Numpad5',
                 102: 'Numpad6',
                 103: 'Numpad7',
                 104: 'Numpad8',
                 105: 'Numpad9',
                 106: 'Multiply',
                 107: 'Add',
                 109: 'Subtract',
                 110: 'Decimal',
                 111: 'Divide',
                 112: 'F1',
                 113: 'F2',
                 114: 'F3',
                 115: 'F4',
                 116: 'F5',
                 117: 'F6',
                 118: 'F7',
                 119: 'F8',
                 120: 'F9',
                 121: 'F10',
                 122: 'F11',
                 123: 'F12',
                 144: 'Numlock',
                 160: 'Lshift',
                 161: 'Rshift',
                 162: 'Lcontrol',
                 163: 'Rcontrol',
                 164: 'Lmenu',
                 165: 'Rmenu',
                 186: 'Oem_1',
                 187: 'Oem_Plus',
                 188: 'Oem_Comma',
                 189: 'Oem_Minus',
                 190: 'Oem_Period',
                 191: 'Oem_2',
                 192: 'Oem_3',
                 219: 'Oem_4',
                 220: 'Oem_5',
                 221: 'Oem_6',
                 222: 'Oem_7',
                 #223: 'OEM_8',
                 1001: 'mouse left',  # mouse hotkeys
                 1002: 'mouse right',
                 1003: 'mouse middle',
                 1000: 'mouse move',  # single event hotkeys
                 1004: 'mouse wheel up',
                 1005: 'mouse wheel down',
                 1010: 'Ctrl',  # merged hotkeys
                 1011: 'Alt',
                 1012: 'Shift',
                 1013: 'Win',
                 }

    event_types = {0x100: 'key down',  # WM_KeyDown for normal keys
                   0x101: 'key up',  # WM_KeyUp for normal keys
                   0x104: 'key down',  # WM_SYSKEYDOWN, used for Alt key.
                   0x105: 'key up',  # WM_SYSKEYUP, used for Alt key.
                   }
    # these are used for specifying the hook type we want to make
    WH_KEYBOARD_LL = 0x00D
    WH_MOUSE_LL = 0x0E
    # the Windows quit message, if the program quits while listening.
    WM_QUIT = 0x0012


    class Hook(object):
        """Main hotkey class used to and listen for hotkeys. Set an event handler to check what keys are pressed."""

        def __init__(self, warn_unrecognised = False):
            """Initializer of the Hook class, creates class attributes. If warn_unrecognised is True, warn when an unrecognised key is pressed."""
            self.warn_unrecognised = warn_unrecognised
            self.pressed_keys = []
            self.keyboard_id = None
            self.mouse_id = None
            self.mouse_is_hook = False
            self.keyboard_is_hook = True

        def handler(self, event):
            """Handle keyboard and mouse events."""
            raise NotImplementedError()
        
        def stop(self):
            """Stop this object from listening."""
            windll.user32.PostQuitMessage (0)
        
        def hook(self, keyboard=True, mouse=False):
            """Hook mouse and/or keyboard events"""
            self.mouse_is_hook = mouse
            self.keyboard_is_hook = keyboard

            # check that we are going to hook into at least one device
            if not self.mouse_is_hook and not self.keyboard_is_hook:
                raise Exception("You must hook into either the keyboard and/or mouse events")

            if self.keyboard_is_hook:
                def keyboard_low_level_handler(code, event_code, kb_data_ptr):
                    """Used to catch keyboard events and deal with the event"""
                    key_code = 0xFFFFFFFF & kb_data_ptr[0]  # key code
                    current_key = ID_TO_KEY.get(key_code) # check the type of event (see ID_TO_KEY for a list)
                    if current_key is None:
                        event = None # We can check this later.
                        if self.warn_unrecognised:
                            warn('Unrecognised key ID %d.' % key_code)
                    else:
                        event_type = event_types[0xFFFFFFFF & event_code]

                        if event_type == 'key down':  # add key to those down to list
                            if current_key not in self.pressed_keys:
                                self.pressed_keys.append(current_key)

                        if event_type == 'key up':  # remove when no longer pressed
                            try:
                                self.pressed_keys.remove(current_key)
                            except ValueError:
                                pass # current_key is not in the list.

                        # wrap the keyboard information grabbed into a container class
                        event = KeyboardEvent(current_key, event_type, self.pressed_keys, key_code)

                    # Call the event handler to deal with keys in the list
                    try:
                        if event:
                            self.handler(event)
                    except Exception as e:
                        warn('While handling {}, self.handler produced a traceback:\n{}'.format(event, format_exc()))
                    finally:
                        # TODO: fix return here to use non-blocking call
                        return CallNextHookEx(self.keyboard_id, code, event_code, kb_data_ptr)

                keyboard_pointer = _callback_pointer(keyboard_low_level_handler)

                self.keyboard_id = SetWindowsHookExA(WH_KEYBOARD_LL, keyboard_pointer,
                                                     GetModuleHandleA(None),
                                                     0)

            if self.mouse_is_hook:
                def mouse_low_level_handler(code, event_code, kb_data_ptr):
                    """Used to catch and deal with mouse events"""
                    current_key = MOUSE_ID_TO_KEY.get(event_code)  # check the type of event (see MOUSE_ID_TO_KEY for a list)
                    if current_key is None:
                        event = None # We can check this later.
                        if self.warn_unrecognised:
                            warn('Unrecognised mouse ID %d.' % event_code)
                    else:
                        if current_key != 'Move':  # if we aren't moving, then we deal with a mouse click
                            event_type = MOUSE_ID_TO_EVENT_TYPE[event_code]
                            # the first two members of kb_data_ptr hold the mouse position, x and y
                            event = MouseEvent(current_key, event_type, kb_data_ptr[0], kb_data_ptr[1])

                    try:
                        if event:
                            self.handler(event)
                    except Exception as e:
                        warn('While handling {}, self.handler produced a traceback:\n{}'.format(event, format_exc()))
                    finally:
                       # TODO: fix return here to use non-blocking call
                        return CallNextHookEx(self.mouse_id, code, event_code, kb_data_ptr)

                mouse_pointer = _callback_pointer(mouse_low_level_handler)
                self.mouse_id = SetWindowsHookExA(WH_MOUSE_LL, mouse_pointer,
                                                  GetModuleHandleA(None), 0)

            atexit.register(UnhookWindowsHookEx, self.keyboard_id)
            atexit.register(UnhookWindowsHookEx, self.mouse_id)

            message = wintypes.MSG()
            while self.mouse_is_hook or self.keyboard_is_hook:
                msg = GetMessageW(byref(message), 0, 0, 0)
                if msg in [0, -1]:
                    self.unhook_keyboard()
                    self.unhook_mouse()
                    break # Exit the loop.

                else:
                    TranslateMessage(byref(message))
                    DispatchMessageW(byref(message))

        def unhook_mouse(self):
            """Stop listening to the mouse"""
            if self.mouse_is_hook:
                self.mouse_is_hook = False
                UnhookWindowsHookEx(self.mouse_id)

        def unhook_keyboard(self):
            """Stop listening to the keyboard"""
            if self.keyboard_is_hook:
                self.keyboard_is_hook = False
                UnhookWindowsHookEx(self.keyboard_id)

Pyinstaller 将Python脚本打包成 exe
------------------------------------

1. 官方手册：

https://pyinstaller.readthedocs.io/en/v3.3.1/

2. 安装

.. code::

    pip install PyInstaller
    
3. 打包脚本

.. code::

    pyinstaller -F EEDModule.py
    
4. pyinstaller命令的参数介绍

    -h                    显示帮助
    -v                    显示版本号
    -p                    指定额外的import路径, 类似于使用PYTHONPATH, 参见PYTHONPATH
    -F, --onefile         生成结果是一个exe文件, 所有的第三方依赖、资源和代码均被打包进该exe内
    -D, --onedir          生成结果是一个目录, 各种第三方依赖、资源和exe同时存储在该目录
    --add-data            打包额外资源, 用法：pyinstaller main.py --add-data=src;dest。windows以;分割，linux以:分割
    --add-binary          打包额外的代码, 用法：同--add-data。与--add-data不同的是，用binary添加的文件，pyi会分析它引用的文件并把它们一同添加进来
    --runtime-hook        指定用户--runtime-hook, 如果设置了此参数, 则--runtime-hook会在运行main.py之前被运行
    --runtime-tmpdir      指定运行时的临时目录, 默认使用系统临时目录
    --hidden-import       打包额外py库, pyi在分析过程中, 有些import没有正确分析出来, 运行时会报import error, 这时可以使用该参数
    --clean               在本次编译开始时, 清空上一次编译生成的各种文件, 默认不清除
    
5. 参考

https://blog.csdn.net/weixin_39000819/article/details/80942423
https://pyinstaller.readthedocs.io/en/v3.3.1/usage.html
https://blog.csdn.net/bearstarx/article/details/81054134