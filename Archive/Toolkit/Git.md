##### 时光机
1. ` git reflog `
2. `git reset HEAD@{index}`

##### 删除历史提交记录
1. `git checkout --orphan new_branch`
2. `git add .`
3. `git commit -am "reset log"`
4. `git branch -D master`
5. `git push -f origin master`

#### git log
- `git log` 查看**提交**记录
#### git status
- `git status` 查看**未提交**文件（M：修改； A：暂存）状态
#### git diff
- `git diff` 查看**未暂存**文件相比**暂存**文件的变化
- `git diff --staged` 查看**暂存**但未提交文件相比**提交**文件的变化
- `git diff HEAD` 查看上面两者的并集


#### git branch
`git branch -f <oldbranch> <anotherbranch>/<relative reference>`  切换分支指向位置
#### git reset
`git reset HEAD^` 取消提交(不适用于远程开发)
`git revert HEAD` Create new commits which reverse the effect of earlier ones    撤销更改并分享给别人(适用于远程开发)

#### git cherry-pick
将一些提交复制到当前所在的位置（`HEAD`）下
`git cherry-pick <分支名~i>/<Hash值>`

#### git rebase
在另一个分支上重新应用来自一个分支的提交
应用：
- 先用 `git rebase -i` 将提交重新排序，然后把我们想要修改的提交记录挪到最前
- 然后用 `git commit --amend` 来进行一些小修改
- 接着再用 `git rebase -i` 来将他们调回原来的顺序

#### git describe
`git describe <ref>` 
输出：
`<tag>_<numCommits>_g<hash>`

`tag` 表示的是离 `ref` 最近的标签， `numCommits` 是表示这个 `ref` 与 `tag` 相差有多少个提交记录， `hash` 表示的是你所给定的 `ref` 所表示的提交记录哈希值的前几位。

#### git fetch
- `git fetch <remote_name> <source>:<destination>` 与git push相反，此处source为远程仓库分支（如果本地没有destination分支，则会自动创建）<font color="#ff0000">：右边不能是当前分支</font>（因为最后要执行merge，不能自己merge自己）
	- push时将<source>`<source>`**留空** 会删除本地远程分支origin/source和远程仓库`<source>`分支（空 push foo）
	- fetch时将`<source>`**留空** 会创建本地foo新分支（空 fetch foo）

#### git pull
`git pull origin <remote_branch>:<local_branch>` 
pull = fetch + merge(合并到本地HEAD指向的分支)


#### 设置远程追踪分支
1. `git checkout -b foo origin/main` 创建追踪远程main分支的本地分支foo
2. `git branch -u origin/main foo` 将本地分支foo设置为跟踪远程main

![[Pasted image 20240322172115.png]]
![[Pasted image 20240322172421.png]]
## 基础

- `git help <command>`: 获取 git 命令的帮助信息
- `git init`: 创建一个新的 git 仓库，其数据会存放在一个名为 `.git` 的目录下
- `git status`: 显示当前的仓库状态
- `git add <filename>`: 添加文件到暂存区
- `git commit`: 创建一个新的提交
    - 如何编写 [良好的提交信息](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)!
    - 为何要 [编写良好的提交信息](https://chris.beams.io/posts/git-commit/)
- `git log`: 显示历史日志
- `git log --all --graph --decorate`: 可视化历史记录（有向无环图）
- `git diff <filename>`: 显示与暂存区文件的差异
- `git diff <revision> <filename>`: 显示某个文件两个版本之间的差异
- `git checkout <revision>`: 更新 HEAD 和目前的分支

## 分支和合并

- `git branch`: 显示分支
- `git branch <name>`: 创建分支
- `git checkout -b <name>`: 创建分支并切换到该分支
    - 相当于 `git branch <name>; git checkout <name>`
- `git merge <revision>`: 合并到当前分支
- `git mergetool`: 使用工具来处理合并冲突
- `git rebase`: 将一系列补丁变基（rebase）为新的基线

## 远端操作

- `git remote`: 列出远端
- `git remote add <name> <url>`: 添加一个远端
- `git push <remote> <local branch>:<remote branch>`: 将对象传送至远端并更新远端引用
- `git branch --set-upstream-to=<remote>/<remote branch>`: 创建本地和远端分支的关联关系
- `git fetch`: 从远端获取对象/索引
- `git pull`: 相当于 `git fetch; git merge`
- `git clone`: 从远端下载仓库

## 撤销

- `git commit --amend`: 编辑提交的内容或信息
- `git reset HEAD <file>`: 恢复暂存的文件
- `git checkout -- <file>`: 丢弃修改
- `git restore`: git2.32版本后取代git reset 进行许多撤销操作

# Git 高级操作

- `git config`: Git 是一个 [高度可定制的](https://git-scm.com/docs/git-config) 工具
- `git clone --depth=1`: 浅克隆（shallow clone），不包括完整的版本历史信息
- `git add -p`: 交互式暂存
- `git rebase -i`: 交互式变基
- `git blame`: 查看最后修改某行的人
- `git stash`: 暂时移除工作目录下的修改内容
- `git bisect`: 通过二分查找搜索历史记录
- `.gitignore`: [指定](https://git-scm.com/docs/gitignore) 故意不追踪的文件