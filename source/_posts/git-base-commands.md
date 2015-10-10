title: git base commands
date: 2015-09-24 11:52:12
tags: git
---
#### 初始化
	$ mkdir test
	$ cd test
	$ git init

#### config
	$ git config --global user.name username
	$ git config --global user.email useremail.chn@gmail.com

#### 文件修改 、查看状态、文件添加以及修改提交
	$ echo "hello world">>new.txt
	$ git status
	$ git add new.txt
	$ git status
提交第一个版本
	$ git commit -m "first" 

	$ echo "hello new world">>new.txt
	$ git status
	$ git add new.txt
	$ git status
提交第二个版本
	$ git commit -m "second" 

#### 版本切换
查看更新记录
	$ git log 
使用能够区别版本的前几位即可
	$ git checkout commit-id

#### 远程提交
添加远程仓库
	$ git remote add origin https://github.com/windpuller/test.git 
将修改提交到服务器端
	$ git push -u origin master 

#### 检出仓库
创建本地仓库的克隆版本
	$ git clone /path/to/repository 
创建远端服务器的克隆版本
	$ git clone uxername@host:/past/to/repository 

#### 分支
创建分支
	$ git branch branch1 
切换分支
	$ git checkout branch1 
删除分支
	$ git branch -d branch1 
将分支推送到远端仓库，不推送的话分支是不为他人所见的
	$ git push origin branch1

#### 更新与合并
更新本地仓库至最新改动，完成了获取fetch并合并merge改动的功能
	$ git pull
合并其他分支到当前分支
	$ git merge branch
在以上两种情况下，git都会尝试自动合并改动。但是不是每次合并都能成功，可能出现冲突。
这时候就需要修改文件来手动合并这些冲突。改完文件之后，需要执行以下命令以将它们标记为合并成功：
	$ git add filename
在合并改动之前，可以使用如下命令预览差异：
	$ git diff source_branch target_branch

#### 标签
为某个版本创建标签
	$ git tag tag-name commit-ID

#### 替换本地改动
如果操作失误，可以使用HEAD中的最新内容替换掉工作目录中的文件，已添加到缓存区的改动和新文件都不会受到影响
	$ git checkout -- filename
如果想要放弃本地的所有改动和提交，可以到服务器上获取最新的版本历史，并将本地主分支指向它：
	$ git fetch origin
	$ git reset --hard origin/master 

#### 实用小贴士
内建的图形化git
	$ gitk 
彩色的git输出
	$ git config color.ui true 
显示历史记录时，每个提交的信息只显示一行
	$ git config format.pretty oneline 
交互式添加文件到暂存区
	$ git add -i 
#### 参考文献：
1. http://rogerdudler.github.io/git-guide/index.zh.html