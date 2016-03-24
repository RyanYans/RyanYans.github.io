---
layout:     post
title:      "菜鸟谈谈 Git & Github!"
subtitle:   " \"快速了解Github,使用git进行简单操作。\""
date:       2016-03-24 00:30:00
author:     "RyanYans"
header-img: "img/post-bg-git.jpg"
catalog: true
tags:
    - Git
    - Github
---

-------------------------

###### 前几天到图书馆借了本书---《Github入门与实践》，看着挺简单的，借回来就没翻几页。还自傲的认为这玩意半天都能摸透玩熟，殊不知。。。
![Github之旅](http://www.embeddedlinux.org.cn/uploads/allimg/141011/1107470.png)

## 开始我的Git之旅

* Git是什么？
 
 	Git是目前世界上最先进的分布式版本控制系统（没有之一）。  
 
* Git有什么特点？ 
 
	\# 用Git管理文件的话，更新的历史会保存在Git，所以不需要备份文件啦。非常方便吧！  
	\# 你知道吗？几乎所有很火的开源软件都能在github上面找到源代码。  
	\# 总的来说github就是一个巨大的宝库，去上面看看能让我们能紧跟时代潮流，通过查看源代码以及贡献码(pull request)来实现学习与提高。这么高逼格的东西，良心小猿会私藏吗？当然会无私的与你们分享。 
 
### 1. Git入门
使用Git前，需要先建立一个仓库(repository)。您可以使用一个已经存在的目录作为Git仓库或创建一个空目录。使用您当前目录作为Git仓库，我们只需使它初始化：

	 $ git init

或者先转到要存放的目录：

	$ git init ./newrepo

从现在开始，我们将假设您在Git仓库根目录下，除非另有说明。


#### * 添加新文件  


我们有一个仓库，但什么也没有，可以使用add命令添加文件。

	 $ git add filename

此时可以继续使用add... 继续添加任务文件。  
当然也有批处理方式，来到目录：

	 $ git add .

#### * 提交版本

	 $ git commit -m "Adding files because..."

-m 后面来让你写自己的版本注释信息。（强烈建议大家都写上，因为输入说明对自己对别人阅读都很重要！）

### 2. Git与Github协作

* 开始前的准备
	1. 创建账号(不做累赘)
	2. 设置SSH Key:  

			$ ssh-keygen -t rsa -C "your_email@example.com"  

遇到提示按回车，接着输入密码...  
然后就会看到fingerprint值和一只<草泥马(忘了是羊还是马勒)>

		接着输入：  

				$ cat ~/.ssh/id_rsa.pub

按下回车！就会看到公开密钥内容。整体复制到github的setting—SSH Keys上。

	3. 在github上创建版本New repository:  
	
		创建成功后在版本页面会有URL---这就是仓库号啦！
	
	当然此时可以Clone该版本在你主机上。  
	git-bash下:	

		$ git clone URL

以后就是大象放到冰箱了(三部走)：

	$ git add  
	$ git commit -m "Your View"
	$ git push  

酱紫就把项目上传至线上仓库(Github)啦！~很方便吧。  
还有很多细节好玩的东西等你去挖掘，我在这就不多说了。

###### 附上Github 简明教程：http://www.w3cschool.cc/w3cnote/git-guide.html

********************  

### Git基本常用命令如下：

* mkdir：         XX (创建一个空目录 XX指目录名)

* pwd：          显示当前目录的路径。

* git init          把当前的目录变成可以管理的git仓库，生成隐藏.git文件。

* git add XX       把xx文件添加到暂存区去。

* git commit –m “XX”  提交文件 –m 后面的是注释。

* git status        查看仓库状态

* git diff  XX      查看XX文件修改了那些内容

* git log          查看历史记录

* git reset  --hard HEAD^ 或者 git reset  --hard HEAD~ 回退到上一个版本

　　(如果想回退到100个版本，使用git reset –hard HEAD~100 )

* cat XX         查看XX文件内容

* git reflog       查看历史记录的版本号id

* git checkout -- XX  把XX文件在工作区的修改全部撤销。

* git rm XX          删除XX文件

* git remote add origin https://github.com/tugenhua0707/testgit 关联一个远程库

* git push –u(第一次要用-u 以后不需要) origin master 把当前master分支推送到远程库

* git clone https://github.com/tugenhua0707/testgit  从远程库中克隆

* git checkout –b dev  创建dev分支 并切换到dev分支上

* git branch  查看当前所有的分支

* git checkout master 切换回master分支

* git merge dev    在当前的分支上合并dev分支

* git branch –d dev 删除dev分支

* git branch name  创建分支

* git stash 把当前的工作隐藏起来 等以后恢复现场后继续工作

* git stash list 查看所有被隐藏的文件列表

* git stash apply 恢复被隐藏的文件，但是内容不删除

* git stash drop 删除文件

* git stash pop 恢复文件的同时 也删除文件

* git remote 查看远程库的信息

* git remote –v 查看远程库的详细信息

* git push origin master  Git会把master分支推送到远程库对应的远程分支上  


---

---
