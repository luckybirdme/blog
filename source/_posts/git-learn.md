---
title: Git 简介
date: 2020-04-11 08:16:38
tags: Git
---
> Git

<!-- more -->

## 一. Git 介绍
##### 1. Git
Git 是一个开源的分布式版本控制系统，用于管理文件版本，主要是代码文件版本。

GitHub 是一个基于 Git 系统实现的，面向开源及私有软件项目的，在线代码托管平台。


##### 2. SVN
SVN 是集中式管理的版本控制器，而 Git 是分布式，这是两者最核心的区别。

SVN 版本库是集中存放在中央服务器的，首先从中央服务器取得最新的版本，然后进行文件修改，再把文件推送给中央服务器。所以工作时对网络有要求。而且中央服务器出现问题，将会出现灾难性影响。

##### 3. 特点
Git 分布式版本控制系统根本没有明确“中央服务器”，每个人的电脑上都是一个完整的版本库，可以查看历史记录，创建分支，修改文件后先提交到本地版本库，即使断网也不影响以上操作。
Git 分布式版本控制系统通常也有一台充当“中央服务器”的电脑，但这个服务器的作用仅仅是用来方便“交换”大家的修改，没有它大家也一样干活，只是交换修改不方便而已。一般是最后才一次性提交到“中央服务器”进行“交换”修改，而且即使“中央服务器”出现问题，本地版本库依然是完整，不会出现灾难性影响。

![](http://qiniucdn.luckybird.me/blog/img/2020/Git-SVN.png)


## 二. Git 框架

![](http://qiniucdn.luckybird.me/blog/img/2020/Git-main.png)

##### 1. 工作区域
- Workspace： 工作区，就是你平时存放项目代码的地方

- Index / Stage： 暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息

- Repository： 仓库区（或版本库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本

- Remote： 远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换

##### 3. 工作流程
- 在工作目录中添加、修改文件；
- 将需要进行版本管理的文件放入暂存区域；
- 将暂存区域的文件提交到git仓库。


## 三. Git 命令

- 命令把这个目录变成Git可以管理的仓库
```sh
[root@VM_133_231_centos git_home]# git init
Initialized empty Git repository in /root/git_home/.git/
```
- 克隆Git远程仓库到本地仓库
```sh
git clone https://www.github.com/your_project
```

- 添加文件到Git本地仓库
```sh
[root@VM_133_231_centos git_home]# touch readme.md
[root@VM_133_231_centos git_home]# git add readme.md
```

- 查看文件状态
```sh
[root@VM_133_231_centos git_home]# git status -s
 M readme.md
```

- 对比文件变化
```sh
[root@VM_133_231_centos git_home]# git diff readme.md
diff --git a/readme.md b/readme.md
index 557db03..61cac29 100644
--- a/readme.md
+++ b/readme.md
@@ -1 +1,2 @@
 Hello World
+This is a readme file

```


- 提交文件到Git本地仓库
```sh
# 指定文件
[root@VM_133_231_centos git_home]# git commit readme.md
[master b5332d6] commit readme.md
 1 file changed, 1 insertion(+)
# 全部提交
[root@VM_133_231_centos git_home]# git commit -a
[master 69a57d6] Tell me why
 1 file changed, 1 insertion(+)

```


- 查看提交记录
```sh
[root@VM_133_231_centos git_home]# git log
commit 486d21750ecc9439847e37e8f6a21ac0280b4bde
Author: luckybirdme <wwwluckybirdme@163.com>
Date:   Sun Apr 12 10:21:34 2020 +0800

    modified readme.md

commit a4d5ffc5a76cb9b476f5f7fef52dcc37c9ef544d
Author: luckybirdme <wwwluckybirdme@163.com>
Date:   Sun Apr 12 10:20:01 2020 +0800

    add readme.md

```

- 回滚到指定版本
```sh
[root@VM_133_231_centos git_home]# git reset --hard a4d5ffc5a76cb9b476f5f7fef52dcc37c9ef544d
HEAD is now at a4d5ffc add readme.md

```

- 提交Git本地仓库变化到远程仓库
```sh
git push 
```

- 取回GIt远程仓库的变化到本地仓库，但不与本地工作区域分支合并
```sh
git fetch
```

- 切换到指定版本分支
```sh
# 创建分支，并切换到新分支
[root@VM_133_231_centos git_home]# git checkout -b panda
Switched to a new branch 'panda'
# 查看分支
[root@VM_133_231_centos git_home]# git branch
  master
* panda
# 切换分支
[root@VM_133_231_centos git_home]# git checkout master
Switched to branch 'master'
[root@VM_133_231_centos git_home]# git branch
* master
  panda

```

- 取回GIt远程仓库的变化到本地仓库，并且与本地工作区域分支合并
```sh
git pull
```

- 查看配置
```sh
git config --list
```
- 设置参数
```sh
#初次commit之前，需要配置用户邮箱及用户名，使用以下命令：
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
