---
title: Ubuntu16.04&14.04下的docker安装
---

## 安装步骤
#### 卸载旧版本

```
$ sudo apt-get remove docker docker-engine docker.io
```

#### 安装linux的额外镜像包（仅ubuntu14.04需要）
执行如下操作，允许你使用aufs存储驱动。ubuntu16.04使用默认的overlay2存储驱动。

```
$ sudo apt-get update

$ sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
```
#### 设置安装源
通过如下步骤，设置安装源仓库，这里我们使用阿里源

```
$ sudo apt-get update

$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
    
$ curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -  

$ sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```


#### 安装docker社区版
通过如下命令将安装docker最新版本

```
$ sudo apt-get update

$ sudo apt-get install docker-ce
```


#### 检验是否安装成功
通过运行hello-world实例便可检验是否安装成功
```
$ sudo docker run hello-world
```


#### 创建一个docker用户组(可选)
docker后台进程是绑定的Unix的socket而不是TCP端口。默认情况下，Unix的socket属于用户root，其它用户要使用要通过sudo命令。由于这个原因，docker daemon通常使用root用户运行。

　　为了避免使用sudo当你使用docker命令的时候，创建一个Unix组名为docker并且添加用户。当docker daemon启动，它会分配Unix socket读写权限给所属的docker组。

```
$ sudo groupadd docker

$ sudo usermod -aG docker $USER
```
校验生效。通过运行docker命令不带sudo：docker run hello-world，如果失败会有以下类似的信息：Cannot connect to the Docker daemon. Is 'docker daemon' running on this host?确保DOCKER_HOST环境变量没有设置。如果有取消它。


## 卸载方法 
#### 卸载docker社区版:

```
$ sudo apt-get purge docker-ce 
```
#### 删除镜像、容器、卷

```
$ sudo rm -rf /var/lib/docker
```

## 参考资料
[Install Docker in ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)  
[ubuntu16.04安装docker](https://www.cnblogs.com/lighten/p/6034984.html)  
