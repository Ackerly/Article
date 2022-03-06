# Pinia上手指南
Vuex作为Vue前端框架官宣的状态管理工具，一直在与时俱进。然而其繁琐的actions、mutations以及模块嵌套一直饱受开发者诟病，我们始终期待着使用一款简洁的状态管理器。而Pinia的出现，去掉了模块的多层嵌套，移除了复杂的mutation操作，极大地满足了我们的诉求。  
**安装**  
```
# with yarn
yarn add pinia --save
# or with npm
npm install pinia --save
```
**创建pinia并挂载到app上**  
```
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";
import { createPinia } from "pinia";

const app = createApp(App),
  pinia = createPinia();
app.use(pinia).use(router).mount("#app");
```
**定义store**  
Pinia的各个store之间是扁平化的，因此使用起来十分简洁，不像Vuex的module嵌套那么复杂。通过defineStore方法来声明一个store，传入的第一个变量为该store的id，应保证其唯一性；第二个参数为初始化配置，包含state, getters, actions等可选项, 各项都类似Vuex。值得注意的是，state为一个函数，返回一个对象，就如同vue的options API中的data选项。此外，Pinia认为mutations会增加操作复杂度，因此去除了mutations选项有多种方式可以修改state的值，甚至可以直接修改。
```
// src/store/userStore.ts

// 导入并使用defineStore来定义store
import { defineStore } from "pinia";

interface UserState {
  username: string;
  password: string;
  role: string;
}

export const useUserStore = defineStore("user", {
  state: (): UserState => {
    return {
      username:"Amy",
      // ...
    };
  },
  getters: {
    // ...
  },
  actions: {
    // ...
  },
});

```
## store的各个属性介绍
**store的各个属性介绍**  
state为一个函数，其返回值为一个对象，用于存储公共状态。对state或state的属性进行修改的行为叫做mutation。在Pinia中，不提供mutations选项。修改state的mutation行为有多种：
- 直接修改
  ```
    // xxx.vue
    import { ref, computed, defineComponent } from "vue";import { useUserStore } from "@/store/userStore";
    import { useUserStore } from "@/store/userStore";
    export default defineComponent({
    setup() {
        const userStore = useUserStore();
        const username = computed(()=>userStore.username)
        userStore.username = "张三";
        return {username}
    }
    })

  ```
- 在action里修改 (也属于直接修改)  
  先在actions声明相应的action方法，使用this可以获取store实例
  ```
    // src/store/userStore.ts
    // 导入并使用defineStore来定义store
    import { defineStore } from "pinia";

    interface UserState {
    username: string;
    password: string;
    role: string;
    }

    export const useUserStore = defineStore("user", {
    state: (): UserState => {
        return {
        username: "",
        password: "",
        role: "",
        };
    },
    actions: {
        // 使用this指代本store实例
        setUserInfo(username: string): void {
        this.username = username;
        },
        setRole(role: string): void {
        this.role = role;
        },
    },
    });
  ```
  然后在setup中调用相应的action就行了:  
  ```
    import { useUserStore } from "@/store/userStore";
    setup(){
    const userStore = useUserStore()
    userStore.setRole("至尊宝");
    }
  ```
- 使用$patch来修改部分state属性, 可以传入一个对象或者函数
  ```
    import { useUserStore } from "@/store/userStore";
    setup(){
    const userStore = useUserStore();
    // 使用$patch传入对象来修改state部分属性
    // 此时mutation.type为 'patch object'
    userStore.$patch({
        username: "李四",
        password: "李四爱吃蛋糕",
        role: "father"
    })
  ```
- 使用$state来重设整个state，此时传入的对象必须包含state的所有属性
  ```
    setup(){
    // 此时mutation.type为 'patch function'
        userStore.$state = {
            username: "狗蛋",
            password: "gogogo",
            role: "son"
        }
    }
  ```

