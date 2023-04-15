## Git

### 1. git pull 和 git fetch 的区别

git pull 是 git fetch + git merge

git fetch 只是将远程仓库的最新的版本下载到本地，但是不会自动 merge，相当于工作区中的文件并没有更新

git pull 会从远程仓库获取到最新的版本并 merge 到本地。

git fetch 更保险一些，git pull 操作更简单

### 2. git merge 和 git rebase 的区别

git merge 操作合并分支会让两个分支的共同提交点之后每一次提交都按照提交时间（并不是 push 时间）排序，并且会将两个分支的最新一次 commit 点进行合并成一个新的 commit，最终的分支树呈现非整条线性直线的形式

git rebase 操作实际上是将当前执行 rebase 分支的所有基于原分支提交点之后的 commit 打散成一个一个的 patch，并重新生成一个新的 commit hash 值，再次基于原分支目前最新的 commit 点上进行提交，并不根据两个分支上实际的每次提交的时间点排序，rebase 完成后，切到基分支进行合并另一个分支时也不会生成一个新的 commit 点，可以保持整个分支树的完美线性

### 3. 什么是 git cherry-pick？

命令 git cherry-pick 通常用于把特定提交从存储仓库的一个分支引入到其他分支中。常见的用途是从维护的分支到开发分支进行向前或回滚提交。
