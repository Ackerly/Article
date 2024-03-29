# 前端架师了解的git知识
## 分支管理策略
git 分支强大的同时也非常灵活，如果没有一个好的分支管理策略，团队人员随意合并推送，就会造成分支混乱，各种覆盖，冲突，丢失等问题。  
目前最流行的分支管理策略，也称工作流（Workflow），主要包含三种：  
- Git Flow
- GitHub Flow
- GitLab Flow

通常分支有4大类：  
- dev-*
- develop
- staging
- release

dev-* 是一组开发分支的统称，包括个人分支，模块分支，修复分支等，团队开发人员在这组分支上进行开发。  
开发前，先通过 merge 合并 develop 分支的最新代码；开发完成后，必须通过 cherry-pick 合并回 develop 分支。  
develop 是一个单独分支，对应开发环境，保留最新的完整的开发代码。它只接受 cherry-pick 的合并，不允许使用 merge。  
staging 分支对应测试环境。当 develop 分支有更新并且准备发布测试时，staging 要通过 rebase 合并 develop 分支，然后将最新代码发布到测试服务器，供测试人员测试。  
测试发现问题后，再走 dev-* -> develop -> staging 的流程，直到测试通过。  
release 则表示生产环境。release 分支的最新提交永远与线上生产环境代码保持同步，也就是说，release 分支是随时可发布的。  
当 staging 测试通过后，release 分支通过 rebase 合并 staging 分支，然后将最新代码发布到生产服务器。
## 为什么合并到 develop 必须用 cherry-pick
使用 merge 合并，如果有冲突，会产生分叉；dev-* 分支多而杂，直接 merge 到 develop 会产生错综复杂的分叉，难以理清提交进度。  
而 cherry-pick 只将需要的 commit 合并到 develop 分支上，且不会产生分叉，使 git 提交图谱（git graph）永远保持一条直线。  
模块开发分支完成后，需要将多个 commit 合为一个 commit，再合并到 develop 分支，避免了多余的 commit，这也是不用 merge 的原因之一。  
## 为什么合并到 staging/release 必须用 rebase
rebase 译为变基，合并同样不会产生分叉。当 develop 更新了许多功能，要合并到 staging 测试，不可能用 cherry-pick 一个一个把 commit 合并过去。因此要通过 rebase 一次性合并过去，并且保证了 staging 与 develop 完全同步。  
release 也一样，测试通过后，用 rebase 一次性将 staging 合并过去，同样保证了 staging 与 release 完全同步。  
## commit 规范与提交验证
为了直观的看出 commit 的更新内容，开发者社区诞生了一种规范，将 commit 按照功能划分，加一些固定前缀，比如 fix:，feat:，用来标记这个 commit 主要做了什么事情。  
目前主流的前缀包括以下部分：  
- build：表示构建，发布版本可用这个
- ci：更新 CI/CD 等自动化配置
- chore：杂项，其他更改
- docs：更新文档
- feat：常用，表示新增功能
- fix：常用：表示修复 bug
- perf：性能优化
- refactor：重构
- revert：代码回滚
- style：样式更改
- test：单元测试更改

这些前缀每次提交都要写，刚开始很多人还是记不住的。这里推荐一个非常好用的工具，可以自动生成前缀。  
首先全局安装：  
``` 
npm install -g commitizen cz-conventional-changelog
```
创建 ~/.czrc 文件，写入如下内容：  
``` 
{ "path": "cz-conventional-changelog" }
```
用 git cz 命令来代替 git commit 命令，然后上下箭选择前缀，根据提示即可方便的创建符合规范的提交。  
有了规范之后，光靠人的自觉遵守是不行的，还要在流程上对提交信息进行校验.这个时候用到一个新东西 —— git hook，也就是 git 钩子。  
git hook 的作用是在 git 动作发生前后触发自定义脚本。这些动作包括提交，合并，推送等，可以利用这些钩子在 git 流程的各个环节实现自己的业务逻辑。  
git hook 分为客户端 hook 和服务端 hook。  
客户端 hook 主要有四个：  
- pre-commit：提交信息前运行，可检查暂存区的代码
- prepare-commit-msg：不常用
- commit-msg：非常重要，检查提交信息就用这个钩子
- post-commit：提交完成后运行

服务端 hook 包括：  
- pre-receive：非常重要，推送前的各种检查都在这
- post-receive：不常用
- update：不常用

大多数团队是在客户端做校验，可以用 commit-msg 钩子在客户端对 commit 信息做校验。不需要我们手动去写校验逻辑，社区有成熟的方案：husky + commitlint  
husky 是创建 git 客户端钩子的神器，commitlint 是校验 commit 信息是否符合上述规范。两者配合，可以阻止创建不符合 commit 规范的提交，从源头保证提交的规范。  
## 误操作的撤回方案
开发中频繁使用 git 拉取推送代码，难免会有误操作。撤回主要是两个命令：reset 和 revert。  
**git reset**  
reset 命令的原理是根据 commitId 来恢复版本。因为每次提交都会生成一个 commitId，所以说 reset 可以帮你恢复到历史的任何一个版本。  
reset 命令格式如下：  
``` 
$ git reset [option] [commitId]
```
比如，要撤回到某一次提交，命令是这样：  
``` 
$ git reset --hard cc7b5be
```
用 git log 命令查看提交记录，可以看到 commitId 值，这个值很长，取前 7 位即可。  
这里的 option 用的是 --hard，其实共有 3 个值，具体含义如下：
- --hard：撤销 commit，撤销 add，删除工作区改动代码
- --mixed：默认参数。撤销 commit，撤销 add，还原工作区改动代码
- --soft：撤销 commit，不撤销 add，还原工作区改动代码

