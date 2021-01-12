# 使用pywin32处理excel文件

Windows官方接口
https://docs.microsoft.com/zh-cn/office/vba/api/word.document.exportasfixedformat
https://docs.microsoft.com/en-us/office/vba/api/excel.range.exportasfixedformat

```Python
#!/usr/bin/env python
#-*- coding:utf-8 -*-
 
#######################################################
#  用于批量删除excel的指定行                          #
#  适用于所有office，前提需要安装pywin32和office软件  #
#######################################################
 
import os
import sys
import time
import glob
import shutil
import string
import os.path
import traceback
import ConfigParser
import win32com.client
 
SPATH = ""            #需处理的excel文件目录
DPATH = ""            #处理后的excel存放目录
 
SKIP_FILE_LIST = []   #需要跳过的文件列表
MAX_SHEET_INDEX = 1   #每个excel文件的前几个表需要处理
DELETE_ROW_LIST = []  #需要删除的行号
 
def dealPath(pathname=''):
	'''deal with windows file path'''
	if pathname:
		pathname = pathname.strip()
	if pathname:
		pathname = r'%s'%pathname
		pathname = string.replace(pathname, r'/', '\\')
		pathname = os.path.abspath(pathname)
		if pathname.find(":\\") == -1:
			pathname = os.path.join(os.getcwd(), pathname)
	return pathname
 
class EasyExcel(object):
	'''class of easy to deal with excel'''
	
	def __init__(self):
		'''initial excel application'''
		self.m_filename = ''
		self.m_exists = False
		self.m_excel = win32com.client.DispatchEx('Excel.Application') #也可以用Dispatch，前者开启新进程，后者会复用进程中的excel进程
		self.m_excel.DisplayAlerts = False                             #覆盖同名文件时不弹出确认框
	
	def open(self, filename=''):
		'''open excel file'''
		if getattr(self, 'm_book', False):
			self.m_book.Close()
		self.m_filename = dealPath(filename) or ''
		self.m_exists = os.path.isfile(self.m_filename)
		if not self.m_filename or not self.m_exists:
			self.m_book = self.m_excel.Workbooks.Add()
		else:
			self.m_book = self.m_excel.Workbooks.Open(self.m_filename)
	
	def reset(self):
		'''reset'''
		self.m_excel = None
		self.m_book = None
		self.m_filename = ''
		
	def save(self, newfile=''):
		'''save the excel content'''
		assert type(newfile) is str, 'filename must be type string'
		newfile = dealPath(newfile) or self.m_filename
		if not newfile or (self.m_exists and newfile == self.m_filename):
			self.m_book.Save()
			return
		pathname = os.path.dirname(newfile)
		if not os.path.isdir(pathname):
			os.makedirs(pathname)
		self.m_filename = newfile
		self.m_book.SaveAs(newfile)
	
	def close(self):
		'''close the application'''
		self.m_book.Close(SaveChanges=1)
		self.m_excel.Quit()
		time.sleep(2)
		self.reset()
	
	def addSheet(self, sheetname=None):
		'''add new sheet, the name of sheet can be modify,but the workbook can't '''
		sht = self.m_book.Worksheets.Add()
		sht.Name = sheetname if sheetname else sht.Name
		return sht
	
	def getSheet(self, sheet=1):
		'''get the sheet object by the sheet index'''
		assert sheet > 0, 'the sheet index must bigger then 0'
		return self.m_book.Worksheets(sheet)
		
	def getSheetByName(self, name):
		'''get the sheet object by the sheet name'''
		for i in xrange(1, self.getSheetCount()+1):
			sheet = self.getSheet(i)
			if name == sheet.Name:
				return sheet
		return None
	
	def getCell(self, sheet=1, row=1, col=1):
		'''get the cell object'''
		assert row>0 and col>0, 'the row and column index must bigger then 0'
		return self.getSheet(sheet).Cells(row, col)
		
	def getRow(self, sheet=1, row=1):
		'''get the row object'''
		assert row>0, 'the row index must bigger then 0'
		return self.getSheet(sheet).Rows(row)
		
	def getCol(self, sheet, col):
		'''get the column object'''
		assert col>0, 'the column index must bigger then 0'
		return self.getSheet(sheet).Columns(col)
	
	def getRange(self, sheet, row1, col1, row2, col2):
		'''get the range object'''
		sht = self.getSheet(sheet)
		return sht.Range(self.getCell(sheet, row1, col1), self.getCell(sheet, row2, col2))
	
	def getCellValue(self, sheet, row, col):
		'''Get value of one cell'''
		return self.getCell(sheet,row, col).Value
		
	def setCellValue(self, sheet, row, col, value):
		'''set value of one cell'''
		self.getCell(sheet, row, col).Value = value
		
	def getRowValue(self, sheet, row):
		'''get the row values'''
		return self.getRow(sheet, row).Value
		
	def setRowValue(self, sheet, row, values):
		'''set the row values'''
		self.getRow(sheet, row).Value = values
	
	def getColValue(self, sheet, col):
		'''get the row values'''
		return self.getCol(sheet, col).Value
		
	def setColValue(self, sheet, col, values):
		'''set the row values'''
		self.getCol(sheet, col).Value = values
	
	def getRangeValue(self, sheet, row1, col1, row2, col2):
		'''return a tuples of tuple)'''
		return self.getRange(sheet, row1, col1, row2, col2).Value
	
	def setRangeValue(self, sheet, row1, col1, data):
		'''set the range values'''
		row2 = row1 + len(data) - 1
		col2 = col1 + len(data[0]) - 1
		range = self.getRange(sheet, row1, col1, row2, col2)
		range.Clear()
		range.Value = data
		
	def getSheetCount(self):
		'''get the number of sheet'''
		return self.m_book.Worksheets.Count
	
	def getMaxRow(self, sheet):
		'''get the max row number, not the count of used row number'''
		return self.getSheet(sheet).Rows.Count
		
	def getMaxCol(self, sheet):
		'''get the max col number, not the count of used col number'''
		return self.getSheet(sheet).Columns.Count
		
	def clearCell(self, sheet, row, col):
		'''clear the content of the cell'''
		self.getCell(sheet,row,col).Clear()
		
	def deleteCell(self, sheet, row, col):
		'''delete the cell'''
		self.getCell(sheet, row, col).Delete()
		
	def clearRow(self, sheet, row):
		'''clear the content of the row'''
		self.getRow(sheet, row).Clear()
		
	def deleteRow(self, sheet, row):
		'''delete the row'''
		self.getRow(sheet, row).Delete()
		
	def clearCol(self, sheet, col):
		'''clear the col'''
		self.getCol(sheet, col).Clear()
		
	def deleteCol(self, sheet, col):
		'''delete the col'''
		self.getCol(sheet, col).Delete()
		
	def clearSheet(self, sheet):
		'''clear the hole sheet'''
		self.getSheet(sheet).Clear()
		
	def deleteSheet(self, sheet):
		'''delete the hole sheet'''
		self.getSheet(sheet).Delete()
	
	def deleteRows(self, sheet, fromRow, count=1):
		'''delete count rows of the sheet'''
		maxRow = self.getMaxRow(sheet)
		maxCol = self.getMaxCol(sheet)
		endRow = fromRow+count-1
		if fromRow > maxRow or endRow < 1:
			return
		self.getRange(sheet, fromRow, 1, endRow, maxCol).Delete()
		
	def deleteCols(self, sheet, fromCol, count=1):
		'''delete count cols of the sheet'''
		maxRow = self.getMaxRow(sheet)
		maxCol = self.getMaxCol(sheet)
		endCol = fromCol + count - 1
		if fromCol > maxCol or endCol < 1:
			return
		self.getRange(sheet, 1, fromCol, maxRow, endCol).Delete()
		
 
def echo(msg):
	'''echo message'''
	print msg
	
def dealSingle(excel, sfile, dfile):
	'''deal with single excel file'''
	echo("deal with %s"%sfile)
	basefile = os.path.basename(sfile)
	excel.open(sfile)
	sheetcount = excel.getSheetCount()
	if not (basefile in SKIP_FILE_LIST or file in SKIP_FILE_LIST):
		for sheet in range(1, sheetcount+1):
			if sheet > MAX_SHEET_INDEX: 
				continue
			reduce = 0
			for row in DELETE_ROW_LIST:
				excel.deleteRow(sheet, row-reduce)
				reduce += 1
			#excel.deleteRows(sheet, 2, 2)
	excel.save(dfile)
	
def dealExcel(spath, dpath):
	'''deal with excel files'''
	start = time.time()
	#check source path exists or not 
	spath = dealPath(spath)
	if not os.path.isdir(spath):
		echo("No this directory :%s"%spath)
		return
	#check destination path exists or not
	dpath = dealPath(dpath)
	if not os.path.isdir(dpath):
		os.makedirs(dpath)
	shutil.rmtree(dpath)
	#list the excel file
	filelist = glob.glob(os.path.join(spath, '*.xlsx'))
	if not filelist:
		echo('The path of %s has no excel file'%spath)
		return
	#deal with excel file
	excel = EasyExcel()
	for file in filelist:
		basefile = os.path.basename(file)
		destfile = os.path.join(dpath, basefile)
		dealSingle(excel, file, destfile)
	echo('Use time:%s'%(time.time()-start))
	excel.close()
	
def loadConfig(configfile='./config.ini'):
	'''parse config file'''
	global SPATH
	global DPATH
	global SKIP_FILE_LIST
	global MAX_SHEET_INDEX
	global DELETE_ROW_LIST
	
	file = dealPath(configfile)
	if not os.path.isfile(file):
		echo('Can not find the config.ini')
		return False
	parser = ConfigParser.ConfigParser()
	parser.read(file)
	SPATH = parser.get('pathconfig', 'spath').strip()
	DPATH = parser.get('pathconfig', 'dpath').strip()
	filelist = parser.get('otherconfig', 'filelist').strip()
	index = parser.get('otherconfig', 'maxindex').strip()
	rowlist = parser.get('otherconfig', 'deleterows').strip()
	if filelist:
		SKIP_FILE_LIST = filelist.split(";")
	if rowlist:
		DELETE_ROW_LIST = map(int, rowlist.split(";"))
	MAX_SHEET_INDEX = int(index) if index else MAX_SHEET_INDEX
		
	
def main():
	'''main function'''
	loadConfig()
	if SPATH and DPATH and MAX_SHEET_INDEX:
		dealExcel(SPATH, DPATH)
	raw_input("Please press any key to exit!")
	
if __name__=="__main__":
    main()
```

