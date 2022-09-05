# 手写Vue.extend方法
# 概述
用法：使用基础Vue构造器创建一个“子类”，其参数是一个包含“组件选项”的对象。data选项是特例，在Vue.extend()中，它必须是函数。
``` 
<div id="test"></div>
let Custom = Vue.extend({
    tempplate:"<h1>{{title}}</h1>"
    data() {
        return {
            title: "vue extend"
        }
    }
})
new Custom().$mount('test')
```
全局API和实例不一样，是在Vue原型上挂在方法，也就是Vue.prototype上挂载方法
``` 
Vue.extend = function(extendOptions) {...}
```
作用：Vue.extend的作用是创建一个子类，然后继承Vue身上的一些功能

## 实现
extend实现源码,initExtend会在new Vue时调用 
``` 
  function initExtend (Vue) {
    Vue.cid = 0;
    var cid = 1;

    Vue.extend = function (extendOptions) {
      extendOptions = extendOptions || {};
      var Super = this;
      var SuperId = Super.cid;
      var cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {});
      if (cachedCtors[SuperId]) {
        return cachedCtors[SuperId]
      }

      var name = extendOptions.name || Super.options.name;
      if (name) {
        validateComponentName(name);
      }

      var Sub = function VueComponent (options) {
        this._init(options);
      };
      Sub.prototype = Object.create(Super.prototype);
      Sub.prototype.constructor = Sub;
      Sub.cid = cid++;
      Sub.options = mergeOptions(
        Super.options,
        extendOptions
      );
      Sub['super'] = Super;

      // For props and computed properties, we define the proxy getters on
      // the Vue instances at extension time, on the extended prototype. This
      // avoids Object.defineProperty calls for each instance created.
      if (Sub.options.props) {
        initProps$1(Sub);
      }
      if (Sub.options.computed) {
        initComputed$1(Sub);
      }

      // allow further extension/mixin/plugin usage
      Sub.extend = Super.extend;
      Sub.mixin = Super.mixin;
      Sub.use = Super.use;

      // create asset registers, so extended classes
      // can have their private assets too.
      ASSET_TYPES.forEach(function (type) {
        Sub[type] = Super[type];
      });
      // enable recursive self-lookup
      if (name) {
        Sub.options.components[name] = Sub;
      }

      // keep a reference to the super options at extension time.
      // later at instantiation we can check if Super's options have
      // been updated.
      Sub.superOptions = Super.options;
      Sub.extendOptions = extendOptions;
      Sub.sealedOptions = extend({}, Sub.options);

      // cache constructor
      cachedCtors[SuperId] = Sub;
      return Sub
    };
  }
```
Vue.extend方法增加了缓存策略。相关代码：
``` 
function initExtend(Vue) {
    Vue.cid = 0;
    var cid = 1;
    Vue.extend = function(extendOptions) {
          extendOptions = extendOptions || {};
          var Super = this;
          var SuperId = Super.cid;
          var cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {});
         if(cachedCtors[SuperId]){
          return cachedCtors[SuperId];
         }
         ...
         cachedCtors[SuperId] = Sub;
         return Sub
    }
}
```
每个构造构造实例都有唯一的cid，这样子构造函数能用于原型并缓存它们。作为参数的extendOptions第一次添加属性_Ctor={}，下次再用同一extendOptions来创建子类的时候，这个extendOptions身上就会有_Ctor这个属性了，然后再赋值。
对name进行校验，如果name不合格会在开发环境下发出警告,然后创建子类并将它返回，但是这一步没有继承的逻辑，此时子类不能用，还不具备Vue的能力
``` 
var name = extendOptions.name || Super.options.name;
if (name) {
    validateComponentName(name);
}
var Sub = function VueComponent (options) {
    this._init(options);
};
```
子类继承Vue的能力，并为子类添加cid
``` 
Sub.prototype = Object.create(Super.prototype);
Sub.prototype.constructor = Sub;
Sub.cid = cid++;
```
合并父类选项与子类选项的逻辑，并将父类保存到子类的super属性中，mergeOption会将两个选项合并为一个新对象，如果选项中存在props，初始化它
``` 
Sub.options = mergeOptions(
 Super.options,
 extendOptions
)
Sub['super'] = Super;
if(Sub.options.props){
 initProps(Sub);
}
```
初始化props的作用是将key代理到_props中，例如vm.name实际可以访问到的是Sub.prototype._props.name
``` 
function initProps$1 (Comp) {
    var props = Comp.options.props;
    for (var key in props) {
      proxy(Comp.prototype, "_props", key);
    }
}
function proxy (target, sourceKey, key) {
    sharedPropertyDefinition.get = function proxyGetter () {
      return this[sourceKey][key]
    };
    sharedPropertyDefinition.set = function proxySetter (val) {
      this[sourceKey][key] = val;
    };
    Object.defineProperty(target, key, sharedPropertyDefinition);
}
```
若果存在computed则初始化
``` 
if (Sub.options.computed) {
    initComputed$1(Sub);
}
function initComputed$1 (Comp) {
    var computed = Comp.options.computed;
    for (var key in computed) {
      defineComputed(Comp.prototype, key, computed[key]);
    }
}
```
将父类存在的属性依次复制到子类中,复制的方法包括extend、mixin、use、component、directive和filter
```
  Sub.extend = Super.extend;
  Sub.mixin = Super.mixin;
  Sub.use = Super.use;

  ASSET_TYPES.forEach(function (type) {
    Sub[type] = Super[type];
  });
  if (name) {
    Sub.options.components[name] = Sub;
  }
  Sub.superOptions = Super.options;
  Sub.extendOptions = extendOptions;
  Sub.sealedOptions = extend({}, Sub.options);
```

原文: 
[学习vue源码（2） 手写Vue.extend方法](https://juejin.cn/post/6844904181401141262)
