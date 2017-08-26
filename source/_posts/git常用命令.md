---
title: git常用命令
date: 2016-08-26 19:03:36
comments: true
categories: [other]
tags: [git]
---
# 准备工作
## 下载客户端
	https://help.github.com/articles/set-up-git
	windows 下需要安装 Git Bash命令行工具 安装完后操作同 linux
## 配置git
	$ git config --global user.name "tangyao"
	$ git config --global user.email "tangyao@taobao.com"
	** 查看设置情况 **
	$ git config --get user.name
	$ git config --get user.email
## 建立ssh链接
	创建 sshkey用于连接服务器的时候认证
	$ cd .ssh
		保证. ssh目录下名称为id_rsa, id_rsa.pub的文件是唯一的，如果已经存在的话先备份一下。
	$ mkdir back_rsa
	$ cp id_rsa* back_rsa
	$ rm id_rsa*
	$ ssh-keygen -t rsa -C "tangyao@taobao.com"
	复制id_rsa.pub中的内容添加到 github中
	登陆 github系统。点击右上角的 Account Settings--->SSH Public keys ---> add another public keys
	把你本地生成的密钥复制到里面（key文本框中）， 点击 add key 就ok了
	测试是否连接成功
	复制的key里面带空行的样子 （vi取到的可能有问题）
	$ ssh﻿ git@github.com
	echo: Hi coolme200/top! You've successfully authenticated, but GitHub does not provide shell access.
# 建立git项目
## 创建工作目录
	$ mkdir helloworld
	$ cd helloworld (到workspace的项目目录下执行)
## 初始化，否则这不会被认为是一个 git项目
	$ git init
## 设置项目远程版本库地址
	例1（适用 github）
	$ git remote add origin http://github.com/coolme200/hello
		例2（适用 gitlab）
	$ git remote add origin git@gitlab.taobao.ali.com:varworld.git
		错误提示：
		fatal: remote origin already exists.
		解决办法：
	$ git remote rm origin
	注意
		1、先输入 $ git remote rm origin
		2、再输入 $ git remote add origin git@github.com:djqiang/gitdemo.git 就不会报错了！
		3、如果输入 $ git remote rm origin 还是报错的话，error: Could not remove config section 'remote.origin'. 	我们需要修改gitconfig文件的内容
		4、找到你的github的安装路径，我的是C:\Users\ASUS\AppData\Local\GitHub\PortableGit_ca477551eeb4aea0e4ae9fcd3358bd96720bb5	c8\etc
		5、找到一个名为gitconfig的文件，打开它把里面的[remote "origin"]那一行删掉就好了！
## 获取代码
	$ git pull origin master
## 修改代码
	$ git add test.js
## 提交代码至本地版本库
	$ git commit -m 'commit'

## 提交到服务器
### 提交到master分支
	$ git push origin master
		错误提示：
		error: failed to push som refs to ........
		解决办法,先pull 下来 再push 上去
	$ git pull origin master
	 ex:1
	$ git push origin  default to pushing only the current branch to <code> origin </code> use <code> git config remote.origin.push HEAD </code>.

### 创建新的分支、推送修改到new1分支
	$ git push origin new1

# 常用命令
## 查看提交日志信息
	git log --查看版本信息

## 删除文件
	git rm 文件  --恢复

## branch
	git branch bugFix --新建分支bugFix
	git branch -d new1 删除分支
	git push origin :new1 删除服务器分支
## checkout
	git checkout bugFix --切换到bugFix分支
	git checkout -b bugFix --新建并切换分支bugFix
	git checkout master^ -- HEAD向前移动一次
	git checkout master~3 -- HEAD向前移动3个未知
	git checkout -f master HEAD~4 将master指向当前的前4个版本

## 合并分支：
	git merge bugFix --将bugFix合并到当前分钟
	git rebase master -- 将当前分支合并到master分支上
	git cherry-pick C2 C4 -- 将 C2 和 C4下的提交记录，抓过来放到当前分支下
	git rebase -i HEAD4 移动分支的提交顺序（可删除提交）

	合并工具：
	git mergetool --tool=emerge
	命令 f a b n p
	https://www.gnu.org/software/emacs/manual/html_node/emacs/Merge-Commands.html#Merge-Commands

## 拉取远程分支 fetch
	git fetch origin master
		显示分支列表，包括远程。
	git branch -a
	git fetch -p origin
		创建分支new1
## 撤销提交：
	git reset --hard 版本号  --直接撤销提交
	git revert HEAD  --生产一个新的版本，撤销到之前版本

	git log --graph --pretty=online --abbrev-commit  (查看合并的动作)

## 存储
	git stash 暂存目录
	git stash pop  恢复目录

## 引用
	git reflog show  查看所有引用
	git show HEAD@{0}

## 查看状态
	Git status

	git show 版本号  查看版本信息

## clean
	删除 untracked files
	git clean -f

	# 连 untracked 的目录也一起删掉
	git clean -fd

	# 连 gitignore 的untrack 文件/目录也一起删掉 （慎用，一般这个是用来删掉编译出来的 .o之类的文件用的）
	git clean -xfd

	# 在用上述 git clean 前，墙裂建议加上 -n 参数来先看看会删掉哪些文件，防止重要文件被误删
	git clean -nxfd
	git clean -nf
	git clean -nfd

## diff
	git diff 查看目录、索引、HEAD的差异

# 注
	mac git 工具 Sourcetree git代码比较工具

# git仓库超前的情况
	git fetch origin
	git merge origin master
	git pull origin master过程中出现一下错误
	(fatal: refusing to merge unrelated histories)[http://blog.csdn.net/lindexi_gd/article/details/52554159]:使用git pull origin master --allow-unrelated-histories

	(Dealing with non-fast-forward errors)[https://help.github.com/articles/dealing-with-non-fast-forward-errors/]


**参考链接：**
[git](https://git-scm.com/docs)

[如何在同一台电脑配置多个git或者github账号](https://tianqing370687.github.io/2016/08/29/git-%E5%A6%82%E4%BD%95%E5%9C%A8%E5%90%8C%E4%B8%80%E5%8F%B0%E7%94%B5%E8%84%91%E9%85%8D%E7%BD%AE%E5%A4%9A%E4%B8%AAgit%E6%88%96%E8%80%85github%E8%B4%A6%E5%8F%B7/)

[Git Config 命令查看配置文件](https://cnbin.github.io/blog/2015/06/19/git-config-ming-ling-cha-kan-pei-zhi-wen-jian/)

**推荐一个git学习的网站，非常形象的模拟了git的命令：**[http://learngitbranching.js.org/](http://learngitbranching.js.org/)

