# vue规定用普通函数定义方法，为什么react又要我用箭头函数
## this指向丢失
**React中this的丢失**   
react，这是一个很普通的类组件写法：
``` 
class Demo extends React.Component{
    state = {
        someState:'state'
    }
    // ✅推荐
    arrowFunMethod = () => {
        console.log('THIS in arrow function:',this)
        this.setState({someState:'arrow state'})
    }
    // ❌需要处理this绑定
    ordinaryFunMethod(){
        console.log('THIS oridinary function:',this)
        this.setState({someState:'ordinary state'})
    }
    render(){
        return ( 
            <div>
                <h2>{this.state.someState}</h2>
                <button onClick={this.arrowFunMethod}>call arrow function</button>
                <button onClick={this.ordinaryFunMethod}>call ordinary function</button>
            </div>
        )
    }
}
ReactDOM.render(<Demo/>,document.getElementById('root'))
```
组件内我定义了两个方法：一个用箭头函数实现，另一个用普通函数。在调用时分别打印this，结果如下：  
箭头函数中this正确指向了组件实例，但普通函数中却指向了undefined
上面的代码不过是一个「类」，简化一下，就变成了这样：  
``` 
class ReactDemo {
    // ✅推荐
    arrowFunMethod = () => {
      console.log('THIS in arrow function:', this)
    }
    // ❌this指向丢失
    ordinaryFunMethod() {
      console.log('THIS in oridinary function:', this)
    }
  }
  const reactIns = new ReactDemo()
  let arrowFunWithoutCaller = reactIns.arrowFunMethod
  let ordinaryFunWithoutCaller = reactIns.ordinaryFunMethod
  arrowFunWithoutCaller()
  ordinaryFunWithoutCaller()
```
运行一下上面这段代码，会发现结果不出预料：在普通函数中this的指向也丢失了。  
从react代码运行的角度来解释一下：  
首先是事件触发时，回调函数的执行。回调函数不是像这样直接由实例调用：reactIns.ordinaryFunMethod()，而是像上面代码中的，做了一次“代理”，最后被调用时，找不到调用对象了：ordinaryFunWithoutCaller()。这时就出现了this指向undefined的情况。  
但为什么使用箭头函数，this又可以正确指向组件实例呢？首先回顾一个简单的知识点：class是个语法糖，本质不过是个构造函数，把上面的代码用它最原始的样子写出来：  
``` 
'use strict'
function ReactDemo() {
  // ✅推荐
  this.arrowFunMethod = () => {
    console.log('THIS in arrow function:', this)
  }
}
// ❌this指向丢失
ReactDemo.prototype.ordinaryFunMethod = function ordinaryFunMethod() {
  console.log('THIS in oridinary function:', this)
}
const reactIns = new ReactDemo()
```
可以看到：写成普通函数的方法，是被挂载到原型链上的；而使用箭头函数定义的方法，直接赋给了实例，变成了实例的一个属性，并且最重要的是：它是在「构造函数的作用域」被定义的。  
箭头函数没有自己的this，用到的时候只能根据作用域链去寻找最近的那个。放在这里，也就是构造函数这个作用域中的this——组件实例  
这样就可以解释为什么react组件中，箭头函数的this能正确指向组件实例  
**vue中this的丢失**  
上面的组件用vue来写一遍：  
``` 
const Demo = Vue.createApp({
  data() {
      return {
          someState:'state',
      }
  },
  methods:{
      // ❌this指向丢失
      arrowFunMethod:()=>{
          console.log('THIS in arrow function:',this)
          this.someState = 'arrow state'
      },
      // ✅推荐
      ordinaryFunMethod(){
          console.log('THIS in oridinary function:',this)
          this.someState = 'ordinary state'
      }
  },
  template:`
  <div>
      <h2>{{this.someState}}</h2>
      <button @click='this.arrowFunMethod'>call arrow function</button>
      <button @click='this.ordinaryFunMethod'>call ordinary function</button>
  </div>`
})
Demo.mount('#root')
```
运行代码，会发现结果对调了：使用箭头函数反而导致了this指向丢失：this指向了window对象  
vue会把我们传入methods遍历，再一个个赋给到组件实例上，在这个过程就处理了this的绑定（bind(methods[key], vm)）：把每一个方法中的this都绑定到组件实例上。  
普通函数都有自己的this，所以绑定完后，被调用时都能正确指向组件实例。但箭头函数没有自己的this，便无从谈及修改，它只能去找父级作用域中的this。这个父级作用域是谁呢？是组件实例吗？我们知道作用域只有两种：全局作用域和函数作用域。  
vue代码，它本质就是一个对象（具体一点，是一个组件的配置对象，这个对象里面有data、mounted、methods等属性）也就是说，我们在一个对象里面去定义方法，因为对象不构成作用域，所以这些方法的父作用域都是全局作用域。箭头函数要去寻找this，就只能找到全局作用域中的this——window对象了。  
vue对传入的方法methods对象做了处理，在函数被调用前做了this指向的绑定，只有拥有this的普通函数才能被正确的绑定到组件实例上。而箭头函数则会导致this的指向丢失。  

原文: 
[我发现了华点：vue规定用普通函数定义方法，为什么react又要我用箭头函数](https://mp.weixin.qq.com/s/iVO84pOQTRfRXahv5lQzEA)
