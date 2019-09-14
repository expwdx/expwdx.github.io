---
title: 通过docker搭建zookeeper
---


## 参考资料
[ZooKeeper Getting Started Guide](http://zookeeper.apache.org/doc/current/zookeeperStarted.html)  
[Mesos+Zookeeper+Marathon的Docker管理平台部署记录](https://blog.csdn.net/zhufuyi/article/details/72782350?locationNum=6&fps=1)  
[docker部署zookeeper集群](http://blog.sina.com.cn/s/blog_8ea8e9d50102wx24.html)  
[Docker环境搭建ZooKeeper集群](https://www.jianshu.com/p/14d30aa63dc9)


## 环境组网
### 组网策略
准备三台server搭建zookeeper最小化集群，以达成主备份关系，为后期mesos集群的管理和调度做准备


### 节点主机配置
储备三台服务器如下：
主机系统 | ip地址 
---|---
ubuntu16.04 server | 192.168.24.173  
ubuntu16.04 server | 192.168.24.179  
ubuntu16.04 server | 192.168.24.178

## 集群配置
### 软件配置
#### 软件版本
我们选择docker启动容器的方法来部署zookeeper，考虑到后期部署spark on mesos时版本的兼容性，这里选择zookeeper-3.4.13 。如下为集群的软件版本规划：  
软件 | 版本
--- | ---
zookeeper | 3.4.13
mesos | 1.6.1
spark | 2.2.2

#### zookeeper的docker容器选择
在参阅 [docker hub](https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=zookeeper&starCount=0) 中的众多的zookeeper容器后，选择使用人群较多，dockerfile中版本配置比较新的 [jplock/zookeeper:3.4.13](https://hub.docker.com/r/jplock/zookeeper/) 作为本次集群搭建的zookeeper容器版本。  

##### 容器中的软件及版本
软件名 | 来源 | 版本
--- | --- | ---
jdk | openjdk:8-jre-alpine |jdk8
zookeeper | http://apache.mirrors.pair.com | 3.4.13

##### dockerFile

```
FROM openjdk:8-jre-alpine

ARG MIRROR=http://apache.mirrors.pair.com
ARG VERSION=3.4.13

LABEL name="zookeeper" version=$VERSION

RUN apk add --no-cache wget bash \
    && mkdir -p /opt/zookeeper \
    && wget -q -O - $MIRROR/zookeeper/zookeeper-$VERSION/zookeeper-$VERSION.tar.gz \
      | tar -xzC /opt/zookeeper --strip-components=1 \
    && cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg \
    && mkdir -p /tmp/zookeeper

EXPOSE 2181 2888 3888

WORKDIR /opt/zookeeper

# Only checks if server is up and listening, not quorum. 
# See https://zookeeper.apache.org/doc/r3.4.13/zookeeperAdmin.html#sc_zkCommands
HEALTHCHECK CMD [ $(echo ruok | nc 127.0.0.1:2181) == "imok" ] || exit 1

VOLUME ["/opt/zookeeper/conf", "/tmp/zookeeper"]

ENTRYPOINT ["/opt/zookeeper/bin/zkServer.sh"]
CMD ["start-foreground"]
```

### 单个节点的配置
#### zoo.cfg配置

```
$ vi /conf/zoo.cfg

tickTime=2000
initLimit=5
syncLimit=2
#maxClientCnxns=60
#autopurge.snapRetainCount=3
#autopurge.purgeInterval=1

dataDir=/opt/zookeeper/data

clientPort=2181
server.1=192.168.24.173:2888:3888
server.2=192.168.24.178:2888:3888
server.3=192.168.24.179:2888:3888
```
#### myid配置
在集群每个节点的zookeeper中修改或创建./data/myid文件，并在其中写入1~255之间的值，该值即为节点编号。

```
$ mkdir data
$ vi /data/myid
1
```

### 拷贝容器配置到主机

```
$ sudo docker container cp zk178:/opt/zookeeper/conf /opt/zookeeper

$ sudo docker container cp zk178:/opt/zookeeper/data /opt/zookeeper
```


## 启动集群 
### 启动参数策略

### 启动命令

```
docker run -tid --restart=always \
    --net=host \
    -v /opt/zookeeper/data:/opt/zookeeper/data \
    -v /opt/zookeeper/logs:/opt/zookeeper/logs \
    -v /opt/zookeeper/conf:/opt/zookeeper/conf \
    --name=zk173 \
    jplock/zookeeper:3.4.13
```
### 测试
#### 测试命令：
通过如下命令测试集群是否联网正常。
```
echo stat | nc 127.0.0.1 2181
```
#### 测试结果：

```
➜  ~ echo stat | nc 127.0.0.1 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /127.0.0.1:39122[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 3
Sent: 2
Connections: 1
Outstanding: 0
Zxid: 0x10000000c
Mode: follower
Node count: 8
➜  ~ 
```

```
topiot@iots2:~$ echo stat | nc 127.0.0.1 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /127.0.0.1:55782[1](queued=0,recved=1370,sent=1371)
 /127.0.0.1:55790[0](queued=0,recved=1,sent=0)
 /127.0.0.1:55780[1](queued=0,recved=1372,sent=1374)
 /127.0.0.1:55784[1](queued=0,recved=1371,sent=1373)
 /127.0.0.1:55778[1](queued=0,recved=1372,sent=1374)

Latency min/avg/max: 0/0/81
Received: 5639
Sent: 5645
Connections: 5
Outstanding: 0
Zxid: 0x10000000c
Mode: leader
Node count: 8
Proposal sizes last/min/max: 36/36/338
topiot@iots2:~$ 
```

```
topiot@iots3:~$ echo stat | nc 127.0.0.1 2181
Zookeeper version: 3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
Clients:
 /127.0.0.1:39678[1](queued=0,recved=1390,sent=1390)
 /127.0.0.1:39680[1](queued=0,recved=1390,sent=1390)
 /127.0.0.1:39682[1](queued=0,recved=1390,sent=1390)
 /127.0.0.1:41312[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/74
Received: 4326
Sent: 4325
Connections: 4
Outstanding: 0
Zxid: 0x10000000c
Mode: follower
Node count: 8
topiot@iots3:~$ 
```


