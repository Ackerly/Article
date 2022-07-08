# 为什么 Vue2 this 能够直接获取到 data 和 methods
对于下面这一段代码，我们可能每天都会见到：  
``` 
const vm = new Vue({
  data: {
      name: '我是pino',
  },
  methods: {
      print(){
          console.log(this.name);
      }
  },
});
console.log(vm.name); // 我是pino
vm.print(); // 我是pino
```
但是自己实现一个构造函数却实现不了这种效果呢？  
``` 
function Super(options){}

const p = new Super({
    data: {
        name: 'pino'
    },
    methods: {
        print(){
            console.log(this.name);
        }
    }
});

console.log(p.name); // undefined
p.print(); // p.print is not a function
```
**源码实现**  
vue2的入口文件：src/core/instance/index
``` 
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

// 初始化操作是在这个函数完成的
initMixin(Vue)

stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```
initMixin文件中是如何实现的  
``` 
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++

    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
    vm._isVue = true
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    
    // 初始化data/methods...
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
  }
}
```
关注initState这个函数就好了，这个函数初始化了props, methods, watch, computed
- 使用initProps初始化了props
- 使用initMethods初始化了methods
- 使用initData初始化了data
- 使用initComputed初始化了computed
- 使用initWatch初始化了watch

``` 
function initState (vm) {
    vm._watchers = [];
    var opts = vm.$options;
    // 判断props属性是否存在，初始化props
    if (opts.props) { initProps(vm, opts.props); }
    // 有传入 methods，初始化方法methods
    if (opts.methods) { initMethods(vm, opts.methods); }
    // 有传入 data，初始化 data
    if (opts.data) {
      initData(vm);
    } else {
      observe(vm._data = {}, true /* asRootData */);
    }
    // 初始化computed
    if (opts.computed) { initComputed(vm, opts.computed); }
    // 初始化watch
    if (opts.watch && opts.watch !== nativeWatch) {
      initWatch(vm, opts.watch);
    }
}
```
这里只关注initMethods和initData  
initMethods  
``` 
function initMethods (vm, methods) {
    var props = vm.$options.props;
    for (var key in methods) {
      {
          // 判断是否为函数
        if (typeof methods[key] !== 'function') {
          warn(
            "Method \"" + key + "\" has type \"" + (typeof methods[key]) + "\" in the component definition. " +
            "Did you reference the function correctly?",
            vm
          );
        }
        
        // 判断props存在且props中是否有同名属性
        if (props && hasOwn(props, key)) {
          warn(
            ("Method \"" + key + "\" has already been defined as a prop."),
            vm
          );
        }
        // 判断实例中是否有同名属性，而且是方法名是保留的 _ $ （在JS中一般指内部变量标识）开头
        if ((key in vm) && isReserved(key)) {
          warn(
            "Method \"" + key + "\" conflicts with an existing Vue instance method. " +
            "Avoid defining component methods that start with _ or $."
          );
        }
      }
      // 将methods中的每一项的this指向绑定至实例
      // bind的作用就是用于绑定指向，作用同js原生的bind
      vm[key] = typeof methods[key] !== 'function' ? noop : bind(methods[key], vm);
    }
}
```
整个initMethods方法核心就是将this绑定到了实例身上，因为methods里面都是函数，所以只需要遍历将所有的函数在调用的时候将this指向实例就可以实现通过this直接调用的效果  
大部分代码都是用于一些边界条件的判断：  
- 如果不为函数 -> 报错
- props存在且props中是否有同名属性 -> 报错
- 实例中是否有同名属性，而且是方法名是保留的 -> 报错 

