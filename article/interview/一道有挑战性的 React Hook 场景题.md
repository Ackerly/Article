# 一道有挑战性的 React Hook 场景题
有一个 button 和一个 list：  
``` 
<div className="App">
  <button onClick={add}>Add</button>
  {list.map(val => val)}
</div>
```
list 是使用 useState 管理的状态。button 绑定了事件 onClick={add}。点击按钮，会执行 add 方法向 list 中加入一些内容。  
``` 
export default function App() {
  const [list, setList] = useState([]);

  const add = () => {
    // ...
  };

  return (
    <div className="App">
      <button onClick={add}>Add</button>
      {list.map(val => val)}
    </div>
  );
}
```
在 App 外部定义变量 i  
``` 
let i = 0;

export default function App() {
  // ...
}
```
调用 add，会向 list 中添加新的 button，新 button 也绑定了 onClick={add}。  
``` 
const add = () => {
  setList(
    list.concat(
      <button 
        key={i} 
        onClick={add}>
        {i++}
      </button>
    )
  );
};
```
**问题**  
点击这些「数字按钮」，页面会怎么展示呢  
- 比如点击 0，页面会如何展示，list 最终结果是什么
- 点击 6，又会如何展示

**解答**  
真正的现象是，点击数字按钮后：  
- 列表的长度将会变成 点击的数字 + 1
- 并且列表最后一个数字会变成 点击之前最大的数字 + 1

**原理剖析**  
造成这种反直觉现象的原因有两个：  
1. hook 闭包问题
2. state 更新机制

点击按钮会调用的 add 函数：  
``` 
const add = () => {
  setList(
    list.concat(
      <button 
        key={i} 
        onClick={add}>
        {i++}
      </button>
    )
  );
};
```
当执行 add 函数时，由于访问了外层函数 App 内的变量，所以会根据 App 函数上下文形成闭包，闭包内包括：
- add 函数
- list 变量
- setList 方法

list 和 setList 是调用 useState() 返回的。  
这里通常有一个误解：多次调用 useState，返回的 list 都是同一个对象。  
实际上，useState 返回的 list 都是基于 base state 计算出来的：  
每次会将上一次的 prev state 与 update 进行合并得到新的 current state。因此，每次调用 useState 返回的 list 都不是同一个对象，它们的内存地址不同。  
这会导致每个「数字按钮」的 add 函数处于不同的闭包中，每个闭包当中的 list 都不同。  
而变量 i 是声明在 App 外层的模块级变量，在每个闭包中 i 都是相同的。  
``` 
let i = 0;

export default function App() {
  // ...
}
```
所以，在点击 0 时：  
- i 是模块级变量，值为 6
- list 是闭包中的变量，值为 []

add 函数实际上执行的是：  
``` 
setList(
  [].concat(
    <button key={7} onClick={add}>{7}</button>
  )
);
```
所以 list 最终变成了 [7]。  
当点击 2 时：  
- i 是模块级变量，值为 6
- list 是闭包中的变量，值为 [0,1]

add 函数实际上执行的是：  
``` 
setList(
  [0, 1].concat(
    <button key={7} onClick={add}>{7}</button>
  )
);
```
所以 list 最终变成了 [0, 1, 7]。  

**如何解决**  
只要将 list 从闭包中清理出去就可以了，将 setList 参数改为函数形式。  
``` 
setList(list => 
  list.concat(
    <button 
      key={i} 
      onClick={add}>
      {i++}
    </button>
  )
);
```
这样，点击「Add 按钮」或任意「数字按钮」都会正常在 list 后面拼接新按钮。  
**总结**  
state 的更新机制是：  
``` 
current state = base state + update1 + update2 + …
```
所以每次调用 useState，返回值 list 都是不同的对象。  
并且由于闭包的存在，每个「数字按钮」add 函数中的 list 都不同。

原文: 
[一道有挑战性的 React Hook 场景题，考考你的功底](https://mp.weixin.qq.com/s/nvcuZuoPmmeQ0vgDk4-3CA)
