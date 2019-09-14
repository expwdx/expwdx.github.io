---
title: Docker Compose安装
---

## 安装 Docker Compose

#### 官方安装方法
1. 运行如下命令下载最新版本
```
$ sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```
2. 添加执行权限
```
$ sudo chmod +x /usr/local/bin/docker-compose
```
3. 安装 [completion](https://docs.docker.com/compose/completion/) 以便能在bash和zsh的shell中运行completion命令（可选）

4. 测试安装是否成功
```
$ docker-compose --version
docker-compose version 1.22.0, build 1719ceb
```

#### 加速通道安装
Docker Compose 存放在Git Hub，不太稳定。 
你可以也通过执行下面的命令，高速安装Docker Compose。
```
$ sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose  

$ sudo chmod +x /usr/local/bin/docker-compose  
```
## 参考资料
[DaoCloud-install-compose](http://get.daocloud.io/#install-compose)  
[Docker-install-compose](https://docs.docker.com/compose/install/)