bind函数  
``` 
function polyfillBind (fn, ctx) {
    function boundFn (a) {
      var l = arguments.length;
      // 判断参数的个数来分别使用call/apply进行调用
      return l
        ? l > 1
          ? fn.apply(ctx, arguments)
          : fn.call(ctx, a)
        : fn.call(ctx)
    }

    boundFn._length = fn.length;
    return boundFn
}

function nativeBind (fn, ctx) {
  return fn.bind(ctx)
}
// 判断是否支持原生的bind方法
var bind = Function.prototype.bind
  ? nativeBind
  : polyfillBind;

```
bind函数中主要是做了兼容性的处理，如果不支持原生的bind函数，则根据参数个数的不同分别使用call/apply来进行this的绑定，而call/apply最大的区别就是传入参数的不同，一个分别传入参数，另一个接受一个数组。  
hasOwn 用于判断是否为对象本身所拥有的对象，上文通过此函数来判断是否在props中存在相同的属性  
``` 
// 只判断是否为本身拥有，不包含原型链查找
var hasOwnProperty = Object.prototype.hasOwnProperty; 
function hasOwn (obj, key) { 
    return hasOwnProperty.call(obj, key) 
}

hasOwn({}, 'toString') // false
hasOwn({ name: 'pino' }, 'name') // true
```
isReserved  
判断是否为内部私有命名（以$或_开头）  
``` 
function isReserved (str) {
  var c = (str + '').charCodeAt(0);
  return c === 0x24 || c === 0x5F
}
isReserved('_data'); // true
isReserved('data'); // false
``` 
**initData**  
``` 
function initData (vm) {
    var data = vm.$options.data;
    // 判断data是否为函数，如果是函数，在getData中执行函数
    data = vm._data = typeof data === 'function'
      ? getData(data, vm)
      : data || {};
    // 判断是否为对象
    if (!isPlainObject(data)) {
      data = {};
      warn(
        'data functions should return an object:\n' +
        'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
        vm
      );
    }
    // proxy data on instance
    // 取值 props/methods/data的值
    var keys = Object.keys(data);
    var props = vm.$options.props;
    var methods = vm.$options.methods;
    var i = keys.length;
    // 判断是否为props/methods存在的属性
    while (i--) {
      var key = keys[i];
      {
        if (methods && hasOwn(methods, key)) {
          warn(
            ("Method \"" + key + "\" has already been defined as a data property."),
            vm
          );
        }
      }
      if (props && hasOwn(props, key)) {
        warn(
          "The data property \"" + key + "\" is already declared as a prop. " +
          "Use prop default value instead.",
          vm
        );
      } else if (!isReserved(key)) {
        // 代理拦截
        proxy(vm, "_data", key);
      }
    }
    // observe data
    // 监听数据
    observe(data, true /* asRootData */);
}
```
getData  
如果data为函数时，调用此函数对data进行执行  
``` 
function getData (data, vm) {
    // #7573 disable dep collection when invoking data getters
    pushTarget();
    try {
      // 将this绑定至实例
      return data.call(vm, vm)
    } catch (e) {
      handleError(e, vm, "data()");
      return {}
    } finally {
      popTarget();
    }
}
```
proxy  
代理拦截，当使用this.xxx访问某个属性时，返回this.data.xxx  
``` 
// 一个纯净函数
function noop (a, b, c) {}

// 代理对象
var sharedPropertyDefinition = {
    enumerable: true,
    configurable: true,
    get: noop,
    set: noop
};

function proxy (target, sourceKey, key) {
    // get拦截
    sharedPropertyDefinition.get = function proxyGetter () {
      return this[sourceKey][key]
    };
    // set拦截
    sharedPropertyDefinition.set = function proxySetter (val) {
      this[sourceKey][key] = val;
    };
    // 使用Object.defineProperty对对象进行拦截
    Object.defineProperty(target, key, sharedPropertyDefinition);
}
```
对data的处理就是将data中的属性的key遍历绑定至实例vm上，然后使用Object.defineProperty进行拦截，将真实的数据操作都转发到this.data上。

**简略实现**  
``` 
function Person(options) {
      let vm = this
      vm.$options = options

      if(options.data) {
        initData(vm)
      } 
      if(options.methods) {
        initMethods(vm, options.methods)
      }
    }

    function initData(vm) {
      let data = vm._data = vm.$options.data

      let keys = Object.keys(data)

      let len = keys.length
      while(len--) {
        let key = keys[len]
        proxy(vm, "_data", key)
      }
    }

    var sharedPropertyDefinition = {
        enumerable: true,
        configurable: true,
        get: noop,
        set: noop
    };

    function proxy(target, sourceKeys, key) {

      sharedPropertyDefinition.get = function() {
        return this[sourceKeys][key]
      }

      sharedPropertyDefinition.set = function(val) {
        this[sourceKeys][key] = val
      }

      Object.defineProperty(target, key, sharedPropertyDefinition)

    }

    function noop(a, b, c) {}

    function initMethods(vm, methods) {
      for(let key in methods) {
        vm[key] = typeof methods[key] === 'function' ? methods[key].bind(vm) : noop
      }
    }

    let p1 = new Person({
      data: {
        name: 'pino',
        age: 18
      },
      methods: {
        sayName() {
          console.log('I am' + this.name)
        }
      }
    })

    console.log(p1.name) // pino
    p1.sayName() // 'I am pino'
```

参考:  
[为什么 Vue2 this 能够直接获取到 data 和 methods](https://mp.weixin.qq.com/s/Uq-LYSrmn1lOjRuAl4jc2A)
