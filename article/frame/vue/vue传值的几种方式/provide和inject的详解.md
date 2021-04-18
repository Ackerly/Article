这种方式只需要祖先元素将数据放在provide，后代元素通过Inject去监听祖先元素传递的值    
祖先元素  
html部分
```
<template>
    <A :title="msg"></A>
</template>
```
js部分
```
    import A from "./A.vue";
    export default {
        name: "Ancestor",
        components: {
            A,
        },
        data() {
            return {
                msg: "this is ancestor data"
            }
        },

        provide() {
            return {
                msg: this.msg
            }
        }
    }
```
子孙元素  
html
```
<template>
    <div>B{{msg}}</div>
</template>
```
js
```
export default {
    name: "B",
    inject: ["msg"]
}
```
