---
title: 在Linux上部署shadowsocks服务
date: 2018-08-07 22:38:28
tags:
categories:
- 技术笔记
---
### 安装 pip
[适用于 CentOS/Fedora/RedHat/OpenSUSE]
```bash
sudo yum install python-setuptools && easy_install pip
```
[适用于 Debian/Ubuntu]
```bash
sudo apt-get install python-pip
```
<!-- more -->
### 安装 shadowsocks 
```python
pip install shadowsocks
```

### 编写配置文件
随便找个位置创建配置文件：`vim /etc/shadowsocks.json`
```json
{
    "server": "YOUR_SERVER_IP",
    "server_port": SERVER_PORT,
    "local_address": "127.0.0.1",
    "local_port": LOCAL_PORT,
    "password": "YOUR_PASSWORD",
    "timeout": 300,
    "method": "aes-256-cfb",
    "fast_open": false
}
```
`YOUR_SERVER_IP`为本服务器 ip，`SERVER_PORT`为服务器端口，`local_port`为客户端转发端口。

### 添加服务器的开放端口
1. 添加端口
```bash
firewall-cmd --zone=public --add-port=10086/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）
```
2. 重启防火墙服务
```bash
systemctl restart firewalld.service
```
3. 检查开放端口列表
```bash
firewall-cmd --zone=public --list-ports
```

### 启动 shadowsocks 服务
```bash
ssserver -c /etc/shadowsocks.json -d start
```
若想停用服务，则输入
```bash
ssserver -c /etc/shadowsocks.json -d stop
```

### 下载客户端
在 GitHub 下载 shadowsocks 客户端后即可使用。

### 参考资料
> https://bitmingw.com/2015/04/03/install-shadowsocks-server/

> https://blog.csdn.net/qq_26563299/article/details/79031685