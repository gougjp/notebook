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