要格外注意 --hard，使用这个参数恢复会删除工作区代码。也就是说，如果你的项目中有未提交的代码，使用该参数会直接删除掉，不可恢复！  
除了使用 commitId 恢复，git reset 还提供了恢复到上一次提交的快捷方式：  
``` 
$ git reset --soft HEAD^
```
其实平日开发中最多的误操作是这样：刚刚提交完，突然发现了问题，比如提交信息没写好，或者代码更改有遗漏，这时需要撤回到上次提交，修改代码，然后重新提交。  
针对这个流程，git 还提供了一个更便捷的方法：  
``` 
$ git commit --amend
```
这个命令会直接修改当前的提交信息。如果代码有更改，先执行 git add，然后再执行这个命令，比上述的流程更快捷更方便。  
reset 还有一个非常重要的特性，就是真正的后退一个版本。  
比如说当前提交，你已经推送到了远程仓库；现在你用 reset 撤回了一次提交，此时本地 git 仓库要落后于远程仓库一个版本。此时你再 push，远程仓库会拒绝，要求你先 pull。如果你需要远程仓库也后退版本，就需要 -f 参数，强制推送，这时本地代码会覆盖远程代码。  
**git revert**  
revert 与 reset 的作用一样，都是恢复版本，但是它们两的实现方式不同。reset 直接恢复到上一个提交，工作区代码自然也是上一个提交的代码；而 revert 是新增一个提交，但是这个提交是使用上一个提交的代码。它们两恢复后的代码是一致的，区别是一个新增提交（revert），一个回退提交（reset）。  
为 revert 永远是在新增提交，因此本地仓库版本永远不可能落后于远程仓库，可以直接推送到远程仓库，故而解决了 reset 后推送需要加 -f 参数的问题，提高了安全性。  
使用方法：  
``` 
$ git revert -n [commitId]
```
## Tag 与生产环境
git 支持对于历史的某个提交，打一个 tag 标签，常用于标识重要的版本更新。  
目前普遍的做法是，用 tag 来表示生产环境的版本。当最新的提交通过测试，准备发布之时，我们就可以创建一个 tag，表示要发布的生产环境版本。  
比如要发一个 v1.2.4 的版本：  
``` 
$ git tag -a v1.2.4 -m "my version 1.2.4"
```
然后可以查看：  
``` 
$ git show v1.2.4

> tag v1.2.4
Tagger: ruims <2218466341@qq.com>
Date:   Sun Sep 26 10:24:30 2021 +0800

my version 1.2.4
```
最后用 git push 将 tag 推到远程：  
``` 
$ git push origin v1.2.4
```
tag 和在哪个分支创建是没有关系的，tag 只是提交的别名。因此 commit 的能力 tag 均可使用，比如上面说的 git reset，git revert 命令。  
当生产环境出问题，需要版本回退时，可以这样：  
``` 
$ git revert [pre-tag]
# 若上一个版本是 v1.2.3，则：
$ git revert v1.2.3
```
在频繁更新，commit 数量庞大的仓库里，用 tag 标识版本显然更清爽，可读性更佳。  
release 分支与生产环境代码同步。在 CI/CD（下面会讲到）持续部署的流程中，我们是监听 release 分支的推送然后触发自动构建。那是不是也可以监听 tag 推送再触发自动构建，这样版本更新的直观性是不是更好？  
## 永久杜绝 443 Timeout
众所周知的原因，GitHub 拉取和推送的速度非常慢，甚至直接报错：443 Timeout。速度慢超时可能是因为被墙，比如 GitHub 首页打不开。再究其根源，被墙的是访问网站时的 http 或 https 协议，那么其他协议是不是就不会有墙的情况？使用 ssh 协议克隆代码速度会更快。
## hook 实现部署
利用 git hook 实现部署，应该是 hook 的高级应用了。现在有很多工具，比如 GitHub，GitLab，都提供了持续集成功能，也就是监听某一分支推送，然后触发自动构建，并自动部署。  
## 终极应用: CI/CD
CI（Continuous Integration）译为持续集成，CD 包括两部分，持续交付（Continuous Delivery）和持续部署（Continuous Deployment）  
从全局看，CI/CD 是一种通过自动化流程来频繁向客户交付应用的方法。这个流程贯穿了应用的集成，测试，交付和部署的整个生命周期，统称为 “CI/CD 管道”。  
持续集成是频繁地将代码集成到主干分支。当新代码提交，会自动执行构建、测试，测试通过则自动合并到主干分支，实现了产品快速迭代的同时保持高质量。  
持续交付是频繁地将软件的新版本，交付给质量团队或者用户，以供评审。评审通过则可以发布生产环境。持续交付要求代码（某个分支的最新提交）是随时可发布的状态。  
持续部署是代码通过评审后，自动部署到生产环境。持续部署要求代码（某个分支的最新提交）是随时可部署的。  
持续部署与持续交付的唯一区别，就是部署到生产环境这一步，是否是自动化。部署自动化，看似是小小的一步，但是在实践过程中，这反而是 CI/CD 流水线中最难落实的一环。  
运维是手动部署，要实现自动部署，就要有服务器权限，与服务器交互。这也是个大问题，因为运维团队一定会顾虑安全问题，因而推动起来节节受阻
前社区成熟的 CI/CD 方案有很多，比如老牌的 jenkins，react 使用的 circleci，还有GitHub Action等
原文:  
[前端架构师的 git 功力，你有几成火候？](https://juejin.cn/post/7024043015794589727)