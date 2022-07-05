# 16道TypeScript题目  
## 实现 Pick
从类型 T 中选择出属性 K，构造成一个新的类型。  
例如：  
``` 
interface Todo {
  title: string
  description: string
  completed: boolean
}
  
type TodoPreview = MyPick<Todo, 'title' | 'completed'>
  
const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
}
```
解答
``` 
type MyPick<T, K extends keyof T = keyof T> = {
  [P in K]: T[P]
}
```



参考:  
[你以为学会TypeScript了？先看看这16道题能做对多少先](https://juejin.cn/post/7110232056826691591)
