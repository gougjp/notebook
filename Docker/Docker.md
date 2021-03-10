# Docker环境搭建

### Docker安装

系统: Centos7

1. 安装依赖库

    ```Shell
    yum install -y yum-utils device-mapper-persistent-data lvm2
    ```
  
2. 安装docker仓库

    ```Shell
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    ```

3. 更新yum包索引

    ```Shell
    yum makecache fast
    ```

4. 安装最新版本的Docker CE

    ```Shell
    yum install docker-ce
    ```

5. Docker版本查看

    ```Shell
    [root@localhost opt]# docker version
    Client: Docker Engine - Community
     Version:           20.10.5
     API version:       1.41
     Go version:        go1.13.15
     Git commit:        55c4c88
     Built:             Tue Mar  2 20:33:55 2021
     OS/Arch:           linux/amd64
     Context:           default
     Experimental:      true

    Server: Docker Engine - Community
     Engine:
      Version:          20.10.5
      API version:      1.41 (minimum version 1.12)
      Go version:       go1.13.15
      Git commit:       363e9a8
      Built:            Tue Mar  2 20:32:17 2021
      OS/Arch:          linux/amd64
      Experimental:     false
     containerd:
      Version:          1.4.3
      GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
     runc:
      Version:          1.0.0-rc92
      GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
     docker-init:
      Version:          0.19.0
      GitCommit:        de40ad0
    [root@localhost opt]#
    ```

### Docker配置

1. 加速器配置: 阿里云为例, https://cr.console.aliyun.com/cn-qingdao/instances/mirrors

    [](images/jiasuqi.jpeg)

2. 下载镜像: 以Jenkins镜像为例

    镜像搜索:
    
    [](images/imagesearch.jpeg)
    
    因为Jenkins镜像没有latest版本, 所以在pull的时候需要跟tag号
    
    [](images/jenkinsimage.jpeg)

    ```Shell
    [root@localhost opt]# docker pull jenkins:2.60.3
    2.60.3: Pulling from library/jenkins
    55cbf04beb70: Pull complete
    1607093a898c: Pull complete
    9a8ea045c926: Pull complete
    d4eee24d4dac: Pull complete
    c58988e753d7: Pull complete
    794a04897db9: Pull complete
    70fcfa476f73: Pull complete
    0539c80a02be: Pull complete
    54fefc6dcf80: Pull complete
    911bc90e47a8: Pull complete
    38430d93efed: Pull complete
    7e46ccda148a: Pull complete
    c0cbcb5ac747: Pull complete
    35ade7a86a8e: Pull complete
    aa433a6a56b1: Pull complete
    841c1dd38d62: Pull complete
    b865dcb08714: Pull complete
    5a3779030005: Pull complete
    12b47c68955c: Pull complete
    1322ea3e7bfd: Pull complete
    Digest: sha256:eeb4850eb65f2d92500e421b430ed1ec58a7ac909e91f518926e02473904f668
    Status: Downloaded newer image for jenkins:2.60.3
    docker.io/library/jenkins:2.60.3
    [root@localhost opt]#
    [root@localhost opt]# docker images
    REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
    hello-world   latest    bf756fb1ae65   14 months ago   13.3kB
    jenkins       2.60.3    cd14cecfdb3a   2 years ago     696MB
    [root@localhost opt]#
    ```

### Docker常用命令

1. **docker images**: 查看已安装的docker镜像

2. **docker image rm <imageID>**: 删除镜像

3. **docker pull [OPTIONS] NAME[:TAG|@DIGEST]**: 获取镜像

4. **docker search**: 从仓库搜索镜像

5. **docker exec –it <containerID> bash**: 进入docker容器内部

6. **docker run**: 运行docker镜像

7. **docker ps**: 查看运行容器进程

8. **docker run –d**: 在后台运行docker镜像

9. **docker run –p 3306:3306**: 端口映射

10. **docker stop <containerID>**：停止正在运行的容器

### 将容器中的数据目录挂载到宿主机

以Jenkins镜像为例

