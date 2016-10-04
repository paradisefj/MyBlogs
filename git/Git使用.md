#Git使用

1. git init
2. git add 将文件添加进到仓库
3. git commit 将文件提交到仓库
4. git status 查看仓库当前状态
5. git diff 查看版本间的不同
6. git reflog 查看命令历史
7. git checkout — file 丢弃工作区的修改
   - 文件修改后还没有添加到暂存区，撤销修改后回到和版本库一模一样的状态
   - 文件已经添加到暂存区，撤销修改后回到暂存区的状态
8. git rm 删除文件
9. 远程仓库
   -  git remote add origin 远程仓库地址
      origin 为远程仓库的名称
   - git push origin master  将本地修改推送到远程库
      master 分支名称
   - git clone 远程仓库地址
      克隆远程仓库
10. 分支管理
	- git branch <name>
		创建分支
    - git checkout <name>
    	切换到分支
    - git checkout -b <name>
    	创建并且切换分支
    - git branch
    	查看分支
    - git merge <name>
    	将分支合并到当前分支
    - git branch -r <name>
    	删除分支
    - git log --graph
    	查看分支合并图