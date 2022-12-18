# 如何更新 NPM 依赖
Node 软件包管理器（NPM）提供了各种功能来帮助你安装和维护项目的依赖关系。  
由于错误修复、新功能和其他更新，依赖关系可能会随着时间的推移而变得过时。你的项目依赖越多，就越难跟上这些更新。
老旧的软件包会对安全构成威胁，并会对性能产生负面影响。最新的软件包可以防止出现漏洞。这意味着定期检查和更新依赖是很重要的。  
**如何保持依赖是最新的**  
可以逐一查看 package.json 中的每一个单独的包，改变版本，然后运行 npm install <package>@latest 来获得最新版本。但这并不是最有效的方法。  
想象一下，如果你有 20 个或更多的包，可以使用版本升级。相反，你应该制定一个工作流程，在过期的依赖关系数量增加和升级变得越来越难之前，定期检查新版本。  
一个保持更新的工作流程：首先，发现哪些软件包需要更新，以及版本落后的程度。接下来，选择单独或一起批量更新软件包。始终对更新进行测试，以确保没有发生破坏性变化。  
对于主要的更新，你很可能会遇到破坏性的变化。与许多包相比，撤销或处理与一个包有关的代码变化要容易得多。  

**怎样使用 npm outdated 命令**  
```
npm outdated
```
该命令将检查每个已安装的依赖关系，并将当前版本与 npm registry 中的最新版本进行比较。它在终端打印出一个表格，概述了可用的版本。  
它是内置在 NPM 中的，所以不需要下载额外的软件包。npm outdated 是一个很好的开始，可以了解所需的依赖性更新的数量。  
- Current：是当前安装的版本。
- Wanted：是根据 semver 范围内的软件包的最大版本。
- Latest：是在 npm registry 中被标记为最新的软件包版本。

使用这种方法，要安装每个软件包的更新，你只需要运行：  
``` 
npm update
```
使用 npm update 它永远不会更新到一个主要的（major），具有破坏性变化的版本。它更新 package.json 和 package-lock.json 中的依赖关系。它将使用 想要的 版本。  
为了获得 "最新" 的版本，在单个安装中附加 @latest，例如 npm install react@latest。  
**怎样使用 npm-check-updates**  
对于高级和可定制的升级体验，推荐 npm-check-updates。这个包可以做所有 npm oudated 和 npm upgrade 能做的事情，并增加了一些自定义选项。不过，它确实需要安装一个包。  
要开始使用，请在全局安装 npm-check-updates 软件包：  
``` 
npm install -g npm-check-updates
```
运行 ncu 来显示要升级的软件包。与 npm outdated 类似，它不会产生任何变化。  
要升级依赖性，你只需要运行：  
``` 
ncu --upgrade

 // or
 ncu -u
```
- Red （显示红色） = major （主版本，或者说是大版本）
- Cyan （显示青色） = minor（次要版本）
- Green（显示绿色） = patch （补丁版本）

这个方法只更新 package.json 文件中的依赖关系，并且会选择最新的版本，即使它包括一个破坏性的变化。使用这种方法，npm install 不会自动运行，所以一定要在之后运行它来更新 package-lock.json。  
要选择你喜欢的版本类型，运行 ncu --target [patch, minor, latest, newest, greatest]。  

**如何使用 npm-check-updates 互动模式**  
``` 
ncu --interactive

 // or
 ncu -i
```
互动模式允许你选择特定的软件包进行更新。默认情况下，所有软件包都被选中。  
向下浏览每一个软件包，用空格来取消选择，当你准备好升级所有选择的软件包时，回车键（enter）确定。  
有几种方法可以提升交互式 npm-check-updates 的体验。  
``` 
ncu --interactive --format group
```
这个命令将软件包分组并组织成 主版本（major）、次要（minor）和补丁（patch）版本。  
npm-check-updates 提供了其他有用的工具，如 doctor mode，它可以安装升级并运行测试以检查破坏性变化。  



原文:  
[如何更新 NPM 依赖](https://mp.weixin.qq.com/s/TvLWXX4bpVYOalywiupvFQ)
