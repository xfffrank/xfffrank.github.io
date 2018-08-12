---
title: Docker入门指南
date: 2018-07-18 19:26:47
tags:
categories:
- 技术笔记
---

*本文以使用 ubuntu 为例*

### 搜索镜像
```bash
docker search ubuntu
```
找到标了 OFFICIAL 的镜像对应的名称 ubuntu。
<!-- more -->
### 下载镜像并建立容器
```bash
docker run --name MyUbuntu -dt ubuntu
```
* 说明：
    * `--name`：指定容器的名字；
    * `-d`：`d`etach，建立容器后，即脱离当前进程；
    * `-t`：`t`erminal，默认执行`/bin/bash`进程，为了让容器启动后不会立即停止。

### 查看 docker 进程
```bash
docker ps
```
* 说明：
    * `ps`：`p`resent `s`tate，显示正在执行的容器。
    * 在 docker 中，一个容器对应一个进程，进程终止，容器也就释放，停止执行。

查看所有容器，包括运行中的和未运行的：
```bash
docker ps -a
```


### 进入 Ubuntu
```bash
docker exec -it MyUbuntu bash
```
* 说明：
    * `exec`使得容器在退出后依然在后台运行，下次打开前不需要重新`start`
    * `-i`：`i`nterative，可对 terminal 输入资料
    * `-t`：`t`erminal，可对 terminal 显示资料

* 第一次使用 ubuntu 镜像时，必须先执行命令`apt-get update`来同步最新的软件源索引，否则安装许多软件时会提示未找到该package。

* 如需在 ubuntu 中执行`iptables`命令，在建立容器时，需指定`--privileged=true`来赋予容器更多权限：
    ```bash
    docker run --name MyUbuntu --privileged=true -dt ubuntu
    ```

### 退出 Ubuntu
```bash
exit
```

### 停止容器进程
```bash
docker stop MyUbuntu
```
可以再次使用`docker ps`查看进程状态

### 启动容器
```bash
docker start MyUbuntu
```

### 删除容器
```bash
docker rm MyUbuntu
```
*删除之前，需先终止容器进程*

### 容器重命名
```bash
docker rename MyUbuntu MyUbuntu2
```

### 显示已下载的所有镜像
```bash
docker images
```

### 删除 ubuntu 镜像
```bash
docker rmi ubuntu
```
* 说明：
    * `rm`：删除；
    * `i`：`i`mage镜像。

参考资料：
> https://blog.csdn.net/dongdong9223/article/details/52998375
> http://oomusou.io/docker/ubuntu/
