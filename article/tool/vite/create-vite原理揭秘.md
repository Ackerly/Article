# create-vite原理
**npm init && npm create**  
reate 其实就是 init 的一个别名,也就是说 npm create vite@lastest 相当于 => npx create-vite@lastest，latest 是版本号，目前最新版本可以通过以下命令查看。  
```
npm dist-tag ls create-vite
```

**克隆项目 && 调试源码**  
```
git clone https://github.com/lxchuan12/vite-analysis.git
cd vite-analysis/vite2
# npm i -g pnpm
pnpm install
# 在这个 index.js 文件中断点
# 在命令行终端调试
node vite2/packages/create-vite/index.js
```
使用的git subtree保持 vite 记录  
```
# 创建
git subtree add --prefix=vite2 https://github.com/vitejs/vite.git main
# 更新
git subtree pull --prefix=vite2 https://github.com/vitejs/vite.git main
```
找到路径，packages/create-vite 看 package.json  
```
{
  "name": "create-vite",
  "version": "3.0.0",
  "type": "module",
  "bin": {
    "create-vite": "index.js",
    "cva": "index.js"
  },
  "main": "index.js",
  "engines": {
    "node": "^14.18.0 || >=16.0.0"
  },
}
```
type 类型指定为 module 说明是 ES Module。bin 可执行命令为 create-vite 或 别名 cva。可以知道主文件 index.js。代码限制了较高版本的Nodejs。
**主流程 init 函数拆分**
```
// 高版本的node支持，node 前缀
import fs from 'node:fs'
import path from 'node:path'
import { fileURLToPath } from 'node:url'

// 解析命令行的参数 链接：https://npm.im/minimist
import minimist from 'minimist'
// 询问选择之类的  链接：https://npm.im/prompts
import prompts from 'prompts'
// 终端颜色输出的库 链接：https://npm.im/kolorist
import {
  blue,
  cyan,
  green,
  lightRed,
  magenta,
  red,
  reset,
  yellow
} from 'kolorist'

// Avoids autoconversion to number of the project name by defining that the args
// non associated with an option ( _ ) needs to be parsed as a string. See #4606
const argv = minimist(process.argv.slice(2), { string: ['_'] })
// 当前 Nodejs 的执行目录
const cwd = process.cwd()

// 主函数内容省略，后文讲述
async function init() {}
init().catch((e) => {
  console.error(e)
})
```
输出的目标路径  
```
// 命令行第一个参数，替换反斜杠 / 为空字符串
let targetDir = formatTargetDir(argv._[0])

// 命令行参数 --template 或者 -t
let template = argv.template || argv.t

const defaultTargetDir = 'vite-project'
// 获取项目名
const getProjectName = () =>
targetDir === '.' ? path.basename(path.resolve()) : targetDir
```
**延伸函数 formatTargetDir**  
替换反斜杠 / 为空字符串  
```
function formatTargetDir(targetDir) {
  return targetDir?.trim().replace(/\/+$/g, '')
}
```
_prompts 询问项目名、选择框架，选择框架变体等,prompts根据用户输入选择_  
```
let result = {}
try {
    result = await prompts(
      [
        {
          type: targetDir ? null : 'text',
          name: 'projectName',
          message: reset('Project name:'),
          initial: defaultTargetDir,
          onState: (state) => {
            targetDir = formatTargetDir(state.value) || defaultTargetDir
          }
        },
        // 省略若干
      ],
      {
        onCancel: () => {
          throw new Error(red('✖') + ' Operation cancelled')
        }
      }
      )
} catch (cancelled) {
    console.log(cancelled.message)
    return
}
// user choice associated with prompts
// framework 框架
// overwrite 已有目录，是否重写
// packageName 输入的项目名
// variant 变体， 比如 react => react-ts
const { framework, overwrite, packageName, variant } = result
```
_重写已有目录/或者创建不存在的目录_  
```
// user choice associated with prompts
const { framework, overwrite, packageName, variant } = result

// 目录
const root = path.join(cwd, targetDir)

if (overwrite) {
    // 删除文件夹
    emptyDir(root)
} else if (!fs.existsSync(root)) {
    // 新建文件夹
    fs.mkdirSync(root, { recursive: true })
}
```
函数 emptyDir,递归删除文件夹，相当于 rm -rf xxx。  
``` 
function emptyDir(dir) {
  if (!fs.existsSync(dir)) {
    return
  }
  for (const file of fs.readdirSync(dir)) {
    fs.rmSync(path.resolve(dir, file), { recursive: true, force: true })
  }
}
```
_获取模板路径_  
``` 
template = variant || framework || template

console.log(`\nScaffolding project in ${root}...`)

const templateDir = path.resolve(
    fileURLToPath(import.meta.url),
    '..',
    `template-${template}`
)
```
_写入文件函数_  
``` 
const write = (file, content) => {
    // renameFile
    const targetPath = renameFiles[file]
        ? path.join(root, renameFiles[file])
        : path.join(root, file)
    if (content) {
        fs.writeFileSync(targetPath, content)
    } else {
        copy(path.join(templateDir, file), targetPath)
    }
}
```
这里的 renameFiles，是因为在某些编辑器或者电脑上不支持.gitignore  
``` 
const renameFiles = {
  _gitignore: '.gitignore'
}

```
copy && copyDir拷贝文件和文件夹  
``` 
function copy(src, dest) {
  const stat = fs.statSync(src)
  if (stat.isDirectory()) {
    copyDir(src, dest)
  } else {
    fs.copyFileSync(src, dest)
  }
}

/**
 * @param {string} srcDir
 * @param {string} destDir
 */
function copyDir(srcDir, destDir) {
  fs.mkdirSync(destDir, { recursive: true })
  for (const file of fs.readdirSync(srcDir)) {
    const srcFile = path.resolve(srcDir, file)
    const destFile = path.resolve(destDir, file)
    copy(srcFile, destFile)
  }
}
```
_根据模板路径的文件写入目标路径_  
package.json 文件单独处理。它的名字为输入的 packageName 或者获取  
``` 
const files = fs.readdirSync(templateDir)
for (const file of files.filter((f) => f !== 'package.json')) {
    write(file)
}

const pkg = JSON.parse(
    fs.readFileSync(path.join(templateDir, `package.json`), 'utf-8')
)

pkg.name = packageName || getProjectName()

write('package.json', JSON.stringify(pkg, null, 2))
```
**打印安装完成后的信息**  
``` 
const pkgInfo = pkgFromUserAgent(process.env.npm_config_user_agent)
const pkgManager = pkgInfo ? pkgInfo.name : 'npm'

console.log(`\nDone. Now run:\n`)
if (root !== cwd) {
    console.log(`  cd ${path.relative(cwd, root)}`)
}
switch (pkgManager) {
    case 'yarn':
        console.log('  yarn')
        console.log('  yarn dev')
        break
    default:
        console.log(`  ${pkgManager} install`)
        console.log(`  ${pkgManager} run dev`)
        break
}
console.log()
```

延伸的 pkgFromUserAgent 函数  
``` 
/**
 * @param {string | undefined} userAgent process.env.npm_config_user_agent
 * @returns object | undefined
 */
function pkgFromUserAgent(userAgent) {
  if (!userAgent) return undefined
  const pkgSpec = userAgent.split(' ')[0]
  const pkgSpecArr = pkgSpec.split('/')
  return {
    name: pkgSpecArr[0],
    version: pkgSpecArr[1]
  }
}
```
第一句 pkgFromUserAgent函数，是从使用了什么包管理器创建项目，那么就输出 npm/yarn/pnpm 相应的命令。  
``` 
npm create vite@lastest
yarn create vite
pnpm create vite
```



原文:  
[create-vite 原理揭秘](https://mp.weixin.qq.com/s/eqIcLlh3a4TUwx3lk2qLYA)
