# Vue3源码中位运算符
## Vue3中的位运算
vue3的源码中，实现一个ShapeFlags（对元素进行标记判断是普通元素、函数组件、插槽、keep alive组件等等）
``` 
export const enum ShapeFlags {
  ELEMENT = 1,
  FUNCTIONAL_COMPONENT = 1 << 1,
  STATEFUL_COMPONENT = 1 << 2,
  TEXT_CHILDREN = 1 << 3,
  ARRAY_CHILDREN = 1 << 4,
  SLOTS_CHILDREN = 1 << 5,
  TELEPORT = 1 << 6,
  SUSPENSE = 1 << 7,
  COMPONENT_SHOULD_KEEP_ALIVE = 1 << 8,
  COMPONENT_KEPT_ALIVE = 1 << 9,
  COMPONENT = ShapeFlags.STATEFUL_COMPONENT | ShapeFlags.FUNCTIONAL_COMPONENT
}


 if (shapeFlag & ShapeFlags.ELEMENT || shapeFlag & ShapeFlags.TELEPORT) {
  ...
}


if (hasDynamicKeys) {
      patchFlag |= PatchFlags.FULL_PROPS
    } else {
    if (hasClassBinding) {
      patchFlag |= PatchFlags.CLASS
    }
    if (hasStyleBinding) {
      patchFlag |= PatchFlags.STYLE
    }
    if (dynamicPropNames.length) {
      patchFlag |= PatchFlags.PROPS
    }
    if (hasHydrationEventBinding) {
      patchFlag |= PatchFlags.HYDRATE_EVENTS
    }
}
```
## 按位非（~）
``` 
const a = 5;     // 00101

console.log(~a); // 11010 != -6
```
~5的计算步骤
1. 将5转二进制 ＝ 00000101
2. 按位取反 ＝ 11111010
3. 发现符号位(即最高位)为1(表示负数)，将除符号位之外的其他数字取反 ＝ 10000101
4. 末位加1取其补码 ＝ 10000110
5. 转换回十进制 ＝ -6

~5的计算步骤
1. 将-5转二进制 ＝ 10000101
2. 按位取反得到反码 = 11111010
3. 末位加1取其补码 ＝ 11111011
4. 再次取反 = 00000100
5. 转换回十进制 ＝ 4
## 异或（^）
同一位置不相等，则为1
``` 
const a = 5;     // 00101
const b = 3;     // 00011

console.log(a ^ b); // 00110
//  6
```
## <<（左移）,>>（右移）, >>>（无符号右移）
### <<（左移）
左移操作符 (<<) 将第一个操作数向左移动指定位数，左边超出的位数将会被清除，右边将会补零。
``` 
const a = 5; // 00000101
a << 2 // 00010100 = 20
```
### >>（右移）
该操作符会将第一个操作数向右移动指定的位数。向右被移出的位被丢弃，拷贝最左侧的位以填充左侧。由于新的最左侧的位总是和以前相同，符号位没有被改变。所以被称作“符号传播”。
``` 
const a = 5; // 00000101
a >> 2 // 00000001 = 1
```
### 无符号右移
该操作符会将第一个操作数向右移动指定的位数。向右被移出的位被丢弃，左侧用 0 填充。因为符号位变成了 0，所以结果总是非负的。
``` 
const a = -5; // 100...00101原 - 11...111010反 - 11...111011补
a >>> 2 // 0011...1110 = 1073741822
```

原文: 
[Vue3 源码中的位运算，又一个面试考点](https://juejin.cn/post/6946032418520301605?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