config.ini文件如下：

```
[pathconfig]
#;spath表示需要处理的excel文件目录
spath=./tests
#;dpath表示处理后的excel文件目录
dpath=./dest

[otherconfig]
#;filelist表示不需要做特殊处理的excel文件列表,以英文分号分隔
filelist=
#;maxindex表示需要处理每个excel文件的前几张表
maxindex=1
#;deleterows表示需要删除的阿拉伯数字行号，用英文分号分隔
deleterows=2;3
```

### Win32将doc, xlsx, ppt转换成pdf

```Python
#-*- coding:utf-8 -*-

import os
from win32com.client import Dispatch, constants, gencache, DispatchEx


class PDFConverter():
    def __init__(self, pathname, export='.'):
        self._handle_postfix = ['doc', 'docx', 'ppt', 'pptx', 'xls', 'xlsx']
        self._filename_list = list()
        self._export_folder = os.path.join(os.path.abspath('.'), 'pdfconver')
        if not os.path.exists(self._export_folder):
            os.mkdir(self._export_folder)
        self._enumerate_filename(pathname)

    def _enumerate_filename(self, pathname):
        '''
        读取所有文件名
        '''
        full_pathname = os.path.abspath(pathname)
        if os.path.isfile(full_pathname):
            if self._is_legal_postfix(full_pathname):
                self._filename_list.append(full_pathname)
            else:
                raise TypeError('文件{}后缀名不合法! 仅支持如下文件类型: {}.'.format(pathname, ','.join(self._handle_postfix)))
        elif os.path.isfile(full_pathname):
            for relpath, _, files in os.walk(full_pathname):
                for name in files:
                    filename = os.path.join(full_pathname, relpath, name)
                    if self._is_legal_postfix(filename):
                        self._filename_list.append(filename)
        else:
            raise TypeError('文件/文件夹{}不存在或不合法!'.format(pathname))

    def _is_legal_postfix(self, filename):
        return filename.split('.')[-1].lower() in self._handle_postfix and not os.path.basename(filename).startswith('~')
        
    def run_conver(self):
        '''
        进行批量处理, 根据后缀名调用函数执行转换
        '''
        print('需要转换的文件数: {}'.format(len(self._filename_list)))
        for filename in self._filename_list:
            postfix = filename.split('.')[1].lower()
            funCall = getattr(self, postfix)
            print('原文件: {}'.format(filename))
            funCall(filename)
        print('转换完成')

    def doc(self, filename):
        '''
        doc和docx文件转换
        '''
        name = os.path.basename(filename).split('.')[0] + '.pdf'
        exportfile = os.path.join(self._export_folder, name)
        print('保存PDF文件: {}'.format(exportfile))
        gencache.EnsureModule('{00020905-0000-0000-C000-000000000046}', 0, 8, 4)
        w = Dispatch('Word.Application')
        doc = w.Documents.Open(filename)
        doc.ExportAsFixedFormat(exportfile, contants.wdExportFormatPDF, 
            Item=contants.wdExportDocumentWithMarkup,
            CreateBookmarks=constants.wdExportCreateHeadingBookmarks)
        w.Quit(constants.wdDoNotSaveChanges)
        
    def docx(self, filename):
        self.doc(filename)

    def xls(self, filename):
        '''
        xls和xlsx文件转换
        '''
        name = os.path.basename(filename).split('.')[0] + '.pdf'
        exportfile = os.path.join(self._export_folder, name)
        print('保存PDF文件: {}'.format(exportfile))
        xlApp = DispatchEx('Excel.Application')
        xlApp.Visible = False
        xlApp.DisplayAlerts = 0
        books = xlApp.Workbooks.Open(filename, False)
        books.ExportAsFixedFormat(0, exportfile)
        books.Close(False)
        xlApp.Quit()
        
    def xlsx(self, filename):
        self.xls(filename)
        
    def ppt(self, filename):
        '''
        ppt和pptx文件转换
        '''
        name = os.path.basename(filename).split('.')[0] + '.pdf'
        exportfile = os.path.join(self._export_folder, name)
        print('保存PDF文件: {}'.format(exportfile))
        gencache.EnsureModule('{00020905-0000-0000-C000-000000000046}', 0, 8, 4)
        p = Dispatch('PowerPoint.Application')
        ppt = p.Presentations.Open(filename, False, False, False)
        ppt.ExportAsFixedFormat(exportfile, 2, PrintRange=None)
        p.Quit()
        
    def pptx(self, filename):
        self.ppt(filename)
        

if __name__ == '__main__':
    # 支持文件夹批量导入
    # folder = 'tmp'
    # pathname = os.path.join(os.path.abspath('.'), folder)
    
    # 也支持单个文件的转换
    pathname = 'inwarehouse_templet.xlsx'
    
    pdfConver = PDFConverter(pathname)
    pdfConver.run_conver()
```

