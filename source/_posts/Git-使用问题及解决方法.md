---
title: Git 使用问题及解决方法
date: 2018-06-09 12:56:26
categories:
- 技术笔记
tags:
---
### 如何创建文件夹  
  在网页上点击新建文件，在命名框中添加`/`就变成文件夹

### 远程仓库
  - 添加
`$git remote add [name] [url]`
  - 删除
`$git remote rm [name]`

### 分支
- 删除远程分支  
`$git push origin --delete <remoteBranchName>`
- 推送本地分支，若远程不存在该分支会自动创建  
`$git push <remoteRepository> <localBranchName>`  
- 合并其他分支到当前分支  
`$git merge <branchName>(要合并的分支)`
- 强制删除本地分支  
`git branch -D <localBranchName>`

### 在 Github 上添加公钥         
1.`$ ssh -T git@github.com`检查权限  
  `Permission denied (publickey).`  

2.生成 ssh 密钥  
```
$ ssh-keygen -t rsa -C "xfffrank"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/FrankXie/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/FrankXie/.ssh/id_rsa.
Your public key（使用这个文件） has been saved in /home/FrankXie/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:ae01V2/IEOMh12ZXKSaL33Ro1cBmvHMcq0fuoQ12o+0 xfffrank
The key's randomart image is:
+---[RSA 2048]----+
|          . =+..=|
|           = *O+o|
|          . *=+++|
|         + . *o*+|
|        S o * Ooo|
|       . . o B B |
|          . . X o|
|             o + |
|              .E |
+----[SHA256]-----+
```



3.复制公钥  

```
$ cat /home/FrankXie/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCohUb8EJqFuzaNJbCL8ck044lgvjsJr8PgCNerp(省略。。。)+kX5e5Go920A1+wcgGJewKqurPHHgjYCfo9YCFbm1rP4Ne4cmpvmaL91v xfffrank
```
4.在 Github settings 上添加该公钥   

5.测试权限  

```
$ ssh -T git@github.com
Hi xfffrank! You've successfully authenticated, but GitHub does not provide shell access.
```

### 本地分支与远程分支冲突的解决办法
- 若本地仓库是远程仓库的子集，即本地包含的文件远程都有，可以先将本地分支推送到远程仓库，再通过 pull request 与 master 分支合并。

### 解决 .gitignore 文件失效
```
git rm -r --cached . #从 git 中删除所有文件记录
git add . ＃重新加入文件
git commit -m "fixed untracked files"
```

### 版本回退
```
git reset --hard [commit_id]
```
注：通过`git log`命令查看每个 commit 版本的id；`commit_id`不需要写全


