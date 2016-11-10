##分支命令

1. 创建分支 `git checkout -b feature-git-learning`(创建了一个feature-git-learning分支，并切换到feature-git-learning分支上去，主要使用这个命令) (`git checkout feature-git-learning` 是切换到feature-git-learning分支上，若是feature-git-learning分支不存在，就会执行失败)
2. 显示当前分支状态 `git status`(查看当前所在的分支，也可以查看当前仓库的状态，比如说，代码是否添加到暂存中)
3. 删除本地分支 `git branch -D feature-git-learning` (删除本地的feature-git-learning分支)(需要注意，删除本地feature-git-learning分支的时候，当前不能够在feature-git-learning分支上，需要切换到其他分支，一般而言我们切换到master或者是develop分支)
4. 删除远程分支 `git push --delete origin feature-git-learning` (删除远程库中的feature-git-learning分支)
5. 抓取远程分支 `git fetch origin` (使用这个命令会把远程库中存在远程的分支拉取下来，若本地存在，便会更新本地)
6. 拉取远程分支 `git push --rebase` (使用rebase可以在远程库来下的同时解决冲突问题)(使用`git push`一般就够了)

##解决冲突
1. rebase冲突
2. merge冲突

##Git别名
