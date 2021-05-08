---
title: docker安装
date: 2021-05-07 20:17:50
categories: 
  - docker
tags:
  - docker
  - docker install
---

### 1.安装依赖包

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 2.设置阿里云镜像源

```shell
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 3.安装 Docker-CE

- 重建 Yum 缓存。

- 安装 Docker-CE ，请执行一下命令进行安装：

```shell
sudo yum install docker-ce
```

### 4.启动 Docker-CE
```shell
sudo systemctl enable dockersudo systemctl start docker
```
### 5.[可选]为 Docker 建立用户组
docker 命令与 Docker 引擎通讯之间通过 UnixSocket ，但是能够有权限访问 UnixSocket 的用户只有 root 和 docker 用户组的用户才能够进行访问，所以我们需要建立一个 docker 用户组，并且将需要访问 docker 的用户添加到这一个用户组当中来。
- 建立 Docker 用户组
```shell
sudo groupadd docker
```
- 添加当前用户到 docker 组
```shell
sudo usermod -aG docker $USER
```
- 刷新用户组
```shell
sudo newgrp docker
```

参考：https://www.cnblogs.com/myzony/p/9071210.html