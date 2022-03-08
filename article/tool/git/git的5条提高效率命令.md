# git的5条提高效率命令
## stash
stash 命令能够将还未 commit 的代码存起来，让你的工作目录变得干净。  
**应用场景**  
应用场景：某一天你正在 feature 分支开发新需求，突然产品经理跑过来说线上有bug，必须马上修复。而此时你的功能开发到一半，于是你急忙想切到 master 分支，因为当前有文件更改了，需要提交commit保持工作区干净才能切分支。由于情况紧急，你只有急忙 commit 上去，commit 信息也随便写了个“暂存代码”，于是该分支提交记录就留了一条黑历史.  
**命令使用**  
```
git stash
```
就这么简单，代码就被存起来了,当你修复完线上问题，切回 feature 分支，想恢复代码也只需要：
```
git stash apply
```
**相关命令**  
```
# 保存当前未commit的代码
git stash

# 保存当前未commit的代码并添加备注
git stash save "备注的内容"

# 列出stash的所有记录
git stash list

# 删除stash的所有记录
git stash clear

# 应用最近一次的stash
git apply

# 应用最近一次的stash，随后删除该记录
git pop

# 删除最近的一次stash
git stash drop
```
当有多条 stash，可以指定操作stash，首先使用stash list 列出所有记录：  
```
$ git stash list
stash@{0}: WIP on ...
stash@{1}: WIP on ...
stash@{2}: On ...
```
应用第二条记录：
```
$ git stash apply stash@{1}
```
## reset --soft
回退你已提交的 commit，并将 commit 的修改内容放回到暂存区。  
git reset --hard 会被提及的比较多，它能让 commit 记录强制回溯到某一个节点。而 git reset --soft 的作用正如其名，--soft (柔软的) 除了回溯节点外，还会保留节点的修改内容。  
**应用场景**  
应用场景1：有时候手滑不小心把不该提交的内容 commit了，这时想改回来，只能再 commit 一次，又多一条“黑历史”  
应用场景2：规范些的团队，一般对于 commit 的内容要求职责明确，颗粒度要细，便于后续出现问题排查。本来属于两块不同功能的修改，一起 commit 上去，这种就属于不规范。这次恰好又手滑了，一次性 commit 上去。  
**命令使用**  
```
# 恢复最近一次 commit
git reset --soft HEAD^
```
reset --soft 相当于后悔药，给你重新改过的机会。对于上面的场景，就可以再次修改重新提交，保持干净的 commit 记录。  
对于已经 push 的 commit，也可以使用该命令，不过再次 push 时，由于远程分支和本地分支有差异，需要强制推送 git push -f 来覆盖被 reset 的 commit。  
在 reset --soft 指定 commit 号时，会将该 commit 到最近一次 commit 的所有修改内容全部恢复，而不是只针对该 commit。  
举个栗子：  
commit 记录有 c、b、a，reset 到 a，此时的 HEAD 到了 a，而 b、c 的修改内容都回到了暂存区。  
## cherry-pick
将已经提交的 commit，复制出新的 commit 应用到分支里  
**应用场景**  
应用场景1：有时候版本的一些优化需求开发到一半，可能其中某一个开发完的需求要临时上，或者某些原因导致待开发的需求卡住了已开发完成的需求上线。这时候就需要把 commit 抽出来，单独处理。  
应用场景2：有时候开发分支中的代码记录被污染了，导致开发分支合到线上分支有问题，这时就需要拉一条干净的开发分支，再从旧的开发分支中，把 commit 复制到新分支。  
**命令使用**  
复制单个：  
1. 需要把 b 复制到另一个分支，首先把 commitHash 复制下来，然后切到 master 分支。
2. 需要把 b 复制到另一个分支，首先把 commitHash 复制下来，然后切到 master 分支。
3. 完成后看下最新的 log，b 已经应用到 master，作为最新的 commit 了。commitHash 和之前的不一样，但是提交时间还是保留之前的。

