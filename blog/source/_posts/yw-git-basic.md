---
title: 运维 Git日常笔记
date: 2017-12-14 22:10:05
tags:
  - Git
categories:
  - 运维
---
#### 常用命令
```
git push origin master		# 将本地origin提交到远端 
git push origin master -f       # 将本地origin 强制覆盖远端
git pull --rebase upstream master# 将upstream master 和本地master合并到本地
git checkout -b xxx   		# 创建并切换到xxx分支;相当于（git branch dev；git checkout dev）
git checkout xxx      		# 切换到xxx分支
git checkout -- xxx   		# 撤销xxx文件的修改内容
git branch            		# 查看本地分支
git branch -d xxx     		# 删除xxx分支
git branch -D xxx     		# 强制删除xxx分支
git status    	      		# 查看文件是否被修改过
git diff      	      		# 查看文件修改
git merge dev         		# 将dev分支的工作成果合并到master分支上，git merge命令用于合并指定分支到当前分支(Fast-forward 说明这次合并是"快进模式")
 
##### git clone #####
git clone git@github.com:ttxsgoto/studypy.git     	# 克隆为origin名称
git clone -o abc git@github.com:ttxsgoto/studypy.git    # -o 指定 克隆出来的名称为abc
git clone git@github.com:ttxsgoto/studypy.git  xxx  	# 将远端的版本克隆下来到 xxx文件下
 
##### git remote #####
git remote -v  		   	# 查看主机名称和远端的地址
git remote show xxx(主机名) 	# 查看xxx的具体相关信息
git remote add name url    	# 添加仓库名称
git remote rm name    	   	# 删除本地仓库名称
git remote rename old new  	# 修改本地仓库名称
 
##### git fetch #####
git fetch    			# 将远程所有的更新取回本地
git fetch 远程主机名    	 	# 将某个远程主机的更新，全部取回本地
git fetch 远程主机名    分支名	# 将远程主机的分支，取回本地
 
##### git pull #####
git pull <远程主机名> <远程分支名>:<本地分支名>
git pull origin master  	# 远程主机的master分支和当前所在分支合并
git pull origin next:master     # 取回origin主机的next分支和本地master分支合并
git pull --rebase <remote>  <master>:<master>    # 合并采用--rebase模式合并
 
##### git push #####
git push <远程主机名> <本地分支名>:<远程分支名>
git push origin master    	# 将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建
git push origin :master    	# 删除远程分支名master
git push -u origin master    	# -u 选项指定一个默认主机，这样后面就可以不加任何参数使用
git push --all origin           # 无论是否存在对应的远程分支，将本地的所有分支都推送到远程主机
git push --force origin         # 远程主机版本比本地版本新，推送时会报错，需要先git pull合并差异在推送，也可以强制推送，添加--force参数
git push origin --tags          # git push 不会推送标签(tag),可以使用--tags选项，来推送标签
 
##### git stash #####
git stash  			# 将工作暂存起来，进行其他操作
git stash pop  			# 将栈中的信息重新打开
git stash list  		# 列出栈中的信息
 
##### git log #####
git log --graph --pretty=oneline --abbrev-commit  # 查看分支合并图
git log --pretty=oneline
 
##### git reset #####
git log   			# 查看详细日志，有时间点
git reflog 			# 查看命令历史，以便确定要回到未来的哪个版本
git reset --hard 83ec811628a08  # 回到相对应的版本，需要知道版本号，版本号没必要写全，前几位就可以

```

#### git 检查修改
1. 已修改，未暂存, 查看更改内容--- git diff   内容修改后，没有使用 git add xxx 添加到缓存区
2. 已暂存，未提交, 查看更改内容--- git diff --cached  使用git add xxx已添加到缓存区
3. 已提交，未推送,  查看更改内容--- git diff master origin/master 使用git commit 提交到本地仓库
    -- 这里master就是你的本地仓库，而origin/master就是你的远程仓库
    -- master是主分支的意思,origin就代表远程
    -- 比较本地master和origin的master区别

#### git 撤销修改
1. 已修改，未暂存,  撤销修改内容--- git reset --hard 或者(git checkout xxx) 内容修改后，没有使用 git add xxx 添加到缓存区
2. 已暂存，未提交,  撤销修改内容--- git reset --hard 或者(git reset 然后执行 git checkout xxx)  使用git add xxx已添加到缓存区
3. 已提交，未推送,  撤销修改内容--- git reset --hard origin/master  使用git commit 提交到本地仓库
    -- git reset --hard origin/master 远程仓库把代码取回来
4. 已推送到远程仓库, 撤销修改内容,回到上一个版本 --- git reset --hard HEAD^ 然后git push origin master -f 
    --  回滚到上一个版本
    --  将本地分支，强制覆盖origin maser

