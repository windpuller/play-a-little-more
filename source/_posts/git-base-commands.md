title: git base commands
date: 2015-09-24 11:52:12
tags: git
---
#### 初始化
<code>$ mkdir test</code>
<code>$ cd test</code>
<code>$ git init</code>

#### config
<code>$ git config --global user.name username</code>
<code>$ git config --global user.email useremail.chn@gmail.com</code>

#### 文件修改 、查看状态、文件添加以及修改提交
<code>$ echo "hello world">>new.txt</code>
<code>$ git status</code>
<code>$ git add new.txt</code>
<code>$ git status</code>
提交第一个版本
<code>$ git commit -m "first"</code> 

<code>$ echo "hello new world">>new.txt</code>
<code>$ git status</code>
<code>$ git add new.txt</code>
<code>$ git status</code>
提交第二个版本
<code>$ git commit -m "second"</code>

#### 版本切换
查看更新记录
<code>$ git log</code>
使用能够区别版本的前几位即可
<code>$ git checkout commit-id</code>

#### 远程提交
添加远程仓库
<code>$ git remote add origin https://github.com/windpuller/test.git</code>
将修改提交到服务器端
<code>$ git push -u origin master</code>

#### 检出仓库
创建本地仓库的克隆版本
<code>$ git clone /path/to/repository</code>
创建远端服务器的克隆版本
<code>$ git clone uxername@host:/past/to/repository</code>

#### 分支
创建分支
<code>$ git branch branch1</code>
切换分支
<code>$ git checkout branch1</code>
删除分支
<code>$ git branch -d branch1 </code>
将分支推送到远端仓库，不推送的话分支是不为他人所见的
<code>$ git push origin branch1</code>

#### 更新与合并
更新本地仓库至最新改动，完成了获取fetch并合并merge改动的功能
<code>$ git pull</code>
合并其他分支到当前分支
<code>$ git merge branch</code>
在以上两种情况下，git都会尝试自动合并改动。但是不是每次合并都能成功，可能出现冲突。
这时候就需要修改文件来手动合并这些冲突。改完文件之后，需要执行以下命令以将它们标记为合并成功：
<code>$ git add filename</code>
在合并改动之前，可以使用如下命令预览差异：
<code>$git diff source_branch target_branch</code>

#### 标签
为某个版本创建标签
<code>$ git tag tag-name commit-ID</code>

#### 替换本地改动
如果操作失误，可以使用HEAD中的最新内容替换掉工作目录中的文件，已添加到缓存区的改动和新文件都不会受到影响
<code>$ git checkout -- filename</code>
如果想要放弃本地的所有改动和提交，可以到服务器上获取最新的版本历史，并将本地主分支指向它：
<code>$ git fetch origin</code>
<code>$ git reset --hard origin/master</code>

#### 实用小贴士
内建的图形化git
<code>$ gitk</code>
彩色的git输出
<code>$ git config color.ui true</code>
显示历史记录时，每个提交的信息只显示一行
<code>$ git config format.pretty oneline</code>
交互式添加文件到暂存区
<code>$ git add -i</code>
#### 参考文献：
1. http://rogerdudler.github.io/git-guide/index.zh.html