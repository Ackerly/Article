# 用 Git Subtree 在多个 Git 项目间双向同步子项目
有赞微商城曾经是一个很大的前后端代码都包含在里面的 Git 项目，为了方便管理我们把前后端代码分离成2个 Git 仓库，进而再作分项目拆分成多个Git 仓库。  
于是，就需要有好的方式同步各个项目共用的Css库、JS库、PHP库（他们都是以独立的 Git 仓库的形式存在）。而且由于开发节奏极快，我们需要这些库是可以在不同项目间双向同步的而不是单向同步。而且，最好能做到被迁移的这部分代码在新的git仓库里保留原有的历史提交记录。  
A项目需要在给某个子项目W里添加一个文件，最方便的方式自然是直接在A项目里改W子项目对应的目录里的代码，然后测试通过后，把这个更改提交到W子项目的 Git仓库里。如果这时候还要先单独更新W子项目的代码然后提交到 Git 服务器，再在A项目里把W子项目的代码更新过来，显然是很麻烦的，更麻烦的是如果发现代码有bug，还得再走一遍这个流程。

方案:  
- Git Submodule：这是Git官方以前的推荐方案
- Git Subtree：从 Git 1.5.2 开始，Git 新增并推荐使用这个功能来管理子项目
- npm：node package manager，实际上不仅仅是 node 的包管理工具
- composer：暂且认为他是php版npm、php版Maven吧
- bower：针对浏览器前端的包管理工具（Web sites are made of lots of things — frameworks, libraries, assets, utilities, and rainbows. Bower manages all these things for you.），这东西很好用，我们在大量使用。

虽然 npm、composer、maven 等更侧重于包的依赖管理，以上几个方案都是能够做到在不同项目中同步同一块代码的，但没法双向同步，更适用于子项目代码比较稳定的情形。  
Git Submodule 和 Git Subtree 都是官方支持的功能，不具有依赖管理的功能，但能满足我们的要求。Git Subtree相对来说会更好一些 。  
**Git Subtree 好在哪里**  
> 经由 Git Subtree 来维护的子项目代码，对于父项目来说是透明的，所有的开发人员看到的就是一个普通的目录，原来怎么做现在依旧那么做，只需要维护这个 Subtree 的人在合适的时候去做同步代码的操作。

## Git Subtree 的原理
有两个伟大的项目——我们叫他P1项目、P2项目，还有一个牛逼的要被多个项目共用的项目——我们叫他S项目。通过简要讲解使用Subtree来同步代码的过程来解释Subtree的原理  
**1、初始化子项目Subtree**  
```
cd P1项目的路径  
git subtree add --prefix=用来放S项目的相对路径 S项目git地址 xxx分支  12
```
这样的命令，把S项目（我们姑且叫他S项目）的代码下载到--prefix所指定的目录，并在P1项目里自动产生一个commit（就是把S目录的内容提交到P1项目里）。  
对于P2项目也做同样的操作  
**2、像往常一样更新代码**  
在P1项目里各种提交commit，其中有些commit会涉及到S目录的更改，正如前面提到的，这是没任何关系的，也不会感受到有任何不一样。  
**提交更改到子项目的Git服务器**  
关键的地方来了： 当维护这个S项目 Subtree 的人希望把最近这段时间对S目录的更改提交到S项目的 Git 服务器上时，他执行一段类似于这样的命令：
```
cd P1项目的路径  
git subtree push --prefix=S项目的路径 S项目git地址 xxx分支  12
```
Git 会遍历所有的commit，从中找出针对S目录的更改，然后把这些更改记录提交到S项目的Git服务器上  
**4、更新子项目新的代码到父项目**  
现在S项目有大量的新代码了，P2项目也想使用这些新代码，维护P2这个Subtree的人只要执行：
```
git subtree pull --prefix=S项目的路径 S项目git地址 xxx分支  1
```
这样就可以将P2项目里S项目目录里的内容更新为S项目xxx分支的最新代码了。  

## 总结的 Git Subtree 简明使用手册  
1. 首先必须确保各个项目已经添加zenjs 这个 remote（关于remote是什么可以看这里）:
```
git remote add zenjs http://github.com/youzan/zenjs.git  1
```
2. 将zenjs添加到各个项目里
```
git subtree add --prefix=components/zenjs zenjs master  1
```
3. 各项目更新zenjs代码的方法:
```
git subtree pull --prefix=components/zenjs zenjs master  1
```
4. 各项目提交zenjs代码的方法:
```
git subtree push --prefix=components/zenjs zenjs hotfix/zenjs_xxxx  1
```

这会在远程的zenjs的仓库里生成一个叫 hotfix/zenjs_xxxx 的的分支，包含了你过去对components/zenjs 所有的更改记录  
把hotfix/zenjs_xxx分支更新并合并到master并提交,这样其他工程就可以更新到你提交的代码了。  
> 有人可能会问，只用master分支，不管版本，太有风险了。  
> 对的，正如我们前面说到的那样，subtree的方案适用的场景是：各个项目共用一个库，而这个库正在快速迭代更新的过程中。如果追求稳定，只需要给库拉出一个如v0.1.0这样的版本号命名的稳定分支，subtree只用这个分支即可。
> 使用的方式就是：A项目经常会对zenjs做更新，所以A项目用subtree来双向同步；B项目只是使用，所以用bower用来按版本来更新代码。

高阶功能: 重新split出一个新起点（这样，每次提交subtree的时候就不会从头遍历一遍了）  
```
git subtree split --rejoin --prefix=components/zenjs --branch new_zenjs  
git push zenjs new_zenjs:master  
```
参考:
[用 Git Subtree 在多个 Git 项目间双向同步子项目](https://juejin.cn/post/6844903762176262157)