### excel的不同sheet存为pdf

```Python
#! -*- coding:utf-8 -*-

import os
import xlrd
from win32com.client import Dispatch, constants, gencache, DispatchEx


class PDFConverter():
    def __init__(self, pathname, sheetnum, export='.'):
        self.sheetnum = sheetnum
        self._handle_postfix = ['doc', 'docx', 'ppt', 'pptx', 'xls', 'xlsx']
        self._filename_list = list()
        self._export_folder = os.path.join(os.path.abspath('.'), 'pdfconver')
        if not os.path.exists(self._export_folder):
            os.mkdir(self._export_folder)
        self._enumerate_filename(pathname)

    def _enumerate_filename(self, pathname):
        '''
        读取所有文件名
        '''
        full_pathname = os.path.abspath(pathname)
        if os.path.isfile(full_pathname):
            if self._is_legal_postfix(full_pathname):
                self._filename_list.append(full_pathname)
            else:
                raise TypeError('文件{}后缀名不合法! 仅支持如下文件类型: {}.'.format(pathname, ','.join(self._handle_postfix)))
        elif os.path.isfile(full_pathname):
            for relpath, _, files in os.walk(full_pathname):
                for name in files:
                    filename = os.path.join(full_pathname, relpath, name)
                    if self._is_legal_postfix(filename):
                        self._filename_list.append(filename)
        else:
            raise TypeError('文件/文件夹{}不存在或不合法!'.format(pathname))

    def _is_legal_postfix(self, filename):
        return filename.split('.')[-1].lower() in self._handle_postfix and not os.path.basename(filename).startswith('~')
        
    def run_conver(self):
        '''
        进行批量处理, 根据后缀名调用函数执行转换
        '''
        print('需要转换的文件数: {}'.format(len(self._filename_list)))
        for filename in self._filename_list:
            postfix = filename.split('.')[1].lower()
            funCall = getattr(self, postfix)
            print('原文件: {}'.format(filename))
            funCall(filename)
        print('转换完成')
        
    def xls(self, filename):
        '''
        xls 和 xlsx 文件转换
        '''
        xlApp = DispatchEx('Excel.Application')
        xlApp.Visible = False
        xlApp.DisplayAlerts = 0
        books = xlApp.Workbooks.Open(filename, False)
        # 循环保存每一个sheet
        for i in range(1, self.sheetnum+1):
            sheetName = books.Sheet(i).Name
            xlSheet = books.Worksheets(sheetName)
            name = sheetName + '.pdf'
            exportfile = os.path.join(self._export_folder, name)
            xlSheet.ExportAsFixedFormat(0, exportfile)
            print('保存PDF文件: {}'.format(exportfile))
        books.Close(False)
        xlApp.Quit()
        
    def xlsx(self, filename):
        self.xls(filename)
        

if __name__ == '__main__':
    # 支持单个文件的转换
    pathname = u'原始数据.xlsx'
    # 获取到文件的sheet数
    b = xlrd.open_workbook(pathname)
    sheetnum = len(b.sheets())
    pdfconverter = PDFConverter(pathname, sheetnum)
    pdfconverter.run_conver()
```

