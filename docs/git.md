# git

## 其他操作

1. [权限/系统等问题导致的diff](https://www.jianshu.com/p/3b8ba804c47b)：

```bash
# 具体命令可以使用的时候查，可以参考下面的

# 当文件夹更改权限（以下命令）或者更改系统时候，git可能会出现很多diff
sudo chmod -R 777 file
sudo chown -R <username>:<username> file
# 若不想管理权限导致的diff可以用下面这条命令
git config --add core.filemode false
# 系统换行导致的diff可以使用下面命令取消
git config --global core.autocrlf true
```

1. git log设置

```bash
# 在~/.bashrc中添加以下内容
alias git-log='git log --pretty=oneline --all --graph --abbrev-commit'
# 然后执行source ~/.bashrc生效
# -all：显示所有分支
# -pretty=oneline：将提交信息显示为一行
# -abbrev-commit：是的输出的commit更简短
# -graph：以图的形式显示
```

## 仓库相关

1. 查看远端仓库：`git remote -v`
2. 重命名仓库：`git remote rename <old_remote_name> <new_remote_name>`（一般默认都叫`origin`）
3. 新建关联远端仓库：`git remote add <another_remote_name> <another_remote_url>`
4. 删除远端仓库关联：`git remote remove <remote_name>`
5. 查看本地仓库所有记录：`git reflog`（记录了本地仓库中 HEAD 和分支的移动记录，包括提交、合并、分支创建和删除等操作）

## 分支相关

1. 查看远端所有分支：`git branch -r`
2. 查看所有分支：`git branch -a`
3. 查看本地分支与远端分支的追踪关系：`git branch -vv`（只能追踪一个，所以如果有多个远端仓库关联，也只显示一个）
4. 设置本地分支与远端分支关联追踪：`git branch --set-upstream-to=<remote_name>/<remote_branch_name> <local_branch_name>`
5. 删除本地分支：`git branch -d <branch_name>`
6. 删除远端分支：`git push <remote_name> --delete <branch_name>`
7. 重命名本地分支：`git branch -m <old_branch_name> <new_branch_name>`
8. 重命名远端分支：先删除远端分支，重命名本地分支，再推送本地分支
9. 推送分支到指定远端仓库：`git push <remote_name> <local_branch_name>`
10. 推送分支到所有远端仓库：`git push --all`（好像不好使？）
11. 获取追踪的远端分支信息：`git fetch`
12. 获取所有远端仓库的所有分支信息：`git fetch --all`
13. 拉取远端分支：`git pull <remote_name> <remote-branch-name>:<local-branch-name>`
14. 恢复删除的分支：
    1. 用`git reflog`查找要恢复的分支的hash
    2. 然后`git checkout -b <branch-name> <reflog-hash>`
15. 回退：
    1. 回退并保留更改：`git reset <hash>`
    2. 回退并清空暂存区和工作区：`git reset --hard <hash>`

## rebase和merge应用示例

1. 查看分支的公共祖先：`git merge-base <branch_1> <branch_2> [<branch_n>]`
2. 查看分支在某节点之后的提交次数：`git rev-list --count <commit-hash>..<branch_name>`

> 一般我用的方法是，在开发分支dev合并到主分支main时候： \
> 在dev分支进行自己的rebase
>
> 在dev分支merge最新的main分支
>
> 在main分支merge解决冲突后的dev分支

### 1. main分支和dev分支同时开发

初始化仓库，并从main分支checkout出dev分支，并与远端仓库同步，git log结果如下

```bash
git-log
* 2df8c24 (HEAD -> dev, origin/main, origin/dev, main) [zheng] 初始化仓库
```

dev分支第一次开发，main分支第一次开发，并与远端仓库同步，git log结果如下：

```bash
git-log
* b8056d4 (HEAD -> main, origin/main) [zheng] main分支第一次开发
| * 1b8cde2 (origin/dev, dev) [zheng] dev分支第一次开发
|/
* 2df8c24 [zheng] 初始化仓库
```

main分支第二次开发，dev分支第二次开发，dev分支第三次开发，并与远端仓库同步，git log结果如下：

```bash
git-log
* 375e55d (HEAD -> dev, origin/dev) [zheng] dev分支第三次开发
* 3c84934 [zheng] dev分支第二次开发
* 1b8cde2 [zheng] dev分支第一次开发
| * 7f080da (origin/main, main) [zheng] main分支第二次开发
| * b8056d4 [zheng] main分支第一次开发
|/
* 2df8c24 [zheng] 初始化仓库
```

### 2. 将dev分支自己的记录rebase

在merge main分支之前，将dev分支进行rebase以精简提交记录（做不做都行，提交太多的话可以做一下）

可以先查看dev分支跟main分支的公共祖先节点：`git merge-base main dev`，结果如下：

```bash
git merge-base main dev
2df8c24e150037461618c1fa217ad665805ee3fc
```

接着查看dev分支从该节点checkout出来之后进行了几次提交，命令及结果如下：

```bash
# 查看有哪些提交
git rev-list 2df8c24e150037461618c1fa217ad665805ee3fc..dev
375e55d6e306288a3bf7c8369217a8c1ee53c753
3c8493453bb095e7fdfff588863b8c743af2f528
1b8cde2e799342c9ddd9b00223825e8ff739297d
# 加上--count查看有几次提交
git rev-list --count 2df8c24e150037461618c1fa217ad665805ee3fc..dev
3
```

将这几次提交用rebase合并成一次提交：`git rebase -i HEAD~3`，会弹出来下面的编辑器：

```bash
pick 1b8cde2 [zheng] dev分支第一次开发
pick 3c84934 [zheng] dev分支第二次开发
pick 375e55d [zheng] dev分支第三次开发

# Rebase 2df8c24..375e55d onto 2df8c24 (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified); use -c <commit> to reword the commit message
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
```

可以直接将后面的两次提交合并到第一次提交（将后面两个pick改成squash即可）

保存并关闭后会弹出另一个编辑框：

```bash
# This is a combination of 3 commits.
# This is the 1st commit message:

[zheng] dev分支第一次开发

# This is the commit message #2:

[zheng] dev分支第二次开发

# This is the commit message #3:

[zheng] dev分支第三次开发

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Fri Jun 21 15:18:15 2024 +0800
#
# interactive rebase in progress; onto 2df8c24
# Last commands done (3 commands done):
#    squash 3c84934 [zheng] dev分支第二次开发
#    squash 375e55d [zheng] dev分支第三次开发
# No commands remaining.
# You are currently rebasing branch 'dev' on '2df8c24'.
#
# Changes to be committed:
#	modified:   README.md
#
```

这边就是rebase之后要用什么信息进行commit，我改成下面这样：

```bash
# This is a combination of 3 commits.
# This is the 1st commit message:

# [zheng] dev分支第一次开发

# This is the commit message #2:

# [zheng] dev分支第二次开发

# This is the commit message #3:

# [zheng] dev分支第三次开发

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Fri Jun 21 15:18:15 2024 +0800
#
# interactive rebase in progress; onto 2df8c24
# Last commands done (3 commands done):
#    squash 3c84934 [zheng] dev分支第二次开发
#    squash 375e55d [zheng] dev分支第三次开发
# No commands remaining.
# You are currently rebasing branch 'dev' on '2df8c24'.
#
# Changes to be committed:
#	modified:   README.md
#
[zheng] dev分支所有开发合并，待merge
```

保存并关闭后，查看git log：

```bash
git-log
* df63990 (HEAD -> dev) [zheng] dev分支所有开发合并，待merge
| * 375e55d (origin/dev) [zheng] dev分支第三次开发
| * 3c84934 [zheng] dev分支第二次开发
| * 1b8cde2 [zheng] dev分支第一次开发
|/
| * 7f080da (origin/main, main) [zheng] main分支第二次开发
| * b8056d4 [zheng] main分支第一次开发
|/
* 2df8c24 [zheng] 初始化仓库
```

强制推送到远端后的git log（因为rebase后跟远端的提交记录不一样了，dev只能强制推送）：

```bash
git-log
* cf3c5ab (HEAD -> dev, origin/dev) [zheng] dev分支所有开发合并，待merge
| * 7f080da (origin/main, main) [zheng] main分支第二次开发
| * b8056d4 [zheng] main分支第一次开发
|/
* 2df8c24 [zheng] 初始化仓库
```

### 3. 采用merge的方式合并（推荐）

#### 3. 1. 将main分支merge到dev分支

先将main分支最新修改进行拉取（多人开发的话，每次都要拉一下保持最新）：`git fetch origin main`

将main分支合并到dev分支：`git merge main`

处理完冲突之后，进行提交并推送。此时git log如下：

```bash
git-log
*   44a9c7c (HEAD -> dev, origin/dev) [zheng] dev merge main
|\\
| * 7f080da (origin/main, main) [zheng] main分支第二次开发
| * b8056d4 [zheng] main分支第一次开发
* | cf3c5ab [zheng] dev分支所有开发合并，待merge
|/
* 2df8c24 [zheng] 初始化仓库
```

#### 3.2. 将dev分支merge到主分支

切换到main分支，并将dev分支merge到main分支：`git merge dev`

这边的例子比较简单，可以直接merge，没有冲突也没有更改，可以直接推送。此时git log如下：

```bash
git-log
*   44a9c7c (HEAD -> main, origin/main, origin/dev, dev) [zheng] dev merge main
|\\
| * 7f080da [zheng] main分支第二次开发
| * b8056d4 [zheng] main分支第一次开发
* | cf3c5ab [zheng] dev分支所有开发合并，待merge
|/
* 2df8c24 [zheng] 初始化仓库
```

#### 3.3. 删除dev分支

此时，dev分支开发完毕并且合并到main分支，可以将其删除

删除本地dev分支：`git branch -d dev`

删除远端dev分支：`git push origin --delete dev`

删除后git log如下：

```bash
git-log
*   44a9c7c (HEAD -> main, origin/main) [zheng] dev merge main
|\\
| * 7f080da [zheng] main分支第二次开发
| * b8056d4 [zheng] main分支第一次开发
* | cf3c5ab [zheng] dev分支所有开发合并，待merge
|/
* 2df8c24 [zheng] 初始化仓库
```

### 4. 采用rebase的方式合并（不推荐）

#### 4.1. 在dev上rebase到main

在dev上rebase到main：`git rebase main`

需要解决冲突，解决完后：`git add .` && `git rebase --continue`

会弹出一个编辑器，内容如下：

```bash
[zheng] dev分支所有开发合并，待merge

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# interactive rebase in progress; onto 7f080da
# Last command done (1 command done):
#    pick cf3c5ab [zheng] dev分支所有开发合并，待merge
# No commands remaining.
# You are currently rebasing branch 'dev' on '7f080da'.
#
# Changes to be committed:
#	modified:   README.md
#
```

我改成如下进行保存关闭：

```bash
# [zheng] dev分支所有开发合并，待merge

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# interactive rebase in progress; onto 7f080da
# Last command done (1 command done):
#    pick cf3c5ab [zheng] dev分支所有开发合并，待merge
# No commands remaining.
# You are currently rebasing branch 'dev' on '7f080da'.
#
# Changes to be committed:
#	modified:   README.md
#
[zheng] dev上进行rebase main
```

此时的git log如下（可以跟2中的最后一个git log对比一下，发现此时的dev接到了main的后面）：

```bash
git-log
* 96bd056 (HEAD -> dev) [zheng] dev上进行rebase main
* 7f080da (origin/main, main) [zheng] main分支第二次开发
* b8056d4 [zheng] main分支第一次开发
| * cf3c5ab (origin/dev) [zheng] dev分支所有开发合并，待merge
|/
* 2df8c24 [zheng] 初始化仓库
```

#### 4.2. 将dev分支merge到主分支

不用提交，直接切换到main分支进行合并：`git merge dev`

此时的git log如下：

```bash
git-log
* 96bd056 (HEAD -> main, dev) [zheng] dev上进行rebase main
* 7f080da (origin/main) [zheng] main分支第二次开发
* b8056d4 [zheng] main分支第一次开发
| * cf3c5ab (origin/dev) [zheng] dev分支所有开发合并，待merge
|/
* 2df8c24 [zheng] 初始化仓库
```

这时候可以直接将main分支push到远端，不需要-f就能成功，git log如下：

```bash
git-log
* 96bd056 (HEAD -> main, origin/main, dev) [zheng] dev上进行rebase main
* 7f080da [zheng] main分支第二次开发
* b8056d4 [zheng] main分支第一次开发
| * cf3c5ab (origin/dev) [zheng] dev分支所有开发合并，待merge
|/
* 2df8c24 [zheng] 初始化仓库
```

#### 4.3. 删除dev分支

删除远端dev分支：`git push origin --delete dev`

删除本地dev分支：`git branch -d dev`

此时git log如下，是一个非常简洁的历史记录：

```bash
git-log
* 96bd056 (HEAD -> main, origin/main) [zheng] dev上进行rebase main
* 7f080da [zheng] main分支第二次开发
* b8056d4 [zheng] main分支第一次开发
* 2df8c24 [zheng] 初始化仓库
```
