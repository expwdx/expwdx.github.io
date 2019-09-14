---
title: 基于docker的Spark on Mesos部署
---


### 前提概要
#### 储备
已搭好一个三台Zookeeper的集群，相关信息如下：
主机系统 | ip地址 | ID
--- | --- | ---
ubuntu16.04 server | 192.168.24.173 | 1
ubuntu16.04 server | 192.168.24.179 | 2
ubuntu16.04 server | 192.168.24.178 | 3

#### 平台软件版本规划
软件 | 版本
--- | ---
zookeeper | 3.4.13
mesos | 1.6.1
spark | 2.2.2


### Mesos部署
#### 拉取docker镜像
##### 镜像源
master: [mesosphere/mesos-master](https://hub.docker.com/r/mesosphere/mesos-master/)  
slave: [mesosphere/mesos-slave](https://hub.docker.com/r/mesosphere/mesos-slave/)
##### 拉镜像
```
$ docker pull mesosphere/mesos-master:1.6.1-rc2
$ docker pull mesosphere/mesos-master:1.6.1-rc2
```

#### 启动容器
##### 设置host
分别登陆三台服务器设置主机ip到环境变量，或者在启动mesos的参数中设置，本次我们选择在启动参数中设置。
```
$ HOST_IP=192.168.24.17x
```
##### master
```
$ docker run -d --net=host --restart always --name m1 \
  -e MESOS_PORT=5050 \
  -e MESOS_HOSTNAME=192.168.24.173 \
  -e MESOS_IP=192.168.24.173 \
  -e MESOS_ZK=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_QUORUM=2 \
  -e MESOS_REGISTRY=in_memory \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  mesosphere/mesos-master:1.5.0
```
```
$ docker run -d --net=host --restart always --name m2 \
  -e MESOS_PORT=5050 \
  -e MESOS_HOSTNAME=192.168.24.179 \
  -e MESOS_IP=192.168.24.179 \
  -e MESOS_ZK=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_QUORUM=2 \
  -e MESOS_REGISTRY=in_memory \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  mesosphere/mesos-master:1.5.0
```
```
$ docker run -d --net=host --restart always --name m3 \
  -e MESOS_PORT=5050 \
  -e MESOS_HOSTNAME=192.168.24.178 \
  -e MESOS_IP=192.168.24.178 \
  -e MESOS_ZK=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_QUORUM=2 \
  -e MESOS_REGISTRY=in_memory \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  mesosphere/mesos-master:1.5.0
```
##### marathon
192.168.24.178下部署marathon:
```
$ docker run -d --net=host --restart always --privileged \
  --name mm3 \
  mesosphere/marathon:latest \
  --master zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  --zk zk://92.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/marathon \
  --http_port 8089 \
  --http_address 192.168.24.178 
```

##### slave
在192.168.24.173下暂时不部署slave，但预留命名m1s1和m1s2. 
```
$ docker run -it -d --restart always --net=host --name m1s1 --privileged \
  -e MESOS_PORT=5051 \
  -e MESOS_HOSTNAME=192.168.24.173 \
  -e MESOS_IP=192.168.24.173 \
  -e MESOS_MASTER=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_CONTAINERIZERS=docker,mesos \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  mesosphere/mesos-slave:1.6.1-rc2
```
```
$ docker run -it -d --restart always --net=host --name m1s2 --privileged \
  -e MESOS_PORT=5052 \
  -e MESOS_MASTER=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  mesosphere/mesos-slave:1.6.1-rc2
```

在192.168.24.179下启动2个slave，分别命名为m2s1和m2s2.
```
$ docker run -it -d --restart always --net=host \
  --name m2s1 \
  --privileged \
  -e MESOS_PORT=5051 \
  -e MESOS_MASTER=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_SWITCH_USER=0 \
  -e MESOS_CONTAINERIZERS=docker,mesos \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /cgroup:/cgroup \
  -v /sys:/sys \
  -v /usr/local/bin/docker:/usr/local/bin/docker \
  mesosphere/mesos-slave:1.5.0 \
  --ip=192.168.24.179 
```
```
$ docker run -it -d --restart always --net=host --name m2s2 --privileged \
  -e MESOS_PORT=5052 \
  -e MESOS_MASTER=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  mesosphere/mesos-slave:1.6.1-rc2
```
```
$ docker run -it -d --restart always --net=host --name m2s3 --privileged \
  -e MESOS_PORT=5053 \
  -e MESOS_MASTER=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_SWITCH_USER=0 \
  -e MESOS_CONTAINERIZERS=docker,mesos \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /cgroup:/cgroup \
  -v /sys:/sys \
  -v /usr/local/bin/docker:/usr/local/bin/docker \
  mesosphere/mesos-slave:1.6.1-rc2 \
  --ip=192.168.24.179 \
  --launcher=posix
```
```
docker run -d --net=host --privileged \
  --name mesos_slave \
  -e MESOS_PORT=5051 \
  -e MESOS_MASTER=zk://192.168.24.179:2181/mesos \
  -e MESOS_SWITCH_USER=0 \
  -e MESOS_CONTAINERIZERS=docker,mesos \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /sys/fs/cgroup:/cgroup \
  -v /sys:/sys \
  -v /usr/bin/docker:/usr/local/bin/docker \
  mesosphere/mesos-slave:1.1.0-2.0.107.ubuntu1404 \
  --ip=192.168.24.179 \
  --launcher=posix
```
在192.168.24.178下启动2个slave，分别命名为m3s1和m3s2.
```
$ docker run -it -d --restart always --net=host --name m3s1 --privileged \
  -e MESOS_PORT=5053 \
  -e MESOS_HOSTNAME=192.168.24.178 \
  -e MESOS_IP=192.168.24.178 \
  -e MESOS_MASTER=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_CONTAINERIZERS=docker,mesos \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /cgroup:/cgroup \
  -v /sys:/sys \
  -v /usr/local/bin/docker:/usr/local/bin/docker \
  mesosphere/mesos-slave:1.6.1-rc2
```
```
$ docker run -it -d --restart always --net=host --name m3s2 --privileged \
  -e MESOS_PORT=5052 \
   -e MESOS_HOSTNAME=192.168.24.178 \
  -e MESOS_IP=192.168.24.178 \
  -e MESOS_MASTER=zk://192.168.24.173:2181,192.168.24.179:2181,192.168.24.178:2181/mesos \
  -e MESOS_LOG_DIR=/var/log/mesos \
  -e MESOS_WORK_DIR=/var/tmp/mesos \
  -v "$(pwd)/log/mesos:/var/log/mesos" \
  -v "$(pwd)/tmp/mesos:/var/tmp/mesos" \
  mesosphere/mesos-slave:1.6.1-rc2
```


### Spark嵌入
