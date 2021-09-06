# Jenkins Note

### Jenkins打开RF报告出错: Opening Robot Framework log failed

**1. 错误信息:**

![](images/open_robot_framework_log_failed.jpg)

**2. 原因:**

Jenkins升级后增加了安全策略 
Jenkins 1.641 / Jenkins 1.625.3 introduce the Content-Security-Policy header to static files served by Jenkins (specifically, DirectoryBrowserSupport). 
This header is set to a very restrictive default set of permissions to protect Jenkins users from malicious HTML/JS files in workspaces, /userContent, 
or archived artifacts.

默认情况下，是禁止执行javascript,css等资源的。
    
![](images/the_default_rule_set.jpg)

**3. 解决方案:**

- 如果你的Jenkins安装为windows服务，也就是下载的是.msi版本，如下解决:

```xml
修改jenkins.xml文件中的
<arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -jar "%BASE%\jenkins.war" --httpPort=8080 --webroot="%BASE%\war"</arguments>
为
<arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -Dhudson.model.DirectoryBrowserSupport.CSP="default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' 'unsafe-inline';" -jar "%BASE%\jenkins.war" --httpPort=8080</arguments>
然后重启Jenkins，并重新生成robotframework日志
```
    
- Jenkins 为war包通过java命令运行, 如下解决：
    
```Shell
关闭jenkins
执行命令：
java -Dhudson.model.DirectoryBrowserSupport.CSP= -jar E:\Jenkins\jenkins.war
重新运行
```
    
- 另外，对于在Linux上的一个Jenkins服务器，没有jenkins.xml文件，可以采用以下步骤

```
1. Connect on your jenkins url (http://[IP]:8080/) 
2. Click on Manage Jenkins from left side panel. 
3. Click on Script Console 
4. Copy this into the field
    System.setProperty("hudson.model.DirectoryBrowserSupport.CSP","sandbox allow-scripts; default-src 'none'; img-src 'self' data: ; style-src 'self' 'unsafe-inline' data: ; script-src 'self' 'unsafe-inline' 'unsafe-eval' ;")
5. Click on Run button.
```

### 在浏览器中打开Jenkins的userContent下的HTML文件时, 如果HTML中有javascript, 则加载javascript脚本失败

这个是jenkins的安全策略设置，默认的设置为：

```
不允许JavaScript
不允许插件(对象/嵌入)
没有内联CSS或CSS允许从其他网站
不允许从其他网站图片
不允许框架
不允许web字体
不允许XHR / AJAX等
```

解决方法一:

在Jenkins的Jenkins Script Console(脚本命令行)里设置如下:

```
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP",  "script-src 'unsafe-inline'")
```

这种方案在Jenkins服务重启过后就失效了

解决方法二: 

Linux下, 在jenkins的启动文件catalina设置如下

```Shell
CATALINA_OPTS="-Dhudson.model.DirectoryBrowserSupport.CSP=\"default-src 'self'; style-src 'self' 'unsafe-inline' www.google.com ajax.googleapis.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' www.google.com; img-src 'self' data:; child-src 'self'\""
```

Windows下, 修改jenkins.xml中arguments

将

```
<arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -jar "E:\Jenkins\jenkins.war" --httpPort=8080 --webroot="%JENKINS_HOME%\war"</arguments>
```

改为

```
<arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -Dhudson.model.DirectoryBrowserSupport.CSP="script-src 'unsafe-inline'" -jar "E:\Jenkins\jenkins.war" --httpPort=8080 --webroot="%JENKINS_HOME%\war"</arguments>
```

附Jenikns其他策略设置:

- 设置自定义:

```
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox; default-src 'self';")
```

- 清除自定义:

```
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")
```

- 恢复默认设置:

```
System.clearProperty("hudson.model.DirectoryBrowserSupport.CSP")
```

- 查看当前设置:

```
System.getProperty("hudson.model.DirectoryBrowserSupport.CSP")
```

### Python调用Jenkins的API

这里举例获取Jenkins上某个Job的最后一次build状态

```Python
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
```

其中headers中的"Basic anVucGluZy5nb3U6QGdqcDEyMzQ=="是HTTP基本认证, 
详见：https://blog.csdn.net/wochunyang/article/details/78675325; 它是通过BASE64编码后的字符：生成方式如下

```Python
import base64
#编码
base64.b64encode('junping.gou:@gjp1234') #冒号前面是用户名，冒号后面是密码
#输出anVucGluZy5nb3U6QGdqcDEyMzQ==
#解码
base64.b64decode('anVucGluZy5nb3U6QGdqcDEyMzQ==')
```

也可以在Jenkins的用户设置界面直接查看API Token: 点击右上的用户名, 点击左边的设置, 点击API Token下的Show API Token 

### 获取JOB的控制台输出

```Shell
curl -u <username>:<password> ${BUILD_URL}/consoleText --output output.log
```

### Jenkins 插件

- Job Configuration History #查看Job配置历史

```
https://plugins.jenkins.io/jobConfigHistory
```

- Sectioned View Plugin #对view中的Job进行分类

```
https://wiki.jenkins.io/display/JENKINS/Sectioned+View+Plugin
```

