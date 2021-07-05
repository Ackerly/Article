<!-- TOC -->

- [Vue3的20个响应式API](#vue3的20个响应式api)
  - [概述](#概述)
  - [Vue内置的20个响应式API](#vue内置的20个响应式api)
    - [ref](#ref)
      - [创建响应式值](#创建响应式值)
      - [Ref解包](#ref解包)
    - [reactive](#reactive)
    - [shallowReactive](#shallowreactive)
    - [readonly](#readonly)
    - [shallowReadonly](#shallowreadonly)
    - [unref](#unref)
    - [shallowRef](#shallowref)
    - [triggerRef](#triggerref)
    - [customRef](#customref)
    - [toRef](#toref)
    - [toRefs](#torefs)
    - [computed](#computed)
    - [watch](#watch)
    - [watchEffect](#watcheffect)
    - [isReactive](#isreactive)
    - [isProxy](#isproxy)
    - [toRaw](#toraw)
    - [markRaw](#markraw)
    - [isRef](#isref)

<!-- /TOC -->
# Vue3的20个响应式API
## 概述
响应式顺序：
effect > track > trigger > effect  
组件渲染过程中，假设正在走一个“effect”（副作用），这个effect会在过程中把它接触到的值（）也就是说会触发到值的get方法）。从而对值进行track（追踪）当值发生改变，就会进行track（追踪）。当值发生改变就会进行trigger（触发），执行effect来完成一个响应！  
在Vue中有三种effect，视图渲染effect、计算属性effect、侦听器effect
``` 
setup() {
  const count = ref(1);
  const computedCount = computed(() => {
    return count.value + 1;
  });
  watch(count, (val, oldVal) => {
    console.log('val :>> ', val);
  });
  const handleAdd = () => {
    count.value++;
  };
  return {
    count,
    computedCount,
    handleAdd
  };
}
```
## Vue内置的20个响应式API
### ref
#### 创建响应式值  
创建独立的响应式值作为refs。ref会返回一个可变的响应对象，该对象作为一个响应式的引用维护着它内部的值，该对象只包含一个名为value的property  
``` 
mport { ref } from 'vue'

const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```
#### Ref解包
ref作为渲染上下文（从setup中返回的对象）上的property返回并可以在模板中访问时，自动浅层次解包内部值。只有访问嵌套的ref时需要在模板中添加.value。
``` 
<template>
  <div>
    <span>{{ count }}</span>
    <button @click="count ++">Increment count</button>
    <button @click="nested.count.value ++">Nested Increment count</button>
  </div>
</template>

<script>
  import { ref } from 'vue'
  export default {
    setup() {
      const count = ref(0)
      return {
        count,

        nested: {
          count
        }
      }
    }
  }
</script>
```

### reactive    
返回对象的响应式副本，响应式是“深层”的，它影响所有嵌套property。返回的proxy不等于原始对象。  
``` 
const count = ref(1)
const obj = reactive({ count })

// ref 会被解包
console.log(obj.count === count.value) // true

// 它会更新 `obj.count`
count.value++
console.log(count.value) // 2
console.log(obj.count) // 2

// 它也会更新 `count` ref
obj.count++
console.log(obj.count) // 3
console.log(count.value) // 3
```
### shallowReactive  
``` 
const shallowReactiveObj = shallowReactive({
  id: 1,
  name: 'front-refiend',
  childObj: { hobby: 'coding' }
});
// 改变 id 是响应式的
shallowReactiveObj.id = 2;
// 改变嵌套对象的属性是非响应式的，但是本身的值是有被改变的
shallowReactiveObj.childObj.hobby = 'play';
```
### readonly  
获取一个对象（响应式或纯对象）或ref并返回原始proxy的制度proxy。只读proxy是深层的：访问的任何嵌套property也是只读的。
``` 
const proxyObj = reactive({
  childObj: {
    hobby: 'coding'
  }
});
const readonlyObj = readonly(proxyObj);

// 如果被拷贝对象 proxyObj 做了修改，打印 readonlyObj.childObj.hobby 也会看到有变更
proxyObj.childObj.hobby = 'play';

console.log('readonlyObj.childObj.hobby :>> ', readonlyObj.childObj.hobby);
// readonlyObj.childObj.hobby :>>  play

// 只读对象被改变，警告
readonlyObj.childObj.hobby = 'play';
// Set operation on key "hobby" failed: target is readonly.
```
### shallowReadonly  
``` 
const shallowReadonlyObj = shallowReadonly({
  id: 1,
  name: 'front-refiend',
  childObj: { hobby: 'coding' }
});

shallowReadonlyObj.id = 2;
// Set operation on key "id" failed: target is readonly. 
// 对象本身的属性不能被修改

shallowReadonlyObj.childObj.hobby = 'runnnig';
// 嵌套对象的属性可以被修改，但是是非响应式的！
```
### unref  
展开一个ref，判断参数为ref，则返回.value，否则返回参数本身。
``` 
// 模拟：在 setup 内定义一个 ref
const num = ref(1);
// 模拟：在 setup 返回，提供 template 使用
function setup() {
  return { num };
}
// 模拟：接收了 setup 返回的对象
const setupReturnObj = setup();
// 定义处理器对象，get 访问器里的 unref 是关键
const shallowUnwrapHandlers = {
  get: (target, key, receiver) =>
    unref(Reflect.get(target, key, receiver)),
  set: (target, key, value, receiver) => {
    const oldValue = target[key];
    if (isRef(oldValue) && !isRef(value)) {
      oldValue.value = value;
      return true;
    } else {
      return Reflect.set(target, key, value, receiver);
    }
  }
};
// 模拟：返回组件实例上下文
const ctx = new Proxy(setupReturnObj, shallowUnwrapHandlers);
// 模拟：template 最终被编译成 render 函数
/* 
  <template>
    <input v-model="num" />
    <div>num：{{num}}</div>
  </template>
  */
function render(ctx) {
  with (ctx) {
    // 模拟：在template中，进行赋值动作 "onUpdate:modelValue": $event => (num = $event)
    // num = 666;
    // 模拟：在template中，进行读取动作 {{num}}
    console.log('num :>> ', num);
  }
}
render(ctx);

// 模拟：在 setup 内部进行赋值动作
num.value += 1;
// 模拟： num 改变 trigger 视图渲染effect，更新视图
render(ctx);
```
### shallowRef  
如果传入的值为true，直接返回传入的原始值，不会申城代理对象
``` 
const shallowRefObj = shallowRef({
  id: 1,
  name: 'front-refiend',
});

上面的对象类似于：
const shallowRefObj = {
  value: {
    id: 1,
    name: 'front-refiend'
  }
};
```
### triggerRef
``` 
const shallowRefObj = shallowRef({
  name: 'front-refined'
});
// 这里不会触发副作用，因为是这个 ref 是浅层的
shallowRefObj.value.name = 'hello~';

// 手动执行与 shallowRef 关联的任何副作用，这样子就能触发了。
triggerRef(shallowRefObj);
```
### customRef
自定义ref。
``` 
<template>
  <div>name：{{name}}</div>
  <input v-model="name" />
</template>

// ...
setup() {
  let value = 'front-refined';
  // 参数是一个工厂函数
  const name = customRef((track, trigger) => {
    return {
      get() {
        // 收集依赖它的 effect
        track();
        return value;
      },
      set(newValue) {
        value = newValue;
        // 触发更新依赖它的所有 effect
        trigger();
      }
    };
  });
  return {
    name
  };
}
```
### toRef
为响应式对象的property创建一个ref，从而保持对其源property的响应式连接。
``` 
// 1. 传递整个响应式对象
function useHello(state) {
  state.name = 'hello~';
}
// 2. 传递一个具体的 ref
function useHello2(name) {
  name.value = 'hello~';
}

export default {
  setup() {
    const state = reactive({
      id: 1,
      name: 'front-refiend'
    });
    // 1. 直接传递整个响应式对象
    useHello(state);
    // 2. 传递一个新创建的 ref
    useHello2(toRef(state, 'name'));
  }
};
```
### toRefs
将响应式对象转换为普通对象，其中结果对象的每个property都是指向原始队形相应property的ref.
解构props:
``` 
export default {
  props: {
    id: Number,
    name: String
  },
  setup(props, ctx) {
    const { id, name } = toRefs(props);
    watch(id, () => {
      console.log('id change');
    });
    
    // 没有使用 toRefs 的话，需要通过这种方式监听
    watch(
      () => props.id,
      () => {
        console.log('id change');
      }
    );
  }
};
```
setup return是转换
``` 
<template>
  <div>id：{{id}}</div>
  <div>name：{{name}}</div>
</template>
// ...
setup() {
  const state = reactive({
    id: 1,
    name: 'front-refiend'
  });

  return {
    ...toRefs(state)
  };
}
```
### computed
它依赖响应式基础数据，当数据变化时候会触发它的更新。computed 主要的优点就是缓存了，可以缓存性能开销比较大的计算。它返回一个 ref 对象。

### watch
### watchEffect
###检查队形是否由readonly创建的制度proxy
``` 
function isReadonly(value) {
    return !!(value && value["__v_isReadonly" /* IS_READONLY */]);
}

// readonly
const originalObj = reactive({ id: 1 });
const copyObj = readonly(originalObj);
isReadonly(copyObj); // true

// 只读 computed 
const firsttName = ref('san');
const lastName = ref('zhang');
const fullName = computed(
  () => firsttName.value + ' ' + lastName.value
);
isReadonly(fullName); // true
```
### isReactive
检查对象是否是 reactive 创建的响应式 proxy。
### isProxy
检查对象是否由reactive或readonly创建的proxy
### toRaw
打印原始对象，对于转换ref对象，仍然保留包装过的对象
``` 
const obj = reactive({ id: 1, name: 'front-refiend' });
console.log(toRaw(obj));
// {id: 1, name: "front-refiend"}
const count = ref(0);
console.log(toRaw(count));
// {__v_isRef: true, _rawValue: 0, _shallow: false, _value: 0, value: 0}
```
### markRaw
标记一个对象，使其永远不会转为proxy，返回对象本身。
### isRef
判断是否是ref对象

参考：  
[Vue3丨进一步了解这 20 个响应式 API，写码如有神](https://segmentfault.com/a/1190000039194351?content_source_url=https://github.com/vue3/vue3-News)
