---
title: Mac中的Python环境变量操作手册
date: 2018-12-06 19:39:01
tags:
---
## 将 python3 命令加入环境变量
查看当前环境变量：
```sh
echo $PATH
```
<!-- more -->
打开`~/.bash_profile`(没有请新建)：
```sh
vi ~/.bash_profile
```
我装的是Python3.7，我的Python执行路径为：`/usr/local/Cellar/python/3.7.0/bin`。
于是写入
```sh
export PATH="/usr/local/Cellar/python/3.7.0/bin:$PATH"
```
激活脚本文件：
```sh
source ~/.bash_profile
```

如果你使用了 zsh 进行终端美化，需要在`~/.zshrc`最后一行写入：
```sh
source ~/.bash_profile   
# 写绝对路径会好一点。在 ~ 目录下使用 pwd 查看当前路径。
```

## 使用软链接改变默认 python 命令
加入环境变量之后，会发现需要打 python3 才看得到 python3，但我不想看到3，只想打 python。

终端敲：
```sh
which python3
```
我得到的是`/usr/local/Cellar/python/3.7.0/bin/python3`。

接下来就是魔法操作了。
```sh
ln -s /usr/local/Cellar/python/3.7.0/bin/python3 /usr/local/bin/python
```
重启终端即可生效。

## 使用别名改变默认 python 命令
不想使用软链接，可使用别名。

终端敲：
```sh
which python3
```
我得到的是`/usr/local/Cellar/python/3.7.0/bin/python3`。

打开`~/.bash_profile`，写入：
```sh
alias python="/usr/local/Cellar/python/3.7.0/bin/python3"
```
激活脚本：
```sh
source ~/.bash_profile
```
完毕。
