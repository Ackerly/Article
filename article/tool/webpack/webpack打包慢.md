# Webpack打包慢处理方式
## 分析项目，对大的库分离出来
1， webpack-bundle-analyzer对项目进行模块分析生成report，查看report后看看那些模块体积过大，然后针对性优化，比如项目中引入常用的UI库element-ui和v-charts等
2. 配置webpack的externals，官方文档的解释：防止将某些import的包（package）打包到bundle中，而是运行时（runtime）再去从外部获取这些扩展依赖，所以可以将体积大的库分离出来：
``` 
// ...
externals: {
    'element-ui': 'Element',
    'v-charts': 'VCharts'
}
```
3.在main.js中移除相关库的import
4.在index.html模板文件中，添加相关库的cdn引用，如：
``` 
<script src="https://unpkg.com/element-ui@2.10.0/lib/index.js"></script>
<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/v-charts/lib/index.min.js"></script>
```
缺点：不能按需加载，会将配置的第三方全部打包进去
## loader范围缩小到src项目文件!一些不必要的loader能关就关
## 优化eslint代码校验
eslint是一个很费时间的步骤，把eslint范围缩写到src且只检查*.js和*.vue.  
生产环境不开启lint，使用pre-commit或者husky在提交前校验
## happypack多线程进行
## HardSourceWebpackPlugin会将模块编译后进行缓存，第一次后速度明显提升
## 使用插件直接拷贝静态文件
## 使用更合理的代码压缩插件
因webpack提供的UglifyJS插件采用单线程压缩，速度很慢，所以将插件替换为webpack-parallel-uglify-plugin插件，此插件可以并行运行UglifyJs插件，可以有效减少构建时间
## 减小文件搜索范围
优化loader配置：include、exclude  
优化module.noParse配置：忽略对部分没采用模块化的文件的递归解析处理  
优化resolve.modules配置：去那些目录下寻找第三方模块  
优化resolve.alias配置  
优化resolve.mainFields配置  
优化resolve.extensions配置：配置在尝试匹配过程中用到的后缀列表
## 使用alias可以更快地找到对应文件
## require模块时不写后缀名，默认webpack会尝试js，json等后缀名匹配，配置extensions，可以webpack少做一点后缀匹配。
## thread-loader可以将非常消耗资源的loaders转存到worker pool中
## 使用cache-loader启用持久化缓存。使用package.json中的postinstall清除缓存记录
## 使用gzip压缩
