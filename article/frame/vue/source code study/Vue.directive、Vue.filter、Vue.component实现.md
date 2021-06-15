# Vue.directive、Vue.filter、Vue.component实现
## Vue.directive
用法: 注册或获取全局指令
```
// 注册 
Vue.directive('my-directive',{
 bind:function(){},
 inserted:function(){},
 update:function(){},
 componentUpdated:function(){},
 unbind:function(){},
})
// 注册（指令函数）
Vue.directive('my-directive',function(){
 // 将被bind和update调用
 ...
})
// getter方法,返回已注册的指令
var myDirective = Vue.directive('my-directive')
```
Vue.directive方法的作用是注册或获取全局指令，而不是让指令生效。其区别是注册指令需要做的事是将指令保存在某个位置，而指令生效是将指令从某个位置拿出来执行它。  
实现：
``` 
ASSET_TYPES.forEach(function (type) {
  Vue[type] = function (
    id,
    definition
  ) {
    if (!definition) {
      return this.options[type + 's'][id]
    } else {
      /* istanbul ignore if */
      if (type === 'component') {
        validateComponentName(id);
      }
      if (type === 'component' && isPlainObject(definition)) {
        definition.name = definition.name || id;
        definition = this.options._base.extend(definition);
      }
      if (type === 'directive' && typeof definition === 'function') {
        definition = { bind: definition, update: definition };
      }
      this.options[type + 's'][id] = definition;
      return definition
    }
  };
});
```
