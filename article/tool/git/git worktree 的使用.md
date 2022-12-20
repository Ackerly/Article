# git worktree 的使用
在同一个 git 仓库中，经常会建立多个分支工作。在多个分支之间切换的时候，如果一个分支不是干净的，则需要暂存再切换，甚者，需要多个分支同时进行的时候，只能重新 git clone ，然后使用远程仓库来进行同步。其实是有点麻烦的。  
git 从 2.6.0 的版本开始增加了新的指令，可以用来解决这个问题，就是：  
``` 
git worktree
```
一个 git 仓库默认有一个 worktree，当需要在同一个仓库兼顾两个或者多个分支的开发时，可以为每一个分支新建一个 worktree 。他们彼此之间不会相互影响，其表现相当于在一个其他的目录重新 git clone 了一把这个 git 仓库。实际上与重建目录不同的是，他们彼此之间又有关联，任何一个 worktree 的提交都会无痛增加到其他的 worktree ，而不需要通过远程仓库做同步。一个分支的工作结束之后，删除那个分支对应的目录即可关闭这个 worktree。总之就是，看来还算完美的一个解决方案。  

**常用**
git worktree add <path> [<branch>]  
增加一个新的 worktree ，并指定了其关联的目录是 path ，关联的分支是 <branch> 。后者是一个可选项，默认值是 HEAD 分支。如果 <branch> 已经被关联到了一个 worktree ，则这次 add 会被拒绝执行，可以通过增加 -f | --force 选项来强制执行。  
同时，可以使用 -b <new-branch> 基于 <branch> 新建分支并使这个新分支关联到这个新的 worktree 。如果 <new-branch> 已经存在，则这次 add 会被拒绝，可以使用 -B 代替这里的 -b 来强制执行，则原来的 <new-branch> 的提交进度会被重置为和 <branch> 一样的位置。
git worktree list  
列出当前仓库已经存在的所有 worktree 的详细情况，包括每个 worktree 的关联目录，当前的提交点的哈希码和当前 checkout 到的关联分支。若没有关联分支，则是 detached HEAD 。  
可以增加 --porcelain 选项，用来改变显示风格。即：使用 label 对应 value 的形式显示上面提到的内容。举例容易看出其中差别:  
``` 
$ git worktree list
 /path/to/bare-source            (bare)
 /path/to/linked-worktree        abcd1234 [master]
 /path/to/other-linked-worktree  1234abc  (detached HEAD)
 
 $ git worktree list --porcelain
 worktree /path/to/bare-source
 bare
 
 worktree /path/to/linked-worktree
 HEAD abcd1234abcd1234abcd1234abcd1234abcd1234
 branch refs/heads/master
 
 worktree /path/to/other-linked-worktree
 HEAD 1234abc1234abc1234abc1234abc1234abc1234a
 detached
```
在删除 worktree 的关联目录之后，清除 worktree 的信息。从而使一个 worktree 完整的删除。

**其他**  
git worktree lock  
git 会定期的自动清除掉已经没有关联目录的那些 worktree 的信息。当你把一个 worktree 的关联目录创建到了一个可移动设备或者一块不是永久挂载的硬盘里的时候，使用这个命令可以防止这个 worktree 的信息被移除。  
git worktree unlock  
与上面的命令是一对。作用是解除锁定。  

原文:  
[git worktree 的使用](https://mp.weixin.qq.com/s/MkD_YIpTsaoxMiV4in7-QQ)
