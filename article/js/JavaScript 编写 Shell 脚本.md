# JavaScript 编写 Shell 脚本
**Shell**  
Shell 的解释器种类众多，常见的有：  
- sh - 即 Bourne Shell。sh 是 Unix 标准默认的 shell
- bash - 即 Bourne Again Shell。bash 是 Linux 标准默认的 shell
- fish - 智能和用户友好的命令行 shell
- xiki - 使 shell 控制台更友好，更强大
- zsh - 功能强大的 shell 与脚本语言

一般在 shell 脚本的开头，#! 告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 解释器。#! 被称作 shebang  

指定 sh 解释器
``` 
#!/bin/sh
```
指定 bash 解释器  
``` 
#!/bin/bash
```

可以用 Node.js 执行一些简单的 Shell 命令：  
``` 
const { execSync } = require("child_process");

exec('git diff orgin/master', (err, data) => {
  if (err) {
    console.log("失败", err);
    process.exit(1);
  } else {
    console.log("成功", data);
  }
});
```
但是这个体验和直接写 Shell 脚本相比就比较差了，需要手动用 child_process 进行包装、每次引入一些额外的依赖库、异常处理也比较麻烦、另外还要考虑转译命令行参数。  
zx 对 child_process 进行了默认包装，对参数进行了转译而且提供了合理的默认值。可以很方便的让我们使用前端熟悉的 JavaScript 语法来编写 Shell 脚本：  
``` 
#!/usr/bin/env zx

await $`cat package.json | grep name`

let branch = await $`git branch --show-current`
await $`dep deploy --branch=${branch}`

await Promise.all([
  $`sleep 1; echo 1`,
  $`sleep 2; echo 2`,
  $`sleep 3; echo 3`,
])

let name = 'foo bar'
await $`mkdir /tmp/${name}`
```
**使用**  
安装（要求 Node.js 版本 >= 16.0.0）：  
``` 
npm i -g zx
```
将脚本写到 .mjs 的文件里，这样可以很方便的直接在顶层使用 await，然后在文件开头声明下面的 shebang：  
``` 
#!/usr/bin/env zx
```
通过下面的方式运行脚本：  
``` 
chmod +x ./script.mjs
./script.mjs
```
使用 zx 运行：  
``` 
zx ./script.mjs
```
可以尝试一下：  
``` 
const list = await $`ls -a`;

console.log(list);

const name = await question('你的名字是啥? ')

console.log(`你的名字是：${name}`);
```
> 所有函数（$、cd、fetch等）都可以直接使用，无需任何导入

它还内置了很多方便的处理函数：  
- $command：使用 child_process 的 spawn 来制定指定的命令，返回一个 Promise
- cd()：进入其他目录。（cd('/project')）
- fetch()：发起方洛请求
- question()：读取用户输入，相当于 readline 的封装
- sleep()：等待一段时间，相当于 setTimeout 的封装
- echo()：大打印文本，也可以直接用 console.log

原文: 
[JavaScript 编写 Shell 脚本](https://mp.weixin.qq.com/s/e__82YNQD9NlUizTqTVuyw)
