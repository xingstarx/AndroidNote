##分支命令

1. 创建分支 `git checkout -b feature-git-learning`(创建了一个feature-git-learning分支，并切换到feature-git-learning分支上去，主要使用这个命令) (`git checkout feature-git-learning` 是切换到feature-git-learning分支上，若是feature-git-learning分支不存在，就会执行失败)
2. 显示当前分支状态 `git status`(查看当前所在的分支，也可以查看当前仓库的状态，比如说，代码是否添加到暂存中)
3. 删除本地分支 `git branch -D feature-git-learning` (删除本地的feature-git-learning分支)(需要注意，删除本地feature-git-learning分支的时候，当前不能够在feature-git-learning分支上，需要切换到其他分支，一般而言我们切换到master或者是develop分支)
4. 删除远程分支 `git push --delete origin feature-git-learning` (删除远程库中的feature-git-learning分支)
5. 抓取远程分支 `git fetch origin` (使用这个命令会把远程库中存在远程的分支拉取下来，若本地存在，便会更新本地)
6. 拉取远程分支 `git pull --rebase` (使用rebase可以在远程库来下的同时解决冲突问题)(使用`git pull`一般就够了)
7. 推送分支到远程 `git push origin feature-git-learning`(推送本地当前分支到远程库中，比如说推送代码到github上)

##解决冲突
1. rebase冲突 (在我们的团队协作开发中，做需求采用的是开新分支的模式，当功能做完后，在被leader review代码的时候，被要求简化提交的commit，或者是修改不规范的代码，或者是跟主干master(或者develop)分支代码冲突。为了保证代码质量，以及commit提交的质量，需要用到rebase。rebase可以合并多条无用的commit，可以保证提交的commit形成一条树的主干结构，这也是跟merge的区别) 举例来说，现在完成了一个新功能，feature-hello-world，发现跟develop分支的代码有冲突，那就需要解决冲突，`git rebase`(直接用来解决冲突)， `git rebase -i`(可以用来合并commit，修改commit)。具体的冲突解决过程，可以在实践操作中体验，命令执行后都是有对应的英文提示的。`git rebase --continue`(当前冲突解决后，进行下一次commit的rebase过程) `git rebase --abort`(取消解决冲突，执行后，当前的rebase命令就被取消，代码恢复到rebase以前) rebase的时候，暂存区不能有内容，必须是干净的。
2. merge冲突 merge冲突相对用的少了，同样也可以用来解决冲突，命令为 `git merge feature-x` 这种方式为快进式的合并方式(不会为这次merge操作生成一个新的commit)  `git merge --no-ff feature-x`(这种会为merge操作生成一个新的commit)


##Pull Request流程
1. Pull Request流程在Github上可是非常流行的，当我们给一些出名的项目提patch，就可以采用pull request的机制。步骤如下，首先fork别人的仓库，自己就相当于复制了一份代码;接着在自己的仓库中创建一个新的feature分支;然后在开发者的项目主页点击pull request，选中自己的feature分支,生成一个pull request请求。这样就完成了，只需要库作者检查合并即可。这对协作开发而言，是非常方便的。

##Git别名

敲一个 `git rebase -i origin/develop` 感觉好费事，要敲好久。就在考虑能不能简化命令，不要敲那么多代码，用简单的几个字母代表复杂的含义,这里介绍我的Git别名

```
alias g='git'
alias ga='git add'
alias gaa='git add --all'
alias gcam='git commit -a -m'
alias gcb='git checkout -b'
alias gcm='git checkout master'
alias gcmsg='git commit -m'
alias gco='git checkout'

alias gd='git diff'
alias gf='git fetch'
alias gfo='git fetch origin'
alias gl='git pull'
alias gm='git merge'
alias gmom='git merge origin/master'
alias gmum='git merge upstream/master'
alias gp='git push'

alias gr='git remote'
alias gra='git remote add'
alias grb='git rebase'
alias grba='git rebase --abort'
alias grbc='git rebase --continue'
alias grbi='git rebase -i'
alias grbm='git rebase master'
alias grbs='git rebase --skip'

alias gss='git status -s'
alias gst='git status'
alias gsta='git stash'
alias gstaa='git stash apply'
alias gstd='git stash drop'
alias gstl='git stash list'
alias gstp='git stash pop'
```

就列举这些常用的命令吧，再介绍，平常我使用的比较多的几个别名命令

1. gaa (添加全部文件到暂存区)
2. gcam "xxx" (添加一条commit "xxx")
3. gfo (抓取远程库的分支到本地)
4. ggpull --rebase(拉取远程库的代码到本地，同时对于当前分支的代码以rebase方式进行合并，以保持一条主干的结构，而非枝繁叶茂的情况)
5. ggpush feature-hello-world(推送feature-hello-world到远程)(如果要强制推代码，可以改为ggpush --force feature-hello-world)(这篇没有提到这个的别名，有需要我在写出来)
6. grb(grbi) (用来进行rebase命令)
7. gst (用来显示当前分支，或者是当前分支的状态)

目前而言使用这7个命令，已经足够解决我95%的问题了。(是不是感觉很简洁，好用)

##配合zsh使用

##配合proxychains4使用


