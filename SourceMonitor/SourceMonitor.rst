SourceMonitor部署
========================

一. 环境安装

* 下载SourceMonitor: http://www.campwoodsw.com/sourcemonitor.html

* 双击安装

* 将安装路径添加到PATH环境变量中

二. command.xml 配置选项

所有的命令都是XML标签的形式定义的, 并且所有命令都在根标签<sourcemonitor_commands>内

* **write_log**

这个标签在执行批处理命令文件时控制日志的输出, 通常情况下, 这个属性是命令行文件的第一个命令; 值为true或者false, 默认为true; 
命令行文件中的这个标签只是在执行脚本期间控制日志, 而不会修改SourceMonitor软件的日志配置

* **log_all_to_console**

如果指定了这个属性标签, SourceMonitor除了将所有日志消息写入日志文件以外, 还会输出到控制台; 通常情况下, 这个命令应该放在所有命令的最开始; 指定方式如下:

.. code::

    <log_all_to_console/>
    
* **command**

文件中的每一个单独命令都必须在command标签中. 一个批处理文件可以包含任何数量的命令. 

* **parse_utf8_files**

解析UTF8文件, 值为true或者false

* **project_file**

工程文件, 可以是相对路径或者绝对路径; 如果指定的文件不存在, 则会创建一个文件新的文件. 文件扩展名必须为.smproj或者.smp; 路径中可以使用环境变量

* **project_language**

该命令只针对新工程, 对于老的工程, 将忽略该选项; 可以取值: C, C++ (or CPP), C# (or CSharp), Java, VB.NET, VB (for VB6), Delphi, HTML


.. code::

    <?xml version="1.0" encoding="UTF-8" ?>

     

    <!-- =======================================================================

    Examples of typical SourceMonitor command line script commands.

    ======================================================================== -->

     

    <sourcemonitor_commands>

     

       <!-- ===================================================================

        The write_log element value is used for execution of commands in this

        file, not the log errors flag set in SourceMonitor's options dialog.

        If this element is missing, command progress and errors will be written

        to the SourceMonitor log file.

        =================================================================== -->

     

       <write_log>true</write_log>

     

       <!-- ===============================================================

         This command is typical for a project that does not yet exist.

         The file name becomes the project name of the new project.

       ================================================================ -->

     

       <command>

     

           <project_file>E:\sfp_plus_10g_epon_olt\04 src\02 firmware\MyProject.smproj</project_file>

     

           <!-- ===============================================================

            For a new project, you must specify the language and the location

            of the source code. You can provide an absolute path for the project

            file (as in this example) or you can provide a relative path. In the

            latter case, the path will be resolved relative to the current

            working directory. If you want to specify the project file relative

            to the location of your script file, you can use element

            <project_file_wrt_script> instead of <project_file>.

     

            You may also set the Modified Complexity and Ignore Blank Lines options.

           ================================================================ -->

     

           <project_language>C</project_language>

           <modified_complexity>true</modified_complexity>

           <ignore_blank_lines>false</ignore_blank_lines>

           <source_directory>E:\sfp_plus_10g_epon_olt\04 src\02 firmware</source_directory>

     

           <!-- ===============================================================

            If the optional flag element, "exclude_subdirectories", is "true"

            then the listed subdirectories are NOT.to be included in the

            project (all other subdirectories WILL be included). If this flag

            element is missing or "false" then only the subdirectories listed

            will be included in the project. The "source_subtree" element

            identifies the parent of a subdirectory tree, all of which will be

            included/excluded. Element "source_subdirectory" applies to a

            single directory and not its children (if any).

           ================================================================ -->

     

           <source_subdirectory_list>

               <exclude_subdirectories>true</exclude_subdirectories>

           </source_subdirectory_list>

     

           <parse_utf8_files>True</parse_utf8_files>

        

           <checkpoint_name>Baseline</checkpoint_name>

     

           <!-- ===============================================================

            For either a new or existing project, you can specify that the maximum

            block depth be display as measured. That means that if the maximum is

            greater than 9 the maximum depth display will show the actual value,

            not "9+". For example:

           ================================================================ -->

     

          <show_measured_max_block_depth>True</show_measured_max_block_depth>

     

           <!-- ===============================================================

            For either a new or existing project, you can override the default

            or current file extensions (set in the Options dialog) for source

            files to be included in a new checkpoint. All files with the

            extensions you specify here will be included in the new checkpoint.

           ================================================================ -->

     

           <file_extensions>*.h,*.c</file_extensions>

           <include_subdirectories>true</include_subdirectories>

     

           <!-- ===============================================================

            If you are creating a project that counts DOC comments separately, then you

            can provide a number in the following element to specify the kind of

            comments to ignore at the beginning and end of each source file:

            1 => normal comments only

            2 => DOC comments only

            3 => both normal and DOC comments

            For example,

           ================================================================ -->

            <ignore_headers_footers>2 DOC only</ignore_headers_footers>

     

           <!-- ===============================================================

            For projects that don't count DOC comments, just set this element to

            true or false:

           ================================================================ -->

           <ignore_headers_footers>True</ignore_headers_footers>

     

           <!-- ===============================================================

            Export of metrics data is supported for a single checkpoint per

            command (the one identified in the <checkpoint_name> tag or one that

            is automatically created as explained below). Exported metrics data

            is specified by the export type: "1" for the project summary as XML,

            "2" for checkpoint details as XML, or "3" project details as CSV.

            You specify the type with a number 1, 2 or 3 in the element's

            contents. All other text is ignored. The following examples are

            all valid specification of export type 2:

              <export_type>2 (project details as XML)</export_type>

              <export_type>Project details as XML: 2</export_type>

              <export_type>2</export_type>

           ================================================================ -->

     

           <export>

               <export_insert>xml-stylesheet type='text/xsl' href='SourceMonitor.xslt'</export_insert>

               <export_file>E:\sfp_plus_10g_epon_olt\04 src\02 firmware\dump.xml</export_file>

               <!-- <export_type>2 (project details as XML)</export_type> -->
               <export_type>5 (Export method metrics in XML format)</export_type>

               <!-- <export_option>2 (export raw numbers instead of percentages and ratios)</export_option> -->
               <export_option>3 (Export method metrics (XML format only))</export_option>

           </export>

     

           <!-- ===============================================================

            When the </command> tag is encountered, the command specified above

            is executed. In this sample command, the project and checkpoint are

            created and the XML export file is generated.

           ================================================================ -->

     

       </command>

    </sourcemonitor_commands>

2. 执行命令

.. code::

    SourceMonitor.exe /C command.xml /L SM.log



三. HTML报告生成

1. Ant安装

* 下载: https://mirror.bit.edu.cn/apache//ant/binaries/

* 解压后将路径添加到PATH环境变量


