# 45 个 Git 经典操作场景
**我刚才提交了什么?**  
如果用 git commit -a 提交了一次变化(changes)，而又不确定到底这次提交了哪些内容。就可以用下面的命令显示当前HEAD上的最近一次的提交(commit):
```
git show  
// 或者
git log -n1 -p  
```
**我的提交信息(commit message)写错了**  
如果你的提交信息(commit message)写错了且这次提交(commit)还没有推(push), 你可以通过下面的方法来修改提交信息(commit message):  
```
git commit --amend --only  
```
这会打开你的默认编辑器, 在这里你可以编辑信息. 另一方面, 你也可以用一条命令一次完成:  
```
git commit --amend --only -m 'xxxxxxx'  
```
如果你已经推(push)了这次提交(commit), 可以修改这次提交(commit)然后强推(force push), 但是不推荐这么做。  
**提交(commit)里的用户名和邮箱不对**  
只是单个提交(commit)，修改它：
```
git commit --amend --author "New Authorname <authoremail@mydomain.com>"  
```
**从一个提交(commit)里移除一个文件**  
```
git checkout HEAD^ myfile  
git add -A  
git commit --amend 
```
这将非常有用，当你有一个开放的补丁(open patch)，你往上面提交了一个不必要的文件，你需要强推(force push)去更新这个远程补丁。  
**删除我的的最后一次提交**  
需要删除推了的提交(pushed commits)，你可以使用下面的方法。可是，这会不可逆的改变你的历史，也会搞乱那些已经从该仓库拉取(pulled)了的人的历史。  
```
git reset HEAD^ --hard  
git push -f [remote] [branch]  
```
如果你还没有推到远程, 把Git重置(reset)到你最后一次提交前的状态就可以了(同时保存暂存的变化):
```
git reset --soft HEAD@{1} 
```
这只能在没有推送之前有用. 如果你已经推了, 唯一安全能做的是 git revert SHAofBadCommit， 那会创建一个新的提交(commit)用于撤消前一个提交的所有变化(changes)；或者, 如果你推的这个分支是rebase-safe的 (例如：其它开发者不会从这个分支拉), 只需要使用 git push -f。  
**删除任意提交(commit)**  
```
git rebase --onto SHA1_OF_BAD_COMMIT^ SHA1_OF_BAD_COMMIT  
git push -f [remote] [branch]  
```
或者做一个 交互式rebase 删除那些你想要删除的提交(commit)里所对应的行。  
**尝试推一个修正后的提交(amended commit)到远程，但是报错：**  
 rebasing和修正(会用一个新的提交(commit)代替旧的, 所以如果之前你已经往远程仓库上推过一次修正前的提交(commit)，那你现在就必须强推(force push) (-f)。注意 – 总是 确保你指明一个分支!
```
git push origin mybranch -f  
```
一般来说, 要避免强推. 最好是创建和推(push)一个新的提交(commit)，而不是强推一个修正后的提交。后者会使那些与该分支或该分支的子分支工作的开发者，在源历史中产生冲突。  
**意外的做了一次硬重置(hard reset)，我想找回我的内容**  
意外的做了 git reset --hard, 你通常能找回你的提交(commit), 因为Git对每件事都会有日志，且都会保存几天。  
```
git reflog  
```
将会看到一个你过去提交(commit)的列表, 和一个重置的提交。选择你想要回到的提交(commit)的SHA，再重置一次:  
```
git reset --hard SHA1234  
```
## 暂存(Staging)
**把暂存的内容添加到上一次的提交(commit)*  
```
git commit --amend  
```
**暂存一个新文件的一部分，而不是这个文件的全部**  
```
git add --patch filename.x  
```
-p 简写。这会打开交互模式， 你将能够用 s 选项来分隔提交(commit)；然而, 如果这个文件是新的, 会没有这个选择， 添加一个新文件时, 这样做:  
```
git add -N filename.x  
```
然后, 你需要用 e 选项来手动选择需要添加的行，执行 git diff --cached 将会显示哪些行暂存了哪些行只是保存在本地了。  
**把在一个文件里的变化(changes)加到两个提交(commit)里**  
git add 会把整个文件加入到一个提交. git add -p 允许交互式的选择你想要提交的部分.  
**想把暂存的内容变成未暂存，把未暂存的内容暂存起来**  
应该将所有的内容变为未暂存，然后再选择你想要的内容进行commit。但假定你就是想要这么做，这里你可以创建一个临时的commit来保存你已暂存的内容，然后暂存你的未暂存的内容并进行stash。然后reset最后一个commit将原本暂存的内容变为未暂存，最后stash pop回来。  
```
git commit -m "WIP"  
git add .  
git stash  
git reset HEAD^  
git stash pop --index 0  
```
注意1: 这里使用pop仅仅是因为想尽可能保持幂等。注意2: 假如你不加上--index你会把暂存的文件标记为为存储。  
## 未暂存(Unstaged)的内容
**把未暂存的内容移动到一个新分支**  
```
git checkout -b my-branch  
```
**把未暂存的内容移动到另一个已存在的分支**  
```
git stash  
git checkout my-branch  
git stash pop 
```
**丢弃本地未提交的变化**  
重置源(origin)和你本地(local)之间的一些提交(commit)：
```
# one commit  
git reset --hard HEAD^  
# two commits  
git reset --hard HEAD^^  
# four commits  
git reset --hard HEAD~4  
# or  
git checkout -f  
```
重置某个特殊的文件, 你可以用文件名做为参数:
```
git reset filename  
```
**丢弃某些未暂存的内容**  
签出(checkout)不需要的内容，保留需要的。  
```
git checkout -p
```
使用 stash， Stash所有要保留下的内容, 重置工作拷贝, 重新应用保留的部分  
```
git stash -p  
# Select all of the snippets you want to save  
git reset --hard  
git stash pop  
```
或者,stash 你不需要的部分, 然后stash drop。
```
git stash -p  
# Select all of the snippets you don't want to save  
git stash drop  
```
## 分支(Branches)
**从错误的分支拉取了内容，或把内容拉取到了错误的分支**
使用 git reflog 情况，找到在这次错误拉(pull) 之前HEAD的指向  
```
git reflog 
ab7555f HEAD@{0}: pull origin wrong-branch: Fast-forward  
c5bc55a HEAD@{1}: checkout: checkout message goes here 
```
重置分支到你所需的提交(desired commit)  
```
git reset --hard c5bc55a  
```
**扔掉本地的提交(commit)，以便我的分支与远程的保持一致**  
```
git reset --hard origin/my-branch  
```
**需要提交到一个新分支，但错误的提交到了main**  
在main下创建一个新分支，不切换到新分支,仍在main下:  
```
git branch my-branch  
```
main分支重置到前一个提交:
```
git reset --hard HEAD^  
```
如果你不想使用 HEAD^, 找到你想重置到的提交(commit)的hash(git log 能够完成)， 然后重置到这个hash。使用git push 同步内容到远程。  
**保留来自另外一个ref-ish的整个文件**  
假设你正在做一个原型方案(原文为working spike (see note)), 有成百的内容，每个都工作得很好。现在, 你提交到了一个分支，保存工作内容:  
```
git add -A && git commit -m "Adding all changes from this spike into one big commit." 
```
当你想要把它放到一个分支里 (可能是feature, 或者 develop), 你关心是保持整个文件的完整，你想要一个大的提交分隔成比较小。  
假设你有:  
- 分支 solution, 拥有原型方案， 领先 develop 分支。  
- 分支 develop, 在这里你应用原型方案的一些内容。  

可以通过把内容拿到你的分支里，来解决这个问题:  
```
git checkout solution -- file1.txt
```
**把几个提交(commit)提交到了同一个分支，而这些提交应该分布在不同的分支里**  
首先, 把main分支重置到正确的提交
```
git reset --hard a13b85e  
```
创建一个新的分支:
```
git checkout -b 21 
```
用 _cherry-pick_ 把对对应的提交放入当前分支。这意味着我们将应用(apply)这个提交(commit)，仅仅这一个提交(commit)，直接在HEAD上面
```
git cherry-pick e3851e8 
```
**删除上游(upstream)分支被删除了的本地分支**  
在github 上面合并(merge)了一个pull request, 你就可以删除你fork里被合并的分支。如果你不准备继续在这个分支里工作, 删除这个分支的本地拷贝会更干净，使你不会陷入工作分支和一堆陈旧分支的混乱之中。  
```
git fetch -p  
```
**不小心删除了分支**  
reflog, 一个升级版的日志，它存储了仓库(repo)里面所有动作的历史。  
```
(main)$ git reflog  
69204cd HEAD@{0}: checkout: moving from my-branch to main  
4e3cd85 HEAD@{1}: commit: foo.txt added  
69204cd HEAD@{2}: checkout: moving from main to my-branch
```
有一个来自删除分支的提交hash(commit hash)，接下来看看是否能恢复删除了的分支
```
(main)$ git checkout -b my-branch-help  
Switched to a new branch 'my-branch-help'  
(my-branch-help)$ git reset --hard 4e3cd85  
HEAD is now at 4e3cd85 foo.txt added  
(my-branch-help)$ ls  
README.md foo.txt  
```
**删除一个分支**  
删除一个远程分支:  
```
git push origin --delete my-branch  
// 或者
git push origin :my-branch  
```
删除一个本地分支:
```
git branch -D my-branch  
```
**从别人正在工作的远程分支签出(checkout)一个分支**  
远程拉取(fetch) 所有分支:
```
git fetch --all  
```
假设你想要从远程的daves分支签出到本地的daves
```
git checkout --track origin/daves  
```
## Rebasing 和合并(Merging)

参考：  
[45 个 Git 经典操作场景，专治不会合代码](https://mp.weixin.qq.com/s/yGLg3kLcqcCBhtBqOE0dyQ)