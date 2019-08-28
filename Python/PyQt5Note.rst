PyQt5 Note
===========

安装
-------

* 对于Python 2

.. code::

    pip install python-qt5


* 对于Python 3
    
.. code::

    pip install pysip
    pip install PyQt5
    pip install PyQt5-tools


实例代码
-----------

* 在QTextEdit中, 修改Enter和Return键是默认处理方式

因为Enter和Return键是默认处理的, 并且在QTextEdit中按下回车键, 主窗口event是不会捕获到这个事件的, 
所以无法在主窗口中用keyPressEvent来修改；我们可以在主窗口中重新定义eventFilter函数, 并对textEdit
调用installEventFilter来注册事件处理函数
    
.. code::

    class SuperMaster(QWidget):
        def __init__(self, parent=None):
            
            ...
            ...
            ...

            self.ui.textEdit_2.installEventFilter(self)

            ...
            ...
            ...

        def eventFilter(self, watched, event):
            if watched == self.ui.textEdit_2:
                if event.type() == QtCore.QEvent.KeyPress:
                    key_event = QtGui.QKeyEvent(event)
                    if key_event.key() in (QtCore.Qt.Key_Enter, QtCore.Qt.Key_Return):
                        print('Enter')
                        if key_event.modifiers() == QtCore.Qt.ControlModifier:
                            print('Ctrl + Enter')
                        elif key_event.modifiers() == QtCore.Qt.AltModifier:
                            print('Alt + Enter')
                        elif key_event.modifiers() == QtCore.Qt.ShiftModifier:
                            print('Shift + Enter')

            return QWidget.eventFilter(self, watched, event)

重新实现QTextEdit类中的keyPressEvent函数来忽略对应键的操作, 并用这个类来创建QTextEdit对象

.. code::

    from PyQt5 import QtCore, QtGui, QtWidgets

    class MyTextEdit(QtWidgets.QTextEdit):
        def __init__(self, parent):
            super(MyTextEdit, self).__init__(parent)
        
        def keyPressEvent(self, event):
            # 没有按回车，默认处理
            if event.key() not in (QtCore.Qt.Key_Return, QtCore.Qt.Key_Enter):
                QtWidgets.QTextEdit.keyPressEvent(self,event)
            else:
                # Alt + 回车和Shift + 回车默认处理; 只按回车、回车+Ctrl或者回车+其他的都不处理
                if event.modifiers() == QtCore.Qt.AltModifier or event.modifiers() == QtCore.Qt.ShiftModifier:
                    QtWidgets.QTextEdit.keyPressEvent(self,event)


* 获取QTextEdit中光标所在行的内容; 前提是不能修改前面回车键的默认操作, 否则会获取所有行的内容

.. code::

    self.ui.textEdit_2.textCursor().block().text()


* 获取QTextEdit中光标所在行的行号; 前提是不能修改前面回车键的默认操作, 否则一直为0

.. code::

    self.ui.textEdit_2.textCursor().blockNumber()


