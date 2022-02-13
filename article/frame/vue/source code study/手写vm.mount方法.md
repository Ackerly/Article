# 手写vm.$mount方法
Vue.extend创建出Vue的子类构造函数后，通过new 得到子类的实例，然后通过$mount挂载到节点，如代码：  
```
<div id="mount-point"></div>
<!-- 创建构造器 -->
var Profile = Vue.extend({
 template:'<p>{{firstName}} {{lastName}} aka{{alias}}</p>',
 data:function(){
  return{
   firstName:'Walter',
   lastName:'White',
   alias:'Heisenberg'
  }
 }
})
<!-- 创建Profile实例，并挂载到一个元素上 -->
new Profile().$mount('#mount-point');
```
## 概述
**使用方式**  
``` 
vm.$mount( [elementOrSelector] )
```
1. 参数  
```  
{ Element | string } [elementOrSelector]
```
2. 返回值
vm，即实例本身。
3. 用法
    - 如果Vue.js实例在实例化时没有收到el选项，则它处于“未挂载”状态，没有关联的DOM元素。
    - 以使用vm.$mount手动挂载一个未挂载的实例
    - 如果没有提供elementOrSelector参数，模板将被渲染为文档之外的元素，并且必须使用原生DOM的API把它插入文档中
    - 这个方法返回实例自身，因而可以链式调用其他实例方法。
4. 这个方法返回实例自身，因而可以链式调用其他实例方法。
``` 
var MyComponent = Vue.extend({
 template:'<div>Hello!</div>',
})
<!-- 创建并挂载到#app（会替换#app） -->
new MyComponent().$mount('#app');
<!-- 创建并挂载到#app（会替换#app） -->
new MyComponent().$mount({el:'#app'});
<!-- 创建并挂载到#app（会替换#app） -->
var component = new MyComponent().$mount();
document.getElementById('app').appendChild(component.$el);
```
- 在不同的构建版本中，vm.$mount的表现都不一样。其差异主要体现在完整版（vue.js）和只包含运行时版本（vue.runtime.js）之间
- 完整版和只包含运行时版本之间的差异在于是否有编译器，而是否有编译器的差异主要在于vm.$mount方法的表现形式
- 在只包含运行时的构建版本中，vm.mount的作用如前面所诉。在完整的构建版本中，vm.mount的作用会稍有不同，它首先会检查template或el选项所提供的模板是否已经转换成渲染函数（render函数）。如果没有，则立即进入编译过程，将模板编译成渲染函数，完成之后再进入挂载与渲染的流程中。
- 只包含运行时版本的vm.$mount没有编译步骤，它会默认实例上已经存在渲染函数，如果不存在，则会设置一个。并且，这个渲染函数在执行时会返回一个空节点的VNode，以保证执行时不会因为函数不存在而报错。同时如果是开发环境下运行，Vue.js会触发警告，提示我们当前使用的是只包含运行时的版本，会让我们提供渲染函数，或者去使用完整的构建版本
- 从原理的角度来讲，完整版和只包含运行时版本之间是包含关系，完整版包含只包含运行时版本

## 完整版vm.$mount的实现原理
1. 实现代码
``` 
const mount = Vue.prototype.$mount;
Vue.prototype.$mount = function(el){
 <!-- 做些什么 -->
 return mount.call(this,el);
}
```
- 将Vue原型上的$mount方法保存在mount中，以便后续使用
- 然后Vue原型上的$mount方法被一个新的方法覆盖了。新方法中会调用原始的方法，这种做法通常被称为函数劫持
- 通过函数劫持，可以在原始功能上新增一些其他功能。

