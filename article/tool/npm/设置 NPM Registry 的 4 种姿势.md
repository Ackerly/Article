# 设置 NPM Registry 的 4 种姿势
**1. Global registry**  
通过设置 npm 或者 pnpm 的 config 来设置 Global Registry，例如  
``` 
# npm
npm config set registry=http://localhost:2000

# or pnpm
pnpm config set registry=http://localhost:2000
```
这样一来，在代码层面就可以通过 process.env.npm_config_registry 读取到这里的配置。  
**2. npmrc**  
无论是 npm 或 pnpm 默认都会从项目的 .npmrc 文件中读取配置，所以当我们需要包的发布要走私有 Registry 的时候，可以这样设置：  
``` 
registry = http://localhost:2000
```
**3. --registry**  
执行 npm publish 或 pnpm publish 的时候，可以通过带上 --registry Option 来告知对应的包管理工具要发布的 Registry，例如：  
``` 
# npm
npm publish --registry=http://localhost:2000

# or pnpm
pnpm publish --registry=http://localhost:2000
```
**4. PublishConfig**  
PublishConfig指的是我们可以在要执行 publish 命令的项目 package.json 中的 publishConfig.registry 来告知 npm 或 pnpm 要发布的 Registry，例如：  
``` 
{
  ...
  "publishConfig": {
    "registry": "http://localhost:2000"
  }
  ...
}
```
**Changeset publish 原理**  
如何让 pnpm changeset publish 知道要把包发布到指定的私有 Registry？  
pnpm changeset publish 命令的本质是执行 changeset 的 publish命令。那么，也就是上面的 4 种设置 Registry 的方式，很可能不是每种都生效的，因为 Changeset 有一套自己的 publish 机制。而这个过程它主要会做这 3 件事：
1. 首先，获取 Package Info，它会从指定的 Registry 获取 Package Info。举个例子，如果是获取 rollup 的 Package Info，那么在这里会是这样： 
``` 
npm info rollup --registry="https://registry.npmjs.org/" --json
```
2. 其次，根据上面拿到的 Package Info 中的 versions 字段（它是一个包含所有已发布的版本的数组），对比本地的 package.json 的 version 字段，判断当前版本是否已发布
3. 如果未发布当前版本，则会根据当前使用的包管理工具执行 publish 命令，并且此时会构造一个它认为应该发布的 Registry 地址，然后重写 env 上的配置，对应的代码会是这样：  
``` 
// packages/cli/src/commands/publish/npm-utils
const envOverride = {
  npm_config_registry: getCorrectRegistry()
};
let { code, stdout, stderr } = await spawn(
  // 动态的包管理工具，例如 pnpm、npm、yarn
  publishTool.name,
  ["publish", opts.cwd, "--json", ...publishFlags],
  {
    env: Object.assign({}, process.env, envOverride)
  }
);
```

整个 changeset publish 的过程还是很简单易于理解的。并且，非常重要的是这个过程牵扯到 Registry 获取的都是由一个名为 getCorrectRegistry() 的函数完成的，它的定义会是这样：  
``` 
// packages/cli/src/commands/publish/npm-utils.ts
function getCorrectRegistry(packageJson?: PackageJSON): string {
  const registry =
    packageJson?.publishConfig?.registry ?? process.env.npm_config_registry;

  return !registry || registry === "https://registry.yarnpkg.com"
    ? "https://registry.npmjs.org"
    : registry;
}
```
在 Changeset 中只支持了 publishConfig 或 env 配置 Registry 的方式，所以如果你尝试其他 2 种方式就会 publish 到 https://registry.yarnpkg.com 或 https://registry.npmjs.org，并且在第一步获取 Package Info 的时候可能就会失败。  

参考:
[设置 NPM Registry 的 4 种姿势](https://mp.weixin.qq.com/s/MYLi4mSgoi5KXj4-_OgT3A)
