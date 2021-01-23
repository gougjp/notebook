# Superxon Note

### 数据回传工具打包exe方法


- 安装对应的包

```Shell
# 这两个包为打包工具
pip install pyinstaller
pip install pywin32
# 这个包为Python文件中引用的
pip install cx-Oracle
```
    
- 进入到对应的目录,执行如下命令

```Shell
pyinstaller -F main.py
```

- 如果出现以下错误，则可能是中文路径导致的

```Shell
Traceback (most recent call last):
  File "c:\python27\lib\runpy.py", line 174, in _run_module_as_main
    "__main__", fname, loader, pkg_name)
  File "c:\python27\lib\runpy.py", line 72, in _run_code
    exec code in run_globals
  File "C:\Python27\Scripts\pyinstaller.exe\__main__.py", line 5, in <module>
  File "c:\python27\lib\site-packages\PyInstaller\__init__.py", line 72, in <module>
    DEFAULT_SPECPATH = compat.getcwd()
  File "c:\python27\lib\site-packages\PyInstaller\compat.py", line 613, in getcwd
    cwd = win32api.GetShortPathName(cwd)
AttributeError: 'module' object has no attribute 'GetShortPathName'
```

### Jenkins 服务器维护

- 重启服务：

    配置文件：
    /etc/init.d/jenkins
    /etc/default/jenkins

```Shell
service jenkins status    #查看Jenkins服务的状态
service jenkins restart   #重启Jenkins服务
service jenkins stop      #停止Jenkins服务
service jenkins start     #启动Jenkins服务
```
    
- 版本更新：

    将下载的新版jenkins.war放到/usr/share/jenkins/目录下，然后重启Jenkins服务

### 产线入库单打印设置

- Excel模板inwarehouse_templet.xlsx设置

    - 安装Adobe PDF虚拟打印机(Adobe Acrobat XI Pro软件)
    
    - 打开Excel文件, 点击文件 \-\> 打印 \-\> 选中打印机Adobe PDF \-\> 选择纸张大小 \-\> 其他纸张大小
    
    - 在弹出的页面设置对话框中, 在页面标签页中选择纸张大小为"A4两等份", 保存即可
    
- Python脚本自动打印