复制多个：  
一次转移多个提交,将 commit1 和 commit2 两个提交应用到当前分支。
```
git cherry-pick commit1 commit2
```
多个连续的commit，也可区间复制：
```
git cherry-pick commit1^..commit2
```
上面的命令将 commit1 到 commit2 这个区间的 commit 都应用到当前分支（包含commit1、commit2），commit1 是最早的提交。
**cherry-pick 代码冲突**  
在 cherry-pick 多个commit时，可能会遇到代码冲突，这时 cherry-pick 会停下来，让用户决定如何继续操作。  
例如:  
1. feature 分支，现在需要把 c、d、e 都复制到 master 分支上。先把起点c和终点e的 commitHash 记下来。
2. 切到 master 分支，使用区间的 cherry-pick。可以看到 c 被成功复制，当进行到 d 时，发现代码冲突，cherry-pick 中断了。这时需要解决代码冲突，重新提交到暂存区。
3. 然后使用 cherry-pick --continue 让 cherry-pick 继续进行下去。最后 e 也被复制进来，整个流程就完成了。

在代码冲突后，放弃或者退出流程：
```
// 放弃 cherry-pick
gits cherry-pick --abort
// 退出 cherry-pick
git cherry-pick --quit
```
## revert
将现有的提交还原，恢复提交的内容，并生成一条还原记录。  
**应用场景**  
有一天测试突然跟你说，你开发上线的功能有问题，需要马上撤回，否则会影响到系统使用。这时可能会想到用 reset 回退，可是你看了看分支上最新的提交还有其他同事的代码，用 reset 会把这部分代码也撤回了。由于情况紧急，又想不到好方法，还是任性的使用 reset，然后再让同事把他的代码合一遍。  
**命令使用**  
revert 普通提交  
```
git revert 21dcd937fe555f58841b17466a99118deb489212
```
revert 掉自己提交的 commit,因为 revert 会生成一条新的提交记录，这时会让你编辑提交信息，编辑完后 :wq 保存退出就好了。  
生成了一条 revert 记录，虽然自己之前的提交记录还是会保留着，但你修改的代码内容已经被撤回了。  
**revert 合并提交**  
刚刚同样的 revert 方法，会发现命令行报错了,因为通常无法 revert 合并，因为不知道合并的哪一侧应被视为主线。此选项指定主线的父编号（从1开始），并允许 revert 反转相对于指定父编号的更改
因为合并提交是两条分支的交集节点，而 git 不知道需要撤销的哪一条分支，需要添加参数 -m 指定主线分支，保留主线分支的代码，另一条则被撤销。  
-m 后面要跟一个 parent number 标识出"主线"，一般使用 1 保留主分支代码。  
```
git revert -m 1 <commitHash>
```
**revert 合并提交后，再次合并分支会失效**  
在 master 分支 revert 合并提交后，然后切到 feature 分支修复好 bug，再合并到 master 分支时，会发现之前被 revert 的修改内容没有重新合并进来。  
因为使用 revert 后， feature 分支的 commit 还是会保留在 master 分支的记录中，当你再次合并进去时，git 判断有相同的 commitHash，就忽略了相关 commit 修改的内容。这时就需要 revert 掉之前 revert 的合并提交  
## reflog
如果说 reset --soft 是后悔药，那 reflog 就是强力后悔药。它记录了所有的 commit 操作记录，便于错误操作后找回记录。
**应用场景**
应用场景：某天你眼花，发现自己在其他人分支提交了代码还推到远程分支，这时因为分支只有你的最新提交，就想着使用 reset --hard，结果紧张不小心记错了 commitHash，reset 过头，把同事的 commit 搞没了。没办法，reset --hard 是强制回退的，找不到 commitHash 了，只能让同事从本地分支再推一次。
**命令使用**  
分支记录如上，想要 reset 到 b,误操作 reset 过头，b 没了，最新的只剩下 a。这时用 git reflog 查看历史记录，把错误提交的那次 commitHash 记下。再次 reset 回去，就会发现 b 回来了。
## 设置 Git 短命令
方式一
```
git config --global alias.ps push
```
方式二  
打开全局配置文件  
```
vim ~/.gitconfig

```
写入内容  
```
[alias] 
        co = checkout
        ps = push
        pl = pull
        mer = merge --no-ff
        cp = cherry-pick

```
使用
```
# 等同于 git cherry-pick <commitHash>
git cp <commitHash>
```

参考：  
[Git不要只会pull和push，试试这5条提高效率的命令](https://juejin.cn/post/7071780876501123085?share_token=3d46085e-742f-44e8-9767-dc47a44e0859)