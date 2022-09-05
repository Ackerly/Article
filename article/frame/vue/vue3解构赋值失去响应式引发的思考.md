# Vue3解构赋值失去响应式引发的思考
proxy 实现响应式的能力， 解决了vue2所遗留下来的一些问题，同时也正由于proxy的特性，也提高了运行时的性能  
但是它也有本身的局限，从而产生一些弊端：  
- 原始值的响应式系统的实现 导致必须将他包装为一个对象， 通过 .value 的方式访问
- ES6 解构，不能随意使用。会破坏他的响应式特性

**原始值的响应式系统的实现**  
``` 
const obj = {
  name: 'win'
}

const handler = {
  get: function(target, key){
    console.log('get--', key)
    return Reflect.get(...arguments)  
  },
  set: function(target, key, value){
    console.log('set--', key, '=', value)
    return Reflect.set(...arguments)
  }
}

const data = new Proxy(obj, handler)
data.name = 'ten'
console.log(data.name,'data.name22')
```
proxy 的使用本身就是对于 对象的拦截， 通过new Proxy 的返回值，拦截了obj 对象  
如此一来，当你访问对象中的值的时候，它会触发 get 方法， 当你修改对象中的值的时候 它会触发 set方法  
但是到了原始值的时候，它没有对象啊，new proxy 排不上用场了  
只能包装一下了，所以就有了使用.value访问了  
``` 
import { reactive } from "./reactive";
import { trackEffects, triggerEffects } from './effect'

export const isObject = (value) => {
    return typeof value === 'object' && value !== null
}

// 将对象转化为响应式的
function toReactive(value) {
    return isObject(value) ? reactive(value) : value
}

class RefImpl {
    public _value;
    public dep = new Set; // 依赖收集
    public __v_isRef = true; // 是ref的标识
    // rawValue 传递进来的值
    constructor(public rawValue, public _shallow) {
        // 1、判断如果是对象 使用reactive将对象转为响应式的
        // 浅ref不需要再次代理
        this._value = _shallow ? rawValue : toReactive(rawValue);
    }
    get value() {
        // 取值的时候依赖收集
        trackEffects(this.dep)
        return this._value;
    }
    set value(newVal) {
        if (newVal !== this.rawValue) {
            // 2、set的值不等于初始值 判断新值是否是对象 进行赋值
            this._value = this._shallow ? newVal : toReactive(newVal);
            // 赋值完 将初始值变为本次的
            this.rawValue = newVal
            triggerEffects(this.dep)
        }
    }
}
```
对于原始值的包装，它被包装为一个对象，通过get value 和set value 方法来进行原始值的访问，从而导致必须有.value 的操作
**proxy实现原理**  
``` 
const obj = {
    count: 1
};
const proxy = new Proxy(obj, {
    get(target, key, receiver) {
        console.log("这里是get");
        return Reflect.get(target, key, receiver);
    },
    set(target, key, value, receiver) {
        console.log("这里是set");
        return Reflect.set(target, key, value, receiver);
    }
});

console.log(proxy)
console.log(proxy.count)
```
通过和Reflect 的配合， 就能实现对于对象的拦截,实现响应式,但是对象在嵌套深一层  
``` 
const obj = {
    count: 1,
    b: {
        c: 2
    }
};


console.log(proxy.b)
console.log(proxy.b.c)
```
上面的无法拦截，必须要来个包装
``` 
const obj = {
    a: {
        count: 1
    }
};

function reactive(obj) {
    return new Proxy(obj, {
        get(target, key, receiver) {
            console.log("这里是get");
            // 判断如果是个对象在包装一次，实现深层嵌套的响应式
            if (typeof target[key] === "object") {
                return reactive(target[key]);
            };
            return Reflect.get(target, key, receiver);
        },
        set(target, key, value, receiver) {
            console.log("这里是set");
            return Reflect.set(target, key, value, receiver);
        }
    });
};
const proxy = reactive(obj);
```
失去响应式的几个情况：
1. 解构 props 对象，因为它会失去响应式
2. 直接赋值reactive响应式对象
3. vuex中组合API赋值

解构 props 对象，因为它会失去响应式  
``` 
const obj = {
    a: {
        count: 1
    },
    b: 1
};
    
    //reactive 是上文中的reactive
   const proxy = reactive(obj);
const {
    a,
    b
} = proxy;
console.log(a)
console.log(b)
console.log(a.count)
```
解构赋值，b 不会触发响应式，a如果你访问的时候，会触发响应式   
解构赋值，区分原始类型的赋值，和引用类型的赋值，原始类型的赋值相当于按值传递， 引用类型的值就相当于按引用传递,相当于
``` 
// 假设a是个响应式对象
  const a={ b:1}
  // c 此时就是一个值跟当前的a 已经不沾边了
  const c=a.b

// 你直接访问c就相当于直接访问这个值 也就绕过了 a 对象的get ，也就像原文中说的失去响应式
```
因为a 是引用类型，proxy判断读取对象如果他是个object 那么就重新包装为响应式  
由于当前特性，导致，如果是引用类型， 你再去访问其中的内容的时候并不会失去响应式  

直接赋值reactive响应式对象  
``` 
const vue = reactive({ a: 1 })
 vue = { b: 2 }
```
然后就发出疑问reactive不是响应式的吗？为啥赋值了以后，他的响应式就没了  
由于解构赋值的问题， 直接禁止了reactive的解构赋值,你用解构赋值操作的时候直接禁用了  
但是为什么props不给禁用呢？因为props 的数据可能不是响应式
原生js 的引用类型的赋值，其实是 按照引用地址赋值  
``` 
// 当reactive 之后返回一个代理对象的地址被vue 存起来，
 // 用一个不恰当的比喻来说，就是这个地址具备响应式的能力
 const vue = reactive({ a: 1 })
 
//  而当你对于vue重新赋值的时候不是将新的对象赋值给那个地址，而是将vue 换了个新地址
// 而此时新地址不具备响应式，可不就失去响应式了吗
vue = { b: 2 }
```
**vuex中组合API赋值**  
``` 
import { computed } from 'vue'
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()
    return {
      // 在 computed 函数中访问 state
      count: computed(() => store.state.count),

      // 在 computed 函数中访问 getter
      double: computed(() => store.getters.double)
    }
  }
}
```
store.getters.double 必须用computed 包裹起来，其实道理是一样的，也是变量赋值的原因

原文: 
[Vue3 解构赋值失去响应式引发的思考](https://mp.weixin.qq.com/s/es2Mwk3_PHUUz869O5ivAA)