```Python
import os
import time
import logging
import datetime
import win32api
import win32print
import win32com.client


def deal_path(pathname=''):
    '''deal with windows file path'''
    if pathname:
        pathname = pathname.strip()

    if pathname:
        pathname = r'%s'%pathname
        pathname = pathname.replace(r'/', '\\')
        pathname = os.path.abspath(pathname)
        if pathname.find(":\\") == -1:
            pathname = os.path.join(os.getcwd(), pathname)

    return pathname

def close_excel_by_force(excel):
    import win32process
    import win32gui
    import win32con

    # Get the window's process id's
    hwnd = excel.Hwnd
    t, p = win32process.GetWindowThreadProcessId(hwnd)
    # Ask window nicely to close
    win32gui.PostMessage(hwnd, win32con.WM_CLOSE, 0, 0)
    # Allow some time for app to close
    # time.sleep(10)
    # If the application didn't close, force close
    try:
        handle = win32api.OpenProcess(win32con.PROCESS_TERMINATE, 0, p)
        if handle:
            win32api.TerminateProcess(handle, 0)
            win32api.CloseHandle(handle)
    except:
        pass


class RWExcel():
    def __init__(self):
        self.m_excel = win32com.client.DispatchEx('Excel.Application')
        self.m_excel.DisplayAlerts = False
        self.m_excel.Visible = False
        self.m_book = None

    def open_excel(self, filename):
        m_filename = deal_path(filename)
        if os.path.isfile(m_filename):
            self.m_book = self.m_excel.Workbooks.Open(m_filename)

    def save_excel(self, filename):
        m_newfile = deal_path(filename)
        self.m_book.SaveAs(m_newfile)

    def close_excel(self, savechange=True):
        self.m_book.Close(SaveChanges=savechange)
        self.m_excel.Quit()
        close_excel_by_force(self.m_excel)
        del self.m_excel

    def get_sheet_by_name(self, name):
        '''get the sheet object by the sheet name'''
        for i in range(1, self.m_book.Worksheets.Count+1):
            sheet = self.m_book.Worksheets(i)
            if name == sheet.Name:
                return sheet

        return None

    def set_cell_value(self, sheet, row, col, value):
        '''set value of one cell'''
        sheet.Cells(row, col).Value = value


class Reports():
    def write_and_print_inbox_report(self, boxnumber, counter, report_datas, printflag, pn):
        '''
        打印入盒报告, 每一个吸塑盒打印一份报告, 报告模板需要放到data目录下, 模板名称为"Report Templet_"加PN加".xlsx"
        '''
        printed_snlist = []
        excelfile = os.path.join('data', 'Report Templet_{}.xlsx'.format(pn))
        if not os.path.exists(excelfile):
            return -1

        datestr = datetime.datetime.today().strftime("%Y%m%d")
        targetdir = os.path.join('data', pn, datestr)
        if not os.path.exists(targetdir):
            os.makedirs(targetdir)

        reportnum = int((len(report_datas) + (counter-1)) / counter)
        for num in range(reportnum):
            rwexcel = RWExcel()
            rwexcel.open_excel(excelfile)
            sheet = rwexcel.get_sheet_by_name('Sheet1')
            index = 0
            for sn in report_datas:
                if sn not in printed_snlist:
                    printed_snlist.append(sn)
                    rwexcel.set_cell_value(sheet, index+8, 2, sn)
                    rwexcel.set_cell_value(sheet, index+8+counter+2, 2, sn)
                    rwexcel.set_cell_value(sheet, index+8, 3, round(float(report_datas[sn][2]), 2))
                    rwexcel.set_cell_value(sheet, index+8+counter+2, 3, round(float(report_datas[sn][0]), 2))
                    rwexcel.set_cell_value(sheet, index+8, 4, round(float(report_datas[sn][3]), 2))
                    rwexcel.set_cell_value(sheet, index+8+counter+2, 4, round(float(report_datas[sn][1]), 2))
                    index += 1

            # 零数入盒后面没有数据的单元格置空
            if index < counter:
                for i in range(counter-index):
                    for col in (1, 5, 6, 7, 8, 9, 10, 11, 12, 13):
                        rwexcel.set_cell_value(sheet, index+8+i, col, None)
                        rwexcel.set_cell_value(sheet, index+8+i+counter+2, col, None)

            datetimestr = datetime.datetime.today().strftime("%Y%m%d%H%M%S")
            if reportnum > 1:
                targetfname = pn + '_' + boxnumber + '_' + datetimestr + '_{}_{}.xlsx'.format(num*counter, (num+1)*counter)
            else:
                targetfname = pn + '_' + boxnumber + '_' + datetimestr + '.xlsx'.format(num*counter, (num+1)*counter)
            targetfile = os.path.join(targetdir, targetfname)
            rwexcel.save_excel(targetfile)
            rwexcel.close_excel()

            if printflag:
                win32api.ShellExecute(0, 'print', targetfile, '/d:"%s"' % win32print.GetDefaultPrinter(), '.', 0)

        return 0

    def write_and_print_warehouse_report(self, ordernumber, batch_list, batchinfo):
        excelfile = os.path.join('data', 'inwarehouse_templet.xlsx')
        if not os.path.exists(excelfile):
            return -1

        rwexcel = RWExcel()
        rwexcel.open_excel(excelfile)
        try:
            sheet = rwexcel.get_sheet_by_name('Sheet1')

            rwexcel.set_cell_value(sheet, 5, 7, r'单号：' + ordernumber)

            datestr = datetime.datetime.today().strftime("%Y-%m-%d")
            rwexcel.set_cell_value(sheet, 6, 2, datestr)

            timestr = datetime.datetime.today().strftime("%H:%M:%S")
            rwexcel.set_cell_value(sheet, 6, 7, r'时间：' + timestr)

            index = 0
            for batch in batch_list:
                rwexcel.set_cell_value(sheet, 9+index, 1, index+1)
                rwexcel.set_cell_value(sheet, 9+index, 2, batchinfo[batch][3])
                rwexcel.set_cell_value(sheet, 9+index, 3, batch)
                rwexcel.set_cell_value(sheet, 9+index, 4, batchinfo[batch][1])
                rwexcel.set_cell_value(sheet, 9+index, 5, batchinfo[batch][0])
                rwexcel.set_cell_value(sheet, 9+index, 6, batchinfo[batch][2])
                index += 1

            targetfname = u'入库单' + ordernumber + '.pdf'
            targetfile = os.path.join(os.getenv('USERPROFILE'), 'Documents', targetfname)
            logging.info('入库单: {}'.format(targetfile))
            sheet.ExportAsFixedFormat(0, targetfname)
            win32api.ShellExecute(0, 'print', targetfile, '/d:"%s"' % win32print.GetDefaultPrinter(), '.', 0)
        except Exception as ex:
            logging.warn(str(ex))
            raise
        finally:
            rwexcel.close_excel(savechange=False)

        return 0
```