**侦听state的变化**  
使用$subscribe来监测state的变化 (类似watch)，接收一个callback函数作为第一个参数，接收的callback主要有mutation和state两个参数，mutation包含修改state的方式、storeId等信息，可以根据相应信息对state进行处理。  
```
userStore.$subscribe((mutation, state) => {
  mutation包含修改state的信息
  // 修改state的事件信息
  console.log(mutation.events);
  // 创建store时传入的第一个参数
  console.log(mutation.storeId);
  // 'direct' | 'patch object' | 'patch function' 三种
  // 其中直接修改和在action里直接修改都是对应 'direct'
  // $patch一个对象是 'patch object'
  // $patch一个函数或者$state重设则是对应 'patch function'
  console.log(mutation.type);

  // state为修改完之后的state的Proxy
  if (mutation.storeId === "user") {
    console.log(state)
    // 实现自动持久化存储
    localStorage.setItem("userInfo", JSON.stringify(state))
  }
})
```
$subscribe仅在使用的vue组件里生效，一旦该组件被卸载了，将不在继续侦听state的变化。如果我们想要让页面卸载后,$subscribe依然生效，只需要给它传入第二个参数{ detached:true } ，使其脱离组件保持独立，从而在该组件卸载后能继续工作：
```
export default {
  setup() {
    const userStore = useUserStore()

    // this subscription will be kept after the component is unmounted
    userStore.$subscribe(callback, { detached: true })
    // ...
  }
}
```
**getters**  
getters同vue里的computed选项，依赖state的属性并返回一个新的值：  
```
import { defineStore } from "pinia";

export const useUserStore = defineStore("user", {
  state: (): UserState => {
    return {
      username: "",
      password: "",
      role: "",
    };
  },
  getters: {
    // 传入参数state, 可解构
    authorityLevel: ({ role }) => {
      return role === "elder" ? 0 : role === "father" ? 1 : role === "son" ? 2 : null;
    }
  }
});
```
getters实现接收参数，只需要让其返回一个接收参数的函数即可，此时的getter将不再具有缓存特性。
```
export const useUserStore = defineStore('user', {
  getters: {
    getUserById: (state) => {
      return (userId) => state.users.find((user) => user.id === userId)
    },
  },
})
```
在vue组件里就可以给相应的getter传参了：  
```
<script>
import {useUserStore} from "@/store/userStore.js";
export default {
  setup() {
    const store = useUserStore()

    return { getUserById: store.getUserById }
  },
}
</script>

<template>
  <p>User 8: {{ getUserById(8) }}</p>
</template>
```
在getter中可以获取别的getter的值，甚至是其它store里的getter或state的值
```
// cartStore.ts

import { defineStore } from "pinia";
import { useUserStore } from "./userStore";

interface Goods {
  name: string,
  price: number,
  count: number,
}

type GoodsList = Goods[]

interface CartState {
  items: GoodsList,
}

export const useCartStore = defineStore("cart", {
  state: (): CartState => {
    return {
      items: []
    }
  },
  getters: {
    owner: () => {
      // 获取其它store的state和getters
      const userStore = useUserStore();
      return `姓名：${userStore.username}, 权限等级：${userStore.authorityLevel}`
    }
  },
})
```
**actions**  
actions类似于vue里的methods选项，其中定义一些方法用于修改state，可以进行异步操作。由于没有mutations选项，因此可以直接在actions中修改state，大大简化了操作。  
```
// src/store/userStore.ts

// 导入并使用defineStore来定义store
import { defineStore } from "pinia";

interface UserState {
  username: string;
  password: string;
  role: string;
}

export const useUserStore = defineStore("user", {
  state: (): UserState => {
    return {
      username: "",
      password: "",
      role: "",
    };
  },
  actions: {
    // 使用this指代本store实例
    setUserInfo(username: string): void {
      this.username = username;
    },
    // 可以进行异步操作
    setPassword(password: string): void {
      const timer = setTimeout(() => {
        this.password = password;
        clearTimeout(timer);
      }, 1000);
    },
    setRole(role: string): void {
      this.role = role;
    },
  },
});
```
可以在actions中调用自己或其它store里的action
```
import { defineStore } from "pinia";
import { useUserStore } from "./userStore";
export const useCartStore = defineStore("cart", {
  state: (): CartState => {
    return {
      items: []
    }
  },
  getters: {
    owner: () => {
      // 获取其它store的state和getters
      const userStore = useUserStore();
      return `姓名：${userStore.username}, 权限等级：${userStore.authorityLevel}`
    }
  },
  actions: {
    // 使用其它store的action
    resetOwnerRole() {
      const userStore = useUserStore();
      userStore.setRole("elder");
    }
    // 使用自己store里的其它action，用this指代store实例
    callOtherAction(){
      this.resetOwnerRole()
    }
  },
})
```
## 在setup中使用store
类似Vuex中的useStore函数，Pinia也提供了相似的用法，在组件的script标签中导入我们自定义的Store函数，调用后赋值给相应的变量即可。state和getters都能直接访问，可以使用computed使被赋值的变量变为响应式。  
```
// xxx.vue

import { ref, computed, defineComponent } from "vue";
import { useUserStore } from "@/store/userStore";
export default defineComponent({
  setup() {
    const userStore = useUserStore(),
      // state
      username = computed(() => userStore.username),
      // 使用computed, 则password成为了响应式数据，而username不是。
      password = computed(() => userStore.password),
      // getters
      authority = computed(() => userSore.authorityLevel)
    return {
      username,
      password,
      authority
    }
  }
})
```
## 在setup外面使用store
需要注意useStore使用的时机，需要在app挂载pinia之后才能使用，以在路由守卫中为例：
```
// src/router/index.ts

// ! 无效，会报错还未安装pinia
// const userStore = useUserStore();

router.beforeEach((to, from, next) => {
  // 有效, 此时vue已经挂载了router，则也挂载了pinia
  const userStore = useUserStore();
  to.path === "/about" && userStore.role === "" && next("/login");
  next();
});
```

参考：  
[看完这篇Pinia上手指南，你还会选择Vuex吗？](https://juejin.cn/post/7068483852674531335?utm_source=gold_browser_extension)