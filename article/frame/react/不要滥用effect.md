# 不要滥用effect
在使用useEffect时有没有发生过以下场景：  
希望状态a变化后「发起请求」，于是你使用了useEffect：  
``` 
useEffect(() => {
  fetch(xxx);
}, [a])
```
随着需求不断迭代，其他地方也会修改状态a。但是在那个需求中，并不需要状态a改变后发起请求。  
不想动之前的代码，又得修复这个bug，于是你增加了判断条件：  
``` 
useEffect(() => {
  if (xxxx) {
    fetch(xxx);
  }
}, [a])
```
某一天，需求又变化了！现在请求还需要b字段。将b作为useEffect的依赖加了进去：  
``` 
useEffect(() => {
  if (xxxx) {
    fetch(xxx);
  }
}, [a, b])
```
随着时间推移，逐渐发现:  
- 是否发送请求与if条件相关
- 是否发送请求还与a、b等依赖项相关
- a、b等依赖项又与很多需求相关

**一些理论知识**  
新文档中这一节名为Synchronizing with Effects，当前还处于草稿状态.effect这一节隶属于Escape Hatches（逃生舱）这一章  
React中有两个重要的概念：
- Rendering code（渲染代码）
- Event handlers（事件处理器）

Rendering code指开发者编写的组件渲染逻辑，最终会返回一段JSX。  
如下组件内部就是Rendering code：  
``` 
function App() {
  const [name, update] = useState('KaSong');
  
  return <div>Hello {name}</div>;
}

```
Rendering code的特点是：不带副作用的纯函数  
如下Rendering code包含副作用（count变化），就是不推荐的写法：  
``` 
let count = 0;

function App() {
  count++;
  const [name, update] = useState('KaSong');
  
  return <div>Hello {name}</div>;
}
```
**处理副作用**  
Event handlers是组件内部包含的函数，用于执行用户操作，可以包含副作用  
下面这些操作都属于Event handlers：  
- 更新input输入框
- 提交表单
- 导航到其他页面

如下例子中组件内部的changeName方法就属于Event handlers：  
``` 
function App() {
  const [name, update] = useState('KaSong');
  
  const changeName = () => {
    update('KaKaSong');
  }
  
  return <div onClick={changeName}>Hello {name}</div>;
}
```
但是，并不是所有副作用都能在Event handlers中解决  
比如，在一个聊天室中，发送消息是用户触发的，应该交给Event handlers处理。  
除此之外，聊天室需要随时保持和服务端的长连接，保持长连接的行为属于副作用，但并不是用户行为触发的。  
对于这种：在视图渲染后触发的副作用，就属于effect，应该交给useEffect处理。  
希望状态a变化后「发起请求」，首先应该明确，你的需求是状态a变化，接下来需要发起请求    
还是某个用户行为需要发起请求，请求依赖状态a作为参数  
如果是后者，这是用户行为触发的副作用，那么相关逻辑应该放在Event handlers中  
假设之前的代码逻辑是：  
1. 点击按钮，触发状态a变化
2. useEffect执行，发送请求

应该修改为：  
1. 点击按钮，在事件回调中获取状态a的值
2. 在事件回调中发送请求

参考:
[React新文档：不要滥用effect哦](https://mp.weixin.qq.com/s/a8k6QzDJEwF8SKhyKYtwyg)
