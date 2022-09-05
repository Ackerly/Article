# element-ui新增组件功能解析  
## 环境准备
**克隆**  
``` 
git clone https://github.com/ElemeFE/element.git
# npm i -g yarn
cd element && npm run dev
```
**开发环境搭建**  
> 首先你需要 Node.js 4+，yarn 和 npm 3+。注意：我们使用 yarn 进行依赖版本的锁定，所以请不要使用 npm install 安装依赖

``` 
// package.json
{
    "script": {
        "bootstrap": "yarn || npm i",
        "build:file": "node build/bin/iconInit.js & node build/bin/build-entry.js & node build/bin/i18n.js & node build/bin/version.js",
        "dev": "npm run bootstrap && npm run build:file && cross-env NODE_ENV=development webpack-dev-server --config build/webpack.demo.js & node build/bin/template.js",
    },
}
```
**组件开发规范**  
> 通过 make new 创建组件目录结构，包含测试代码、入口文件、文档 如果包含父子组件，需要更改目录结构，参考 Button 组件内如果依赖了其他组件，需要在当前组件内引入，参考 Select

make 命令的配置对应根目录 Makefile，通过查看 Makefile 文件我们知道了make new命令对应的是： node build/bin.new.js  

## 主流程  
1. 文件开头判断  

``` 
'use strict';

console.log();
process.on('exit', () => {
  console.log();
});

// 第一个参数没传递报错，退出进程
if (!process.argv[2]) {
  console.error('[组件名]必填 - Please enter new component name');
  process.exit(1);
}
```

> process.argv 属性返回一个数组，由命令行执行脚本时的各个参数组成。它的第一个成员总是 node，第二个成员是脚本文件名，其余成员是脚本文件的参数。

2. 引用依赖等  

``` 
// 路径模块
const path = require('path');
// 文件模块
const fs = require('fs');
// 保存文件
const fileSave = require('file-save');
// 转驼峰
const uppercamelcase = require('uppercamelcase');
// 第一个参数 组件名
const componentname = process.argv[2];
// 第二个参数 组件中文名
const chineseName = process.argv[3] || componentname;
// 转驼峰
const ComponentName = uppercamelcase(componentname);
// package 路径
const PackagePath = path.resolve(__dirname, '../../packages', componentname);
// const Files = [];
```

3. 文件模板  
``` 
const Files = [
  {
    filename: 'index.js',
    content: `import ${ComponentName} from './src/main';

/* istanbul ignore next */
${ComponentName}.install = function(Vue) {
  Vue.component(${ComponentName}.name, ${ComponentName});
};

export default ${ComponentName};`
  },
  {
    filename: 'src/main.vue',
    content: `<template>
  <div class="el-${componentname}"></div>
</template>

<script>
export default {
  name: 'El${ComponentName}'
};
</script>`
  },
//   省略其他
];
```
3. 把 componentname 添加到 components.json
``` 
// 添加到 components.json
const componentsFile = require('../../components.json');
if (componentsFile[componentname]) {
  console.error(`${componentname} 已存在.`);
  process.exit(1);
}
componentsFile[componentname] = `./packages/${componentname}/index.js`;
fileSave(path.join(__dirname, '../../components.json'))
  .write(JSON.stringify(componentsFile, null, '  '), 'utf8')
  .end('\n');

```
4.  把 componentname.scss 添加到 index.scss
``` 
// 添加到 index.scss
const sassPath = path.join(__dirname, '../../packages/theme-chalk/src/index.scss');
const sassImportText = `${fs.readFileSync(sassPath)}@import "./${componentname}.scss";`;
fileSave(sassPath)
  .write(sassImportText, 'utf8')
  .end('\n');
```
5. 把 componentname.d.ts 添加到 element-ui.d.ts
``` 
// 添加到 element-ui.d.ts
const elementTsPath = path.join(__dirname, '../../types/element-ui.d.ts');

let elementTsText = `${fs.readFileSync(elementTsPath)}
/** ${ComponentName} Component */
export class ${ComponentName} extends El${ComponentName} {}`;

const index = elementTsText.indexOf('export') - 1;
const importString = `import { El${ComponentName} } from './${componentname}'`;

elementTsText = elementTsText.slice(0, index) + importString + '\n' + elementTsText.slice(index);

fileSave(elementTsPath)
  .write(elementTsText, 'utf8')
  .end('\n');
```
6. 把新增的组件添加到 nav.config.json
``` 
const navConfigFile = require('../../examples/nav.config.json');

Object.keys(navConfigFile).forEach(lang => {
  let groups = navConfigFile[lang][4].groups;
  groups[groups.length - 1].list.push({
    path: `/${componentname}`,
    title: lang === 'zh-CN' && componentname !== chineseName
      ? `${ComponentName} ${chineseName}`
      : ComponentName
  });
});

fileSave(path.join(__dirname, '../../examples/nav.config.json'))
  .write(JSON.stringify(navConfigFile, null, '  '), 'utf8')
  .end('\n');
  
console.log('DONE!');
```

原文: 
[每次新增页面复制粘贴？100多行源码的 element-ui 新增组件功能告诉你减少重复工作](https://juejin.cn/post/7031331765482422280)
