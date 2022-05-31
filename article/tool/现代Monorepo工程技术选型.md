# 现代Monorepo工程技术选型
**PNPM**  
PNPM 的动机,如它在官方文档介绍的所说：“Saving disk space and boosting installation speed”，节省磁盘空间和提高安装速度。除开这个动机描述的显著优点外，PNPM 内置了对 Monorepo 的支持,并解决了很多令人诟病的问题。  
比较经典的就是 Phantom dependencies（幻影依赖）。由于，默认情况下 yarn、npm 安装的依赖都是会被提升。所以，有时候你可能会遇到 Monorepo 项目中的某个包中的 package.json 没有安装这个依赖，结果实际代码中却使用了这个依赖...  
虽说，PNPM 可以解决这个问题，但是，默认情况下 PNPM 安装的依赖也是会被提升的。如果，需要 PNPM 禁止依赖提升，我们可以通过在 Monorepo 项目工作区下的 .npmrc 文件中 配置[10]，例如只提升 lodash：  
``` 
hoist-pattern[]=*lodash*
```
**Workspace 配置**  
使用 PNPM 的 Monorepo 很简单，只需要在 Monorepo 项目的工作区下新建 pnpm-workspace.yaml 文件并配置：  
``` 
packages:
  - 'packages/**'
```
## 常用依赖相关命令
**pnpm i**  
PNPM 中，安装依赖可以用 pnpm i 来完成。在 Monorepo 的场景下，默认情况下 pnpm i 会安装所有的依赖（包括 packages/*）。此外，pnpm i 还需要用到 3 个选项（Option）：  
- --filter <package>，安装依赖到指定的 package，不声明要安装的依赖包则默认安装 package.json 中的所有依赖
- --prod, P，安装依赖到 dependencies
- --dev, D，安装依赖到 devDependencies

**pnpm remove**  
在 PNPM 中，删除在 package.json 中的某个依赖，可以用 pnpm remove 完成。它的选项（Option）使用和 pnpm i 大同小异。其中，不同地是当我们在工作区想要删除 packages 中所有包的 package.json 中的某个依赖的时候，需要使用 -r，例如移除所有包中的 lodash：  
``` 
pnpm remove lodash -r
```

## Changesets
每次包（Package）的发布，需要修改 package.json 的 version 字段，以及同步更新一下本次发布修改的 CHANGELOG.md。  
这么一来，就会凸显一个问题，每次发布都需要手动地去更新 version、更新 CHANGELOG.md，未免有点繁琐。并且，用过 Lerna 的人，应该都知道 Lerna 内置了对这块的支持。  
论是 PNPM 又或者是下面要说的 Turborepo 都不支持这块，所以 2 者的官方文档都给大家推荐了用于支持这块能力的工具，例如 Changesets、Beachball、Auto等。  
Monorepo 项目中如何使用 Changesets。首先，需要执行在 Monorepo 项目的工作区下，执行如下 2 个命令：  
``` 
pnpm i -DW @changesets/cli
pnpm changeset init
```
前者是安装 Changesets 的 CLI，后者是初始化 .changeset 文件夹以及对应的文件：  
``` 
.changeset
  ｜-- config.json
  ｜__ README.md
```
config.json文件:  
``` 
{
  "$schema": "https://unpkg.com/@changesets/config@1.6.4/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "linked": [],
  "access": "restricted",
  "baseBranch": "master",
  "updateInternalDependencies": "patch",
  "ignore": []
}
```
除开 $schema 这个不需要修改的字段， config.json 文件中列了 7 个字段，各个字段分别代表的作用为：  
- changelog 设置 CHANGELOG.md 生成方式，可以设置 false 不生成，也可以设置为定义生成行为的文件地址或依赖名称，例如 Changsets 提供的 `changelog-git`[16]。其中，定义生成行为的文件固定代码模版为：
``` 
async function getReleaseLine() {}

async function getDependencyReleaseLine() {}

export default {
  getReleaseLine,
  getDependencyReleaseLine
}
```
- commit 设置是否把执行 changeset add 或 changeset publish 操作时对修改用 Git 提交
- linked 设置共享版本的包，而不是独立版本的包，例如一个组件库中主题和单独的组件的关系，也就是修改 Version 的时候，共享的包需要同步一起更新版本
- access 设置执行 npm publish 的 --access 选项，通常情况下我们是公共的包，所以设置 public 即可（注意，它会被 package.json 中的 access 字段重写）
- baseBranch 设置默认的 Git 分支，例如现在 GitHub 的默认分支应该是 main
- updateInternalDependencies 设置互相依赖的包版本更新机制，它是一个枚举（major|minor|patch），例如设置为 minor 时，只有当依赖的包新了 minor 版本或者才会对应地更新 package.json 的 dependencies 或 devDependencies 中对应依赖的版本
- ignore 设置不需要发布的包，这些会被 Changesets 忽略

在初始化 .changeset 文件夹后，就可以正常使用 changeset 相关的命令，主要是这 3 个命令：  
- pnpm chageset 用于生成本次修改的要添加到 CHANGELOG.md 中的描述
- pnpm changeset version 用于生成本次修改后的包的版本
- pnpm changeset publish 用于发布包

如果是在业务场景下，我们通常需要把包发到公司私有的 NPM Registry，而这有很多种配置方式。但是，需要注意的是 Changesets 只支持在每个包中声明 publicConfig.registry 或者配置 process.env.npm_config_registry，对应的代码会是这样：  
``` 
function getCorrectRegistry(packageJson?: PackageJSON): string {
  const registry =
    packageJson?.publishConfig?.registry ?? process.env.npm_config_registry;

  return !registry || registry === "https://registry.yarnpkg.com"
    ? "https://registry.npmjs.org"
    : registry;
}
```
如果在前面说的这 2 种情况下获取不到 registry 的话，Changesets 都是按公共的 Registry 去查找或者发布包的  
## Turborepo
Turborepo 是用于为 JavaScript/TypeScript 的 Monorepo 提供一个极快的构建系统，简单地理解就是用 Turborepo 来执行 Monorepo 项目的中构建（或者其他）任务会非常快！  
可以理解成快是选择 Turborepo 负责 Monorepo 项目多包任务执行的原因。而在 Turborepo 中执行多包任务是通过 turbo run <script>。不过，turbo run 和 lerna run 直接使用有所不同，它需要配置 turbo.json 文件，注册每个需要执行的 script 命令。  
在 Turborepo 中有个 Pipelines的概念，它是由 turbo.json 文件中的 pipline 字段的配置描述，它会在执行 turbo run 命令的时候，根据对应的配置进行有序的执行和缓存输出的文件。  
通常情况下我们一个 Monorepo 项目中的每个包可能会有 dev、build、test、clean 等 4 个命令，那么对应的 turbo.json 的配置会是这样：  
``` 
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "clean": {
      "dependsOn": ["^clean"]
    },
    "test": {
      "dependsOn": ["build"， "lint"]
    },
    "dev": {
      "cache": false
    }
  }
}
```
可以看到，pipeline 中的每个 key 则对应着每个需要执行的 turbo run 命令的名称，其中 dependsOn、outputs、cache 等 3 个字段分别作用为：  
- dependsOn 表示当前命令所依赖的命令，^ 表示 dependencies 和 devDependencies 的所有依赖都执行完 build，才执行 build
- outputs 表示命令执行输出的文件缓存目录，例如我们常见的 dist、coverage 等
- cache 表示是否缓存，通常我们执行 dev 命令的时候会结合 watch 模式，所以这种情况下关闭掉缓存比较切合实际需求

这样一来，就可以使用诸如 turbo run build test 的命令，它则会按 pipeline 的配置依次执行对应的命令。
当然，如果你想每个命令都支持单独执行，可以直接配置为 {} 即可。此外，如果要使用 turbo run 命令，还需要在 package.json 中声明 packageManage 字段为指定的包管理工具及版本，例如 "packageManager": "pnpm@6.30.0"。

参考:    
[现代 Monorepo 工程技术选型，聊聊我的思考](https://mp.weixin.qq.com/s/-6OSeoJUlzI7qkkcDrYNcQ)
