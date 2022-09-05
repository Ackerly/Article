# 为什么说移动端该放弃rem适配方案了
在做移动端各种尺寸屏幕的适配时，用的最多的就是rem方案。我们都写过这样的代码,来设置根字体大小。这个计算公式中设备宽度，UI设计稿尺寸这两个参数比较好理解，可是为什么要除以100呢，为什么不是10,50或者其它的数值呢。  
``` 
const setRem = () => {
  const deviceWidth = document.documentElement.clientWidth;
  // 获取相对UI稿，屏幕的缩放比例
  const rem = (deviceWidth *100) / 750;
  // 动态设置html的font-size
  document.querySelector('html').style.fontSize =  rem + 'px';
};
```
rem方案是viewport的过渡方案，将设计稿除以100，等分为7.5份来实现移动端不同屏幕尺寸适配的原理，与viewport中vw单位的定义，设计思想与想要解决的问题，是相同的。当时浏览器对viewport的支持性不好，而现在已经是2022年了，可以看到，各大浏览器厂商，对viewport的支持率已经很高了。可以放心使用。  

**相对于rem的优势**  
- 语义化更好, rem从本义上来说，是一种字体单位，不是用来做布局和各种屏幕尺寸大小适配的，如上面的示例，用rem做适配单位，计算根字体的时候，计算公式中的100这个参数让人感觉很费解，viewport词更达意
- 可以直接在代码中书写px,借助postcss-px-to-viewport插件转换成vw单位，完美适配移动端各种屏幕尺寸。不用像之前那样，一是要在蓝湖上设置根字体基准尺寸，将设计稿标注的px单位转换成rem单位，然后摘抄到代码中。二是需要用js计算设置根字体大小。前端开发天然喜欢px单位，像rem,em,vw，vh这些单位，一般都不是UI设计稿标注的尺寸，开发时需要转换成本。不如直接在代码中写px直观高效。

**postcss-px-to-viewport方案正确的使用姿势**  
1. 安装postcss-px-to-viewport-8-plugin
``` 
yarn add -D postcss-px-to-viewport-8-plugin
```
2. 在项目下创建postcss.config.js
``` 
module.exports = {
  plugins: {
    'postcss-px-to-viewport-8-plugin': {
      unitToConvert: 'px', // 需要转换的单位，默认为"px"
      viewportWidth: 750, // 设计稿的视口宽度
      unitPrecision: 5, // 单位转换后保留的精度
      propList: ['*','!font-size'], // 能转化为vw的属性列表,!font-size表示font-size后面的单位不会被转换
      viewportUnit: 'vw', // 希望使用的视口单位
      fontViewportUnit: 'vw', // 字体使用的视口单位
      // 需要忽略的CSS选择器，不会转为视口单位，使用原有的px等单位。
      // 下面配置表示类名中含有'keep-px'都不会被转换
      selectorBlackList: ['keep-px'],
      minPixelValue: 1, // 设置最小的转换数值，如果为1的话，只有大于1的值会被转换
      mediaQuery: false, // 媒体查询里的单位是否需要转换单位
      replace: true, //  是否直接更换属性值，而不添加备用属性
      exclude: [/node_modules/], // 忽略某些文件夹下的文件或特定文件，例如 'node_modules' 下的文件
      include: [/src/], // 如果设置了include，那将只有匹配到的文件才会被转换
      landscape: false, // 是否添加根据 landscapeWidth 生成的媒体查询条件 @media (orientation: landscape)
      landscapeUnit: 'vw', // 横屏时使用的单位
      landscapeWidth: 1338, // 横屏时使用的视口宽度
    },
  },
};
```

需要注意的是：  
1. postcss-px-to-viewport 对内联css样式，外联css样式，内嵌css样式有效，对js动态css无效。所以要动态改变css展示效果的话，要使用静态的class定义变化样式，通过js改变dom元素的class实现样式变化
2. Vant组件的设计稿尺寸是375px，可用通过覆盖:root下的Vant的css变量中px单位的方式，对Vant组件做适配
3. vue模板中的px单位不会被转换，如需转换请使用postcss-style-px-to-viewport工具

原文: 
[为什么说移动端该放弃rem适配方案了](https://mp.weixin.qq.com/s/BPngb4QuPW8JJ6jquPp0tw)
