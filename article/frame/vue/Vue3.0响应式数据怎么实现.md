# Vue3.0响应式数据怎么实现
##  Proxy使用
Proxy的定位就是target对象(原对象)的基础上通过handler增加一层”拦截“，返回一个新的代理对象，之后所有在Proxy中被拦截的属性，都可以定制化一些新的流程在上面，
``` 
const target = {}; // 要被代理的原对象
// 用于描述代理过程的
handlerconst handler = {  
    get: function (target, key, receiver) {    
        console.log(`getting ${key}!`);    
        return Reflect.get(target, key, receiver);  
    },  
    set: function (target, key, value, receiver) {    
        console.log(`setting ${key}!`);    
        return Reflect.set(target, key, value, receiver);  
    }
    }
    // obj就是一个被新的代理对象
    const obj = new Proxy(target, handler);obj.a = 1 // setting a!
    console.log(obj.a) // getting a!
```
注意:
- 对Proxy对象的赋值操作也会影响到原对象target，同时对target的操作也会影响Proxy，不过直接操作原对象的话不会触发拦截的内容
``` 
obj.a = 1; // setting a!
console.log(target.a) // 1 不会打印 "getting a!"
```
- 如果handler中没有任何拦截上的处理，那么对代理对象的操作会直接通向原对象
``` 
const target = {};
const handler = {};
const obj = new Proxy(target, handler);
obj.a = 1;
console.log(target.a) // 1
```
既然proxy也是一个对象，那么它就可以做为原型对象，可以把obj的原型指向到proxy上后，对obj的操作会找到原型上的代理对象，如果obj自己有a属性，则不会触发proxy上的get，
``` 
const target = {};
const obj = {};
const handler = {    
    get: function(target, key){            
        console.log(`get ${key} from ${JSON.stringify(target)}`);            
        return Reflect.get(target, key);
    }
}
const proxy = new Proxy(target, handler);
Object.setPrototypeOf(obj, proxy);
proxy.a = 1;
obj.b = 1console.log(obj.a) // get a from {"a": 1}   
1console.log(obj.b) // 1
```
## Proxy是想对那些属性的拦截
1. get(target, propKey, receiver)：拦截对象属性的读取，比如proxy.foo和proxy['foo'];
2. set(target, propKey, value, receiver)：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值;
3. has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值。
4. deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值;
5. ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for…in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性;
6. getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象;
7. defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值;
8. preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值;
9. getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象;
10. isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值;
11. setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截;
12. apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如proxy(…args)、proxy.call(object, …args)、proxy.apply(…);
13. construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(…args);
## Proxy实际场景的应用
js的语法中没有private这个关键字来修饰私有变量，所以基本上所有的class的属性都是可以被访问的，但是在有些场景下我们需要使用到私有变量，现在业界的一些做法都是使用”_变量名“来”约定“这是一个私有变量，但是如果哪天被别人从外部改掉的话，还是没有办法阻止的，然而，当Proxy出现后，可以用代理来处理这种场景，看代码：
``` 
const obj = {    
    _name: 'nanjin',    
    age: 19,    
    getName: () => {        
        return this._name;    
    },    
    setName: (newName) => {        
        this._name = newName;    
        }
}
const proxyObj = obj => new Proxy(obj, {    
    get: (target, key) => {        
        if(key.startsWith('_')){            
            throw new Error(`${key} is private key, please use get${key}`)        
        }        
        return Reflect.get(target, key);    
    },    
    set: (target, key, newVal) => {        
        if(key.startsWith('_')){            
            throw new Error(`${key} is private key, please use set${key}`)        
        }        
        return Reflect.set(target, key, newVal);    
    }
})
const newObj = proxyObj(obj);console.log(newObj._name) // Uncaught Error: _name is private key,please use get_name
newObj._name = 'newname'; // Uncaught Error: _name is private key, please use set_name
console.log(newObj.age) // 19
console.log(newObj.getName()) // nanjin
```
## Vue响应式数据实现
### 2.x版本
vue会利用Object.defineProperty来递归的给data中的数据加上get和set，然后每次set的时候，加入额外的逻辑。来触发对应模板视图的更新，看下伪代码：
``` 
const defineReactiveData = data => {    
    Object.keys(data).forEach(key => {        
        let value = data[key];        
        Object.defineProperty(data, key, {         
            get : function(){            
                console.log(`getting ${key}`)            
                return value;         
            },         
            set : function(newValue){            
                console.log(`setting ${key}`)            
                notify() // 通知相关的模板进行编译           
                 value = newValue;         
            },         
            enumerable : true,         
            configurable : true        
        })    
    })
}
```
实际场景下我们还需要考虑如果某个属性还是对象应该递归下去
``` 
const data = {    
    name: 'nanjing',    
    age: 19
}
defineReactiveData(data)data.name // getting name  'nanjing'
data.name = 'beijing';  // setting name
```
当get和set触发的时候，已经能够同时触发我们想要调用的函数拉，Vue双向绑定过程中，当改变this上的data的时候去更新模板的核心原理就是这个方法，通过它我们就能在data的某个属性被set的时候，去触发对应模板的更新。
``` 
const data = {    
    userIds: ['01','02','03','04','05']
}
defineReactiveData(data);data.userIds // getting userIds ["01", "02", "03", "04", "05"]// 
get 过程是没有问题的，现在我们尝试给数组中push一个数据
data.userIds.push('06') // getting userIds 
```
上面的代码没有触发setting，反而因为取了一次userIds所以触发了一次getting~，不仅如此，很多数组的方法都不会触发setting，比如：push,pop,shift,unshift,splice,sort,reverse这些方法都会改变数组，但是不会触发set，所以Vue为了解决这个问题，重新包装了这些函数，同时当这些方法被调用的时候，手动去触发notify()； 
> Vue 不能检测以下变动的数组
> - 当你利用索引直接设置一个项时，例如：vm.items[indexOfItem] = newValue
> - 当你修改数组的长度时，例如：vm.items.length = newLength这个最根本的原因是因为这2种情况下，受制于js本身无法实现监听，所以官方建议用他们自己提供的内置api来实现，我们也可以理解到这里既不是defineProperty可以处理的，也不是包一层函数就能解决的，这就是2.x版本现在的一个问。

