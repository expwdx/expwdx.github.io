---
title: docker配合gitbook部署文档
---

#### 安装gitbook
```
cnpm install -g gitbook-cli
```

#### 构建目录结构
```
nginx-docker
    --default.conf
    --docker-compose.yml
    --web
        ---note
```
#### 拉取文档仓库并编译
```
git pull && rm -rf _book && gitbook install && gitbook build .
```
#### 编写nginx服务配置
```
➜  nginx-docker cat default.conf 
server {
  listen 80;
  server_name xxx.xxx.xx.xxx;
  root /web;

  location /note {
    alias /web/note/_book;
    index index.html;
  }

  location  ^~ gitbook/ {
    root /web/gitbook/;
   }
}
```
#### 通过docker启动
编写`docker-compse.yml`
```
➜  nginx-docker cat docker-compose.yml 
version: '2'

services:
  nginx:
    image: nginx:latest
    restart: always
    container_name: nginx-web
    network_mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./web:/web
      - ./default.conf:/etc/nginx/conf.d/default.conf
```

#### 问题处理
当执行`gitbook install`非常慢的时候，执行如下命令，变更npm安装源。
```
sudo npm config set registry=http://registry.npm.taobao.org
```