- Timestamper #JOB控制台输出中加入时间戳

```
https://wiki.jenkins.io/display/JENKINS/Timestamper
```

- build-name-setter #设置build显示的名字

```
https://wiki.jenkins.io/display/JENKINS/Build+Name+Setter+Plugin
```

- Parameterized Trigger # 带参数触发下游Job

```
https://plugins.jenkins.io/parameterized-trigger
```

- Build Pipeline Plugin # Pipeline的形式显示JOB的调用流程

```
https://wiki.jenkins.io/display/JENKINS/Build+Pipeline+Plugin
https://www.cnblogs.com/luodengxiong/p/5535218.html
https://blog.csdn.net/GW569453350game/article/details/51882246
```

- Environment Injector #Job中设置环境变量

```
https://wiki.jenkins.io/display/JENKINS/EnvInject+Plugin
```

- AnsiColor   # 使Jenkins控制台输出显示颜色

```
https://plugins.jenkins.io/ansicolor/
```

在pipeline中可以按一下方式使用:

```Groovy
pipeline {
    agent any
    options {
        ansiColor('xterm')
    }
    stages {
        stage('Build') {
            steps {
                echo '\033[34mHello\033[0m \033[33mcolorful\033[0m \033[35mworld!\033[0m'
            }
        }
    }
}
```

或者

```Groovy
ansiColor('css') {
  sh "ls -al"
}

echo 'this will be rendered as-is'
// multiple ansiColor steps within one pipeline are also supported

ansiColor('vga') {
  echo '\033[42m\033[97mWhite letters, green background\033[0m'
}
```

### 在Centos7系统上, 通过tomcat部署Jenkins

首先要安装jdk, 这里省略

1. 下载tomcat

可以在 https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/ 路径下找合适的版本

```Shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.50/bin/apache-tomcat-9.0.50.tar.gz
```

2. 下载Jenkins

```Shell
wget http://mirrors.jenkins-ci.org/war/latest/jenkins.war
```

3. 部署Jenkins

解压apache-tomcat-9.0.50.tar.gz, 并拷贝到/usr/local/目录下, 重名名为tomcat

```shell
tar -zxvf apache-tomcat-9.0.50.tar.gz
mv apache-tomcat-9.0.50 /usr/local/tomcat
```

将jenkins.war拷贝到/usr/local/tomcat/webapps/目录下, 设置JENKINS_HOME环境变量, 然后启动tomcat

```Shell
cp -f jenkins.war /usr/local/tomcat/webapps/
export JENKINS_HOME=/home/jenkins
cd /usr/local/tomcat/bin
./catalina.sh start
```

关闭防火墙

```Shell
查看状态
firewall-cmd --state
停止firewall
systemctl stop firewalld.service
禁止firewall开机启动
systemctl disable firewalld.service 
```

在浏览器中访问Jenkins

http://<Jenkins_server_ip>:8080/jenkins/

另外, 如果想直接通过http://<Jenkins_server_ip>:8080访问Jenkins, 则需要将/usr/local/tomcat/webapps/目录下的ROOT文件夹重命名, 比如为ROOT_BAK, 然后将jenkins.war重命名为ROOT.war, 然后再启动服务

4. Jenkins中打开一些网页JS格式无法正常显示, 可以在catalina.sh文件中加入以下配置
CATALINA_OPTS="-Dhudson.model.DirectoryBrowserSupport.CSP=\"default-src 'self'; style-src 'self' 'unsafe-inline' www.google.com ajax.googleapis.com; script-src 'self' 'unsafe-inline' 'unsafe-eval' www.google.com; img-src 'self' data:; child-src 'self'\""

### tomcat访问本地服务器文件夹中的文件

1. 向/usr/local/tomcat/conf/server.xml文件最后的\<Host\>\<\/Host\>内部添加虚拟路径

```Xml
<Context path="/binaries" docBase="/home/sdr/binaries" debug="0" reloadable="true"/>
```

![](images/context_path.jpg)

path: 匹配url路径开头

docBase: 要访问的本地资源路径信息，不包含文件

可以同时添加多个路径

2. 改web.xml文件中的\<servlet\>配置, 搜索listings

将\<param-value\>false\<\/param-value\> 改为\<param-value\>true\<\/param-value\>

![](images/servlet_param_value.jpg)

3. 如果路径中包含中文路径则需要在server.xml中配置URIEncoding="utf-8"

![](images/server_path_encoding.jpg)

### jenkins中shell脚本编写的两个注意点

在jenkins的build中, 如果用shell脚本的话, 要记住有两个地方要注意

1. 由于默认jenkins是使用/bin/bash -xe xxx.sh来调用脚本的, 所以不同于日常写的脚本, 任何一行返回值不为0都会使得脚本中途退出, 从而build失败.

    解决方法1: 在开头加#!/bin/bash(试过可以)

    解决方法2: 在开头加set +e(没试过, 应该是可以的)

2. 由于jenkins默认在build结束后杀死所有build相关进程, 所以nohup的进程也会被杀死, 如果想正常使用nohup, 要加一句BUILD_ID=DONTKILLME