### 3.X版本
通过proxy实现对data对象的get和set的劫持，并返回一个代理的对象
``` 
const defineReactiveProxyData = data => new Proxy(data,     
    {        
        get: function(data, key){            
            console.log(`getting ${key}`)            
            return Reflect.get(data, key);        
        },        
        set: function(data, key, newVal){            
            console.log(`setting ${key}`)；            
            if(typeof newVal === 'object'){ 
                // 如果是object，递归设置代理                
                return Reflect.set(data, key, defineReactiveProxyData(newVal));       
            }            
            return Reflect.set(data, key, newVal);        
        }    
    })
    const data = {    
        name: 'nanjing',    
        age: 19
    };
    const vm = defineReactiveProxyData(data);vm.name // getting name  
    nanjingvm.age = 20; // setting age  20
```
2.x数组push,pop,shift,unshift,splice,sort,reverse不会触发更新，需要重新包装Array.prototype上的一些方法，使用了proxy后不需要了，而且直接改变数组的length还是通过某一个下标改变数组的内容，proxy都能拦截到这次变化，这比defineProperty方便太多了，
## 扩展
1. Proxy.revocable()，返回可取消的代理对象，一旦取消就不能再从代理对象访问了
``` 
const obj = {};
const handler = {};
const {proxy, revoke} = Proxy.revocable(obj, handler);
proxy.a = 1proxy.a // 
1revoke();
proxy.a // Uncaught TypeError: Cannot perform 'get' on a proxy that has been revoked
```
2. new Proxy出来的是一个新的对象，如果在target中有使用this，被代理后的this将指向新的代理对象，而不是原来的对象，这个时候，如果有些函数是原对象独有的，就会出现this指向导致的问题，这种场景下，建议使用bind来强制绑定this
``` 
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);
proxy.getDate(); // Uncaught TypeError: this is not a Date object.
```
因为代理后的对象并不是一个Date类型的，不具有getDate方法的，所以我们需要在get的时候，绑定一下this的指向
``` 
const target = new Date();
const handler = {    
    get: function(target, key){        
        if(typeof target[key] === 'function'){            
            return target[key].bind(target) // 强制绑定            
            this到原对象        
        }        
        return Reflect.get(target, key)    
    }
};
const proxy = new Proxy(target, handler);
proxy.getDate(); // 6
```

参考:
[你了解vue3.0响应式数据怎么实现吗?](https://juejin.cn/post/6844903861031796743)
