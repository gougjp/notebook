Python向Excel中插入文件
=================================

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