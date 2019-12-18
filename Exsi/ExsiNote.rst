Exsi 6.7 安装
===============================

一. 服务器安装:

去官网下载iso包, 刻录到U盘, 然后从U盘启动即可

No Network Adapters
---------------------------

以前安装6.0的时候没有出现过任何错误, 但是最近安装新版本(6.7)的时候, 出现了以下错误, 提示没有网络适配器

.. image:: images/0-1.jpeg

可以通过以下方式解决(当前是以win7 x64环境):

* 安装 VMware PowerCLI

下载地址: http://down.whsir.com/downloads/VMware-PowerCLI-6.5.0-4624819.exe

.. image:: images/0-2.jpeg

.. image:: images/0-3.jpeg

可能会弹出提示需要下载PowerShell3.0, 下载地址：https://download.microsoft.com/download/E/7/6/E76850B8-DA6E-4FF5-8CCE-A24FC513FD16/Windows6.1-KB2506143-x64.msu

如果还有其他依赖缺失，请自行下载

.. image:: images/0-4.jpeg

.. image:: images/0-5.jpeg

重启系统后重新运行VMware-PowerCLI-6.5.0安装

.. image:: images/0-6.jpeg

.. image:: images/0-7.jpeg

.. image:: images/0-8.jpeg

.. image:: images/0-9.jpeg

.. image:: images/0-10.jpeg

安装完成后电脑桌面会生成一个VMware PowerCLI，运行后如果看到以下报错

.. image:: images/0-11.jpeg

解决办法：开始-所有程序-附件-Windows PowerShell(管理员身份运行), 输入以下命令

.. code::

    Set-ExecutionPolicy Unrestricted

.. image:: images/0-12.jpeg

然后重新启动VMware PowerCLI会有个默认的设置, 直接回车即可, 第二次启动后可以看到如下画面, 此时VMware PowerCLI安装完成

.. image:: images/0-13.jpeg

* 下载 http://down.whsir.com/downloads/ESXi-Customizer-PS-v2.6.0.ps1

* 连网打包ISO:

打开VMware PowerCLI, 进入ESXi-Customizer-PS-v2.6.0.ps1文件所在目录, 执行如下命令:

.\ESXi-Customizer-PS-v2.6.0.ps1  -v67  -vft  -load  net55-r8168,sata-xahci

    参数解释：

    　　-v67    指定是VMware EXSI为 6.7版本

    　　-vft      在线连接到V-Front在线仓库下载驱动到VMware EXSI.ISO镜像包

    　　-load   指定要添加到VMware EXSI.ISO镜像的驱动

    　　Sata-xahci   常用的SATA控制器
    
详细命令参数可以查看此网址： https://www.v-front.de/p/esxi-customizer-ps.html#download

.. image:: images/0-14.jpeg

等执行结束后就可以看到当前目录下有个ESXi-6.7.0-20190802001-standard-customized.iso镜像文件了

* 离线打包ISO, 如果连网导报失败则可以使用该方法:

下载Esxi6.7: http://down.whsir.com/downloads/ESXi670-201905001.zip

下载net55-r8168网卡驱动: http://down.whsir.com/downloads/net55-r8168-8.045a-napi.x86_64.vib; 对于其他网卡可以在https://vibsdepot.v-front.de/wiki/index.php/List_of_currently_available_ESXi_packages中查找

net55-r8168驱动支持网卡型号（Realtek RTL8111B / RTL8168B / RTL8111/RTL8168 / RTL8111C / RTL8111CP / RTL8111D(L) / RTL8168C / RTL8111DP / RTL8111E / RTL8168E / RTL8111F / RTL8411 / RTL8111G / RTL8111GUS / RTL8411B(N) / RTL8118AS / D-Link DGE-528T）

注意：下载的驱动是vib格式。

将ESXi-Customizer-PS-v2.6.0.ps1和ESXi670-201905001.zip放到当前目录下, 然后在当前目录下创建whsir目录, 将net55-r8168-8.045a-napi.x86_64.vib放到whsir目录中, 然后在VMware PowerCLI执行以下命令：

.. code::

.\ESXi-Customizer-PS-v2.6.0.ps1 -izip .\ESXi670-201905001.zip -pkgDir whsir\

注意：打包时会自动在whsir文件夹内查找vib的驱动，生成的ESXI6.7.iso镜像会在当前目录下

如果一切正常，可看到以下内容

.. image:: images/0-15.jpeg

系统内存强制要求大于4G
---------------------------

放入光盘或U盘，开始安装，一直普通流程到Welcome画面，按ALT+F1
登陆界面账号：root 密码为空
cd /usr/lib/vmware/weasel/util
rm upgrade_precheck .pyc #文件可能不存在
mv upgrade_precheck .py upgrade_precheck .py.old
cp upgrade_precheck .py.old upgrade_precheck .py #如果直接把这个文件备份.old
chmod 666 upgrade_precheck .py　　#增加权限后在原文件修改提示权限不允许
vi upgrade_precheck .py
编辑文本界面中查找 MEM_MIN #vi 搜索命令是斜杠 /
MEM_MIN_SIZE= (4*1024) 改成 MEM_MIN_SIZE= (2*1024)
wq! 强制保存退出
ps -c |grep weasel #不结束进程，直接适用ALT+F2貌似无效
kill -9 [进程ID] #
此时正常情况下会跳回欢迎界面，如不跳回按ALT+F2返回继续安装










参考:

https://vibsdepot.v-front.de/wiki/index.php/Welcome

http://www.mamicode.com/info-detail-2754061.html

https://blog.whsir.com/post-4462.html

https://www.cnblogs.com/Sunzz/p/11438066.html