Jenkins Note
==============

Jenkins打开RF报告出错: Opening Robot Framework log failed
--------------------------------------------------------------

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


Python调用Jenkins的API
--------------------------

这里举例获取Jenkins上某个Job的最后一次build状态

.. code::

    import requests
    import json

    api_url = 'http://<serverip>:<serverport>/job/<jobname>/lastBuild/api/json'
    headers = {'Authorization': "Basic anVucGluZy5nb3U6QGdqcDEyMzQ==",}
    querystring = {"pretty": "true"}

    response = requests.request("GET", api_url, headers=headers, params=querystring)
    if response.text:
        result = json.loads(response.text)
        status = result['result'].strip()
        print(status)
        if status == 'SUCCESS':
            exit(0)
        else:
            exit(1)
    exit(0)

其中headers中的"Basic anVucGluZy5nb3U6QGdqcDEyMzQ=="是HTTP基本认证, 
详见：https://blog.csdn.net/wochunyang/article/details/78675325; 它是通过BASE64编码后的字符：生成方式如下

.. code::

    import base64
    #编码
    base64.b64encode('junping.gou:@gjp1234') #冒号前面是用户名，冒号后面是密码
    #输出anVucGluZy5nb3U6QGdqcDEyMzQ==
    #解码
    base64.b64decode('anVucGluZy5nb3U6QGdqcDEyMzQ==')