1. 启动Jenkins镜像

    ```Shell
    [root@localhost local]# docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins:2.60.3
    825cd4485f22ac775f224dbbc4cab69b59b79cb812ed511bdb318f6ab7f9233f
    [root@localhost local]#
    ```
    
2. 将容器中的/var/jenkins_home/userContent目录复制到宿主机中/usr/local/jenkins/userContent

    ```Shell
    [root@localhost local]# mkdir /usr/local/jenkins
    [root@localhost local]# docker cp jenkins:/var/jenkins_home/userContent /usr/local/jenkins/userContent
    [root@localhost local]# ls -al /usr/local/jenkins/
    total 0
    drwxr-xr-x   3 root root  25 Mar 10 10:20 .
    drwxr-xr-x. 13 root root 146 Mar 10 10:19 ..
    drwxr-xr-x   2 root root  24 Mar 10 10:14 userContent
    [root@localhost local]# 
    ```

3. 停止前面运行的容器

    ```Shell
    [root@localhost local]# docker ps
    CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                                              NAMES
    825cd4485f22   jenkins:2.60.3   "/bin/tini -- /usr/l…"   8 minutes ago   Up 8 minutes   0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins
    [root@localhost local]# docker stop 825cd4485f22
    825cd4485f22
    [root@localhost local]#
    ```

4. 再次运行容器, 添加挂载命令

    ```Shell
    [root@localhost local]# docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v /usr/local/jenkins/userContent:/var/jenkins_home/userContent jenkins:2.60.3
    docker: Error response from daemon: Conflict. The container name "/jenkins" is already in use by container "825cd4485f22ac775f224dbbc4cab69b59b79cb812ed511bdb318f6ab7f9233f". You have to remove (or rename) that container to be able to reuse that name.
    See 'docker run --help'.
    [root@localhost local]# docker rm -f jenkins
    jenkins
    [root@localhost local]#
    [root@localhost local]#
    [root@localhost local]# docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v /usr/local/jenkins/userContent:/var/jenkins_home/userContent jenkins:2.60.3
    17755738c01e84d00ac25eeee06582111e52f8a2f283deb068aa92b827fd7535
    [root@localhost local]#
    ```

5. 确认挂载成功, 修改宿主机的/usr/local/jenkins/userContent/目录中的readme.txt文件, 查看容器中文件是否被修改 

    ```Shell
    [root@localhost local]# cd /usr/local/jenkins/userContent/
    [root@localhost userContent]# ls -al
    total 4
    drwxr-xr-x 2 root root 24 Mar 10 10:14 .
    drwxr-xr-x 3 root root 25 Mar 10 10:20 ..
    -rw-r--r-- 1 root root 82 Mar 10 10:14 readme.txt
    [root@localhost userContent]# cat readme.txt
    Files in this directory will be served under your http://yourjenkins/userContent/
    [root@localhost userContent]# echo ============= >> readme.txt
    [root@localhost userContent]# cat readme.txt
    Files in this directory will be served under your http://yourjenkins/userContent/
    =============
    [root@localhost userContent]# docker ps
    CONTAINER ID   IMAGE            COMMAND                  CREATED              STATUS              PORTS                                              NAMES
    17755738c01e   jenkins:2.60.3   "/bin/tini -- /usr/l…"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins
    [root@localhost userContent]# docker exec -it 17755738c01e bash
    jenkins@17755738c01e:/$ cd /var/jenkins_home/userContent/
    jenkins@17755738c01e:~/userContent$ ls -al
    total 8
    drwxr-xr-x  2 root    root      24 Mar 10 02:14 .
    drwxr-xr-x 13 jenkins jenkins 4096 Mar 10 02:27 ..
    -rw-r--r--  1 root    root      96 Mar 10 02:27 readme.txt
    jenkins@17755738c01e:~/userContent$ cat readme.txt
    Files in this directory will be served under your http://yourjenkins/userContent/
    =============
    jenkins@17755738c01e:~/userContent$
    ```