---
title: Hexo博客备份
date: 2018-08-12 11:22:26
tags:
categories:
- 技术笔记
---

#### 编辑 .gitignore 文件：
在 blog 文件夹下编辑`.gitignore`如下
```bash
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
themes/   # 主题文件夹
```
<!-- more-->
其实 blog 文件夹中自带`.gitignore`，我只是在最后添加了一行`themes/`，因为该目录的文件数太多，同时也没必要备份，比如我用的是 NexT 主题，后续恢复时再重新从 [NexT](https://github.com/iissnan/hexo-theme-next) 仓库 clone 一份就可以了。当然默认设置下我们的主题配置文件是在`themes/`目录下的，这个问题下文会提到解决方法。

#### 迁移主题配置文件
在 source 文件夹下新建`_data`目录，复制`themes/next`目录下的主题配置文件`_config.yml`到`_data`目录，重命名为`next.yml`，编辑`next.yml`文件，作如下修改：
```bash
override: true   # 覆盖 NexT 的主题配置文件
```
* 备注：
    1. 此操作在平时就可以完成，这样就避免了每次拉取新的 NexT 版本都要重新修改`_config.yml`。
    2. 不需要备份整个`themes`目录。

#### 在 blog 文件夹下执行以下命令
```bash
git init  # 初始化 git 仓库
git checkout -b hexo  # 新建一个 hexo 分支用于备份
git add .  # 添加所有文件，自动忽略 .gitignore 中列出的文件
git status  # 检查添加到 git 暂存区的文件是否正确
git commit -m "Hexo备份"  # 提交修改
git remote add git@github.com:xfffrank/xfffrank.github.io.git  # 添加远程仓库，注意替换成自己的 git 仓库
git push origin hexo  # 推送到远程 hexo 分支
```

#### 恢复
1. 克隆 hexo 分支到本地
```bash
# 注意替换成自己的 git 仓库
git clone -b hexo git@github.com:xfffrank/xfffrank.github.io.git
```
2. 安装 hexo
```bash
npm install -g hexo-cli 
```

#### 参考资料
> https://shuzang.github.io/blog/2018/04/17/Hexo%E4%B9%8B%E9%A1%B9%E7%9B%AE%E6%81%A2%E5%A4%8D/
> https://github.com/iissnan/hexo-theme-next/issues/328