# 11+ chrome高级调试技巧
## 一键重新发起请求
1. 选中Network
2. 点击Fetch/XHR
3. 选择要重新发送的请求
4. 右键选择Replay XHR

## 在控制台快速发起请求
1. 选中Network
2. 点击Fetch/XHR
3. 选择Copy as fetch
4. 控制台粘贴代码
5. 修改参数，回车搞定

## 复制JavaScript变量
使用copy函数，将对象作为入参执行即可

## 控制台获取Elements面板选中的元素
通过Elements面板选中元素后，如果想通过JS知道它的一些属性，如宽、高、位置等怎么办呢？
1. 通过Elements选择要调试的元素
2. 控制台直接用$0访问

## 截取一张全屏的网页
1. 准备好需要截屏的内容
2. cmd + shift + p 执行Command命令
3. 输入Capture full size screenshot 按下回车

**截取部分元素**  
第三步输入Capture node screenshot即可  

## 一键展开所有DOM元素
1. 按住opt键 + click（需要展开的最外层元素）

## 控制台引用上一次执行的结果
 对某个字符串进行了各种工序，然后我们想知道每一步执行的结果，该咋办？。  
 可能的操作  
 ```
 // 第1步
'fatfish'.split('') // ['f', 'a', 't', 'f', 'i', 's', 'h']
// 第2步
['f', 'a', 't', 'f', 'i', 's', 'h'].reverse() // ['h', 's', 'i', 'f', 't', 'a', 'f']
// 第3步
['h', 's', 'i', 'f', 't', 'a', 'f'].join('') // hsiftaf
 ```
 **更简单的方式**  
 使用$_引用上一次操作的结果，不用每次都复制一遍
 ```
 // 第1步
'fatfish'.split('') // ['f', 'a', 't', 'f', 'i', 's', 'h']
// 第2步
$_.reverse() // ['h', 's', 'i', 'f', 't', 'a', 'f']
// 第3步
$_.join('') // hsiftaf
 ```
 ## 快速切换主题
 1. cmd + shift + p 执行Command命令
 2. 输入Switch to dark theme或者Switch to light theme进行主题切换

## 选择器
document.querySelector和document.querySelectorAll选择当前页面的元素是最常见的需求了，不过着实有点太长了，咱们可以使用$和$$替代。  

## 在控制台安装npm包
有时候想使用比如dayjs或者lodash的某个API，但是又不想去官网查，如果可以在控制台直接试出来就好了。  
console Importer 就是这么一个插件，用来在控制台直接安装npm包。    
1. 安装Console Importer插件
2. $i('name')安装npm包

## Add conditional breakpoint条件断点的妙用
假设有下面这段代码，咱们希望食物名字是🍫时才触发断点，可以怎么弄？  
```
const foods = [
  {
    name: '🍔',
    price: 10
  },
  {
    name: '🍫',
    price: 15
  },
  {
    name: '🍵',
    price: 20
  },
]

foods.forEach((v) => {
  console.log(v.name, v.price)
})
```
这在大量数据下，只想对符合条件时打断点条件将会非常方便。试想如果没有条件断点咱们是不是要点n次debugger？  
1. console源代码在断点处右键添加条件
参考:
[11+ chrome高级调试技巧，学会效率直接提升666%](https://mp.weixin.qq.com/s/5a42BJ94McH9uNOqmkCr_w)