2. 由于el参数支持元素类型或者字符串类型的选择器，所以第一步是通过el获取DOM元素
``` 
const mount = Vue.prototype.$mount;
Vue.prototype.$mount = function(el){
    el = el && query(el);
    return mount.call(this,el);
}
```
使用query获取DOM元素
``` 
function query(el){
    if(typeof el === 'string'){
        const selected = document.querySelector(el);
        if(!selected){
            return document.createElement('div');
        }
        return selected;
    }else{
        return el;
    }
}
```
- 如果el是字符串，则使用doucment.querySelector获取DOM元素，如果获取不到，则创建一个空的div元素
- 如果el不是字符串，那么认为它是元素类型，直接返回el（如果执行vm.$mount方法时没有传递el参数，则返回undefined）

  1. 编译器
     1. 首先判断Vue.js实例中是否存在渲染函数，只有不存在时，才会将模板编译成渲染函数。
     ``` 
     const mount = Vue.prototype.$mount;
     Vue.prototype.$mount = function(el){
      el = el && query(el);
      const options = this.$options;
      if(!options.render){
       <!-- 将模板编译成渲染函数并赋值给options.render -->
      }
         return mount.call(this,el);
     }
     ```
     2. 在实例化Vue.js时，会有一个初始化流程，其中会向Vue.js实例上新增一些方法，这里的this.$options就是其中之一，它可以访问到实例化Vue.js时用户设置的一些参数，例如tempalte和render
     3. 如果在实例化Vue.js时给出了render选项，那么template其实是无效的，因为不会进入模板编译的流程，而是直接使用render选项中提供的渲染函数
     4. Vue.js在官方文档的template选项中也给出了相应的提示。如果没有render选项，那么需要获取模板并将模板编译成渲染函数（render函数）赋值给render选项
     ```  
      const mount = Vue.prototype.$mount;
      Vue.prototype.$mount = function(el){
      el = el && query(el);
      const options = this.$options;
      if(!options.render){
       <!-- 新增获取模板相关逻辑 -->
       let template = options.template;
       if(template){
    
       }else if(el){
        template = getOuterHTML(el);
       }
      }
         return mount.call(this,el);
     }
     ```
     5. 从选项中取出template选项，也就是取出用户实例化Vue.js时设置的模板。如果没有取到，说明用户没有设置tempalte选项。那么使用getOuterHTML方法从用户提供的el选项中获取模板。
     ```  
      function getOuterHTML(el){
      if(el.outerHTML){
       return el.outerHTML;
      }else{
       const container = document.createElement('div');
       container.appendChild(el.cloneNode(true));
       return container.innerHTML;
      }
     }
     ```
     6. getOuterHTML方法会返回参数中提供的DOM元素的HTML字符串
     7. 整体逻辑  
       如果用户没有通过template选项设置模板，那么会从el选项中获取HTML字符串当作模板。如果用户提供了template选项，那么需要对它进一步解析，因为这个选项支持很多种使用方式。template选项可以直接设置成字符串模板，也可以设置为以#开头的选择符，还可以设置成DOM元素
     8. 从不同的格式中将模板解析出来
     ``` 
     const mount = Vue.prototype.$mount;
     Vue.prototype.$mount = function(el){
      el = el && query(el);
      const options = this.$options;
      if(!options.render){
       <!-- 新增获取模板相关逻辑 -->
       let template = options.template;
       if(template){
        if(typeof tempalte === 'string'){
         if(tempalte.charAt(0) === "#"){
          template = idToTemplate(tempalte);
         }
        }else if(tempalte.nodeType){
         template = template.innerHTML;
        }else{
         if(process.env.NODE_ENV !== 'production'){
          warn('invalid template option:'+tempalte,this);
         }
         return this;
        }
       }else if(el){
        template = getOuterHTML(el);
       }
      }
         return mount.call(this,el);
     }
     ```
     9. 如果tempalte是字符串并且以#开头，则它将被用作选择符。通过选择符获取DOM元素后，会使用innerHTML作为模板
     10. 使用idToTemplate方法从选择符中获取模板。idToTemplate使用选择符获取DOM元素之后，将它的innerHTML作为模板。
     ``` 
     function idToTemplate(id){
      const el = query(id);
      return el && el.innerHTML;
     }
     ```
     11. 如果template是字符串，但不是以#开头，就说明template是用户设置的模板，不需要进行任何处理，直接使用即可
     12. 如果template选项的类型不是字符串，则判断它是否是一个DOM元素，如果是，则使用DOM元素的innerHTML作为模板。如果不是，只需要判断它是否具备nodeType属性即可。
     13. 如果tempalte选项既不是字符串，也不是DOM元素，那么Vue.js会触发警告，提示用户template选项是无效的。
     14. 获取模板之后，下一步是将模板编译成渲染函数，通过执行compileToFunctions函数可以将模板编译成渲染函数并设置到this.options上。
     ``` 
   
     const mount = Vue.prototype.$mount;
     Vue.prototype.$mount = function(el){
      el = el && query(el);
      const options = this.$options;
      if(!options.render){
       <!-- 新增获取模板相关逻辑 -->
       let template = options.template;
       if(template){
        if(typeof tempalte === 'string'){
         if(tempalte.charAt(0) === "#"){
          template = idToTemplate(tempalte);
         }
        }else if(tempalte.nodeType){
         template = template.innerHTML;
        }else{
         if(process.env.NODE_ENV !== 'production'){
          warn('invalid template option:'+tempalte,this);
         }
         return this;
        }
       }else if(el){
        template = getOuterHTML(el);
       }
       <!-- 新增编译相关逻辑 -->
       if(tempalte){
        const { render } = compileToFunctions(
         template,
         {...},
         this
        )
        options.render = render;
       }
      }
         return mount.call(this,el);
     }
     ```
     15. 将模板编译成代码字符串并将代码字符串转换成渲染函数的过程是在compileToFunctions函数中完成的，其内部实现如下
     ``` 
     function compileToFunctions(template,options,vm){
      options = extend({},options);
      <!-- 检查缓存 -->
      const key = options.delimiters
      ? String(options.delimiters)+tempalte
      :template;
      if(cache[key]){
       return cache[key];
      }
      <!-- 编译 -->
      const compiled = compile(template,options);
      <!-- 将代码字符串转换为函数 -->
      const res = {};
      res.render = createFunction(compiled.render);
      return (cache[key] = res)
     }
     function createFunction(code){
      return new Function(code);
     }
     ```
     - 首先，将options属性混合到空对象中，其目的是让options称为可选参数。
     - 检查缓存中是否已经存在编译后的模板。如果模板已经被编译，就会直接返回缓存中的结果，不会重复编译，保证不做无用功来提升性能
     - 调用compile函数来编译模板，将模板编译成代码字符串并存储在compiled中的render属性中。
     - 调用createFunction函数将代码字符串转换成函数。
     - 代码字符串被new Function(code)转换成函数之后，当调用函数时，代码字符串会被执行。例如
     ``` 
     const code = 'console.log("Hello Berwin")';
     const render = new Function(code);
     render();//Hello Berwin
     ```
     - 将渲染函数返回给调用方
     16. 当通过compileToFunctions函数得到渲染函数之后，将渲染函数设置到this.$options上
