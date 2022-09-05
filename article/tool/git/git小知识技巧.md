# Git小知识技巧
## 查看谁修改了代码
使用 git blame <file> 来定位代码的最后一次修改。这只能查看本行代码的上体提交，而无法定位本行代码的提交历史。比如项目合作中某人对全部代码进行了格式化，git blame 就失去了作用。此时，可以与另一个有用的命令 git log -p <file> 结合使用，可以查看文件的更改历史与明细， 
``` 
git blame -L 10,12 package.json
git log -p -L 10,12:package.json
```

## 快速切换合并分支
git checkout -，表示切到最近的一次分支。如果你需要把 B 分支的内容合并过来，可以使用 git merge -。
``` 
git checkout -
git merge -
```
- 往往代表最近一次，如 cd - 代表进入最近目录，

## 统计项目
统计项目各个成员 commit 的情况，比如你可以查看你自己的项目的 commit 数以及他人对你项目的贡献数
``` 
git shortlog -sn
git shortlog -sn --no-merges      # 不包含 merge commit
```
## 快速定位提交
如果你的 commit message 比较规范，比如会关联 issuse 或者当前任务或者 bug 的编号，此时根据 commit message 快速定位： git log --grep "Add"。  
如果你的 commit message 不太规范，只记得改了哪几行代码，此时也可以根据每次提交的信息查找关键字，是 git log -S "setTimeout"。  
同时，也可以根据作者，时间来辅助快速定位。  
``` 
git log --since="0 am" 　　　     # 查看今日的提交
git log --author="shfshanyue"     # 查看 shfshanyue 的提交
git log --grep="#12"              # 查找提交信息中包含关键字的提交
git log -S "setTimeout"           # 查看提交内容中包含关键字的提交
```
## 快速定位字符串
git grep <keyword>查找包含关键字的全部文件
``` 
grep -rn <keyword>
grep -rn <keyword> --exclude config.js --exclude-dir node_modules
git grep <keyword>
ag <keyword>
```

原文:
[Git 几点小知识技巧](https://juejin.cn/post/7030441979645263909)
