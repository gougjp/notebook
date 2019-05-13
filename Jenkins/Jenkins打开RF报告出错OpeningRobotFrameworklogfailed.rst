Jenkins打开RF报告出错: Opening Robot Framework log failed
===========================================================

**1. 错误信息:**

.. image:: images/open_robot_framework_log_failed.jpg

**2. 原因:**

Jenkins升级后增加了安全策略 
Jenkins 1.641 / Jenkins 1.625.3 introduce the Content-Security-Policy header to static files served by Jenkins (specifically, DirectoryBrowserSupport). 
This header is set to a very restrictive default set of permissions to protect Jenkins users from malicious HTML/JS files in workspaces, /userContent, 
or archived artifacts.

默认情况下，是禁止执行javascript,css等资源的。
    
.. image:: images/the_default_rule_set.jpg

**3. 解决方案:**

* 如果你的Jenkins安装为windows服务，也就是下载的是.msi版本，如下解决:

.. code::

    修改jenkins.xml文件中的
    <arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -jar "%BASE%\jenkins.war" --httpPort=8080 --webroot="%BASE%\war"</arguments>
    为
    <arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -Dhudson.model.DirectoryBrowserSupport.CSP="default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' 'unsafe-inline';" -jar "%BASE%\jenkins.war" --httpPort=8080</arguments>
    然后重启Jenkins，并重新生成robotframework日志
    
* Jenkins 为war包通过java命令运行, 如下解决：
    
.. code::

    关闭jenkins
    执行命令：
    java -Dhudson.model.DirectoryBrowserSupport.CSP= -jar E:\Jenkins\jenkins.war
    重新运行
    
* 另外，对于在Linux上的一个Jenkins服务器，没有jenkins.xml文件，可以采用以下步骤

.. code::

    1. Connect on your jenkins url (http://[IP]:8080/) 
    2. Click on Manage Jenkins from left side panel. 
    3. Click on Script Console 
    4. Copy this into the field
        System.setProperty("hudson.model.DirectoryBrowserSupport.CSP","sandbox allow-scripts; default-src 'none'; img-src 'self' data: ; style-src 'self' 'unsafe-inline' data: ; script-src 'self' 'unsafe-inline' 'unsafe-eval' ;")
    5. Click on Run button.
    