## 只包含运行时版本的vm.$mount的实现原理
1. 只包含运行时版本的vm.mount方法的核心功能。实现如下  
   ``` 
   Vue.prototype.$mount = function(el){
    el = el && inBrower ? query(el) : undefined;
    return mountComponent(this,el);
   }
   ```
   1. $mount方法将ID转换为DOM元素后，使用mountComponent函数将Vue.js实例挂载到DOM元素上
   2. 将实例挂载到DOM元素上指的是将模板渲染到指定的DOM元素中，而且是持续性的，以后当数据（状态）发生变化时，依然可以渲染到指定的DOM元素中。
   3. 实现这个功能需要开启watcher  
   watcher将持续观察模板中用到的所有数据（状态），当这些数据（状态）被修改时它将得到通知，从而进行渲染操作。这个过程回持续到实例被销毁。
   ``` 
   export function mountComponent(vm,el){
    if(!vm.$options.render){
     vm.$options.render = createEmptyVNode;
     if(process.env.NODE_ENV !== 'production'){
      <!-- 在开发环境发出警告 -->
     }
    }
   }
   ```
   4. mountComponent方法会判断实例上是否存在渲染函数。如果不存在，则设置一个默认的渲染函数createEmptyVNode，该渲染函数执行后，会返回一个注释类型的VNode节点
   5. 如果在mountComponent方法中发现实例上没有渲染函数，则会将el参数指定页面中的元素节点替换成一个注释节点，并且在开发环境下在浏览器的控制台中给出警告。

2. Vue.js实例在不同的阶段会触发不同的生命周期钩子，在挂载实例之前会触发beforeMount钩子函数
``` 
export function mountComponent(vm,el){
 if(!vm.$options.render){
  vm.$options.render = createEmptyVNode;
  if(process.env.NODE_ENV !== 'production'){
   <!-- 在开发环境发出警告 -->
  }
  callHook(vm,'beforeMount')
 }
}

```
钩子函数触发后，将执行真正的挂载操作。挂载操作与渲染类似，不同的是渲染指的是渲染一次，而挂载指的是持续性渲染。挂载之后，每当状态发生变化时，都会进行渲染操作
3. mountComponent具体实现
   ```
   export function mountComponent(vm,el){
    if(!vm.$options.render){
     vm.$options.render = createEmptyVNode;
     if(process.env.NODE_ENV !== 'production'){
      <!-- 在开发环境发出警告 -->
     }
     <!-- 触发生命周期钩子 -->
     callHook(vm,'beforeMount');
     <!-- 挂载 -->
     vm._watcher = new Watcher(vm,()=>{
      vm._update(vm._render())
     },noop);
     <!-- 触发生命周期钩子 -->
     callHook(vm,'mounted');
     return vm;
    }
   }
   ```
   1. vm._update作用：调用虚拟DOM中的patch方法来执行节点的比对与渲染操作。
   2. vm._render作用：执行渲染函数，得到一份新的VNode节点树。
   3. vm._update(vm._render())作用：先调用渲染函数得到一份最新的VNode节点树，然后通过vm._update方法对最新的VNode和上一次渲染用到的旧VNode进行对比并更新DOM节点。简单来说，就是执行了渲染操作。
4. 挂载是持续性的，而持续性的关键就在于new Watcher这行代码。
``` 
export function mountComponent(vm,el){
 if(!vm.$options.render){
  vm.$options.render = createEmptyVNode;
  if(process.env.NODE_ENV !== 'production'){
   <!-- 在开发环境发出警告 -->
  }
  <!-- 触发生命周期钩子 -->
  callHook(vm,'beforeMount');
  <!-- 挂载 -->
  
   vm._update(vm._render())
  
  <!-- 触发生命周期钩子 -->
  callHook(vm,'mounted');
  return vm;
 }
}

```

**总结**  
$mount()的思路就是， 判断 用户传入的option有没有render函数，
1. 有的话就走运行时版本
2. 没有的话就自动生成render函数，然后在执行运行时版本

执行运行时版本的时候
- 通过render()获得Vnode
- 把Vnode传入_update() 实现渲染

参考：  
[学习vue源码（4） 手写vm.$mount方法](https://juejin.cn/post/6844904181438889991)