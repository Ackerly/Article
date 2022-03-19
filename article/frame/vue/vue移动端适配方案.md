# vue 移动端完美适配方案
vue移动方案是使用amfe-flexible 和 postcss-pxtorem 结合的方式。  
amfe-flexible 是配置可伸缩布局方案，主要是将 1rem 设为 viewWidth/10。  
postcss-pxtorem是postcss的插件，用于将像素单元生成rem单位。  
**如何使用和配置**  
1. 安装 amfe-flexible 和 postcss-pxtorem  
```
npm install amfe-flexible --save
npm i postcss-pxtorem@5.1.1  --save  //这个要装5.1.1版本的
```
2. 引入  
```
import 'amfe-flexible';
```
3. postcss-pxtorem 配置  
postcss-pxtorem，可在vue.config.js、.postcssrc.js、postcss.config.js其中之一配置，权重从左到右降低，没有则新建文件，只需要设置其中一个即可,例如vue.config.js 配置的代码配置如下：
```
module.exports = {
    //...其他配置
    css: {
        loaderOptions: {
            postcss: {
                plugins: [
                    require('postcss-pxtorem')({
                        rootValue: 37.5,
                        propList: ['*']
                    })
                ]
            }
        }
    },
}
```
.postcssrc.js或postcss.config.js中配置如下：  
```
module.exports = {
    "plugins": {
        'postcss-pxtorem': {
            rootValue: 37.5,
            propList: ['*']
        }
    }
}
```
注意点：  
1. rootValue根据设计稿宽度除以10进行设置，这边假设设计稿为375，即rootValue设为37.5；
2. propList是设置需要转换的属性，这边*为所有都进行转换

通过以上配置我们就可以在项目使用了:
```
.login-form {
    width: 90%;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
    padding: 20px;
    box-sizing: border-box;
    border-radius: 10px;
    .title {
      position: absolute;
      top: -50px;
      font-size: 24px;
      color: #fff;
      left: 0;
      right: 0;
      text-align: center;
    }
  }
```
代码的产出就是下面这样的:
```
login-wraper .login-form {
    width: 90%;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    background-color: #fff;
    padding: .53333rem; // 注意这个就是转换后的单位
    box-sizing: border-box;
    border-radius: .26667rem;  // 注意这个就是转换后的单位
}

```



参考：
[https://juejin.cn/post/7008180094208311303](vue 移动端完美适配方案，拿走不谢)