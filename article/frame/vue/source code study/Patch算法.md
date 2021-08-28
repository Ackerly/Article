# Patch算法
## 概述
Vue通过VNode节点渲染DOM时，不是通过当前VNode节点去暴力更新DOM节点，而是对比新旧两个VNode通过patch进行比较，通过对比结果找出差异的属性或节点进行按需更新。patch主要过程有：
- 创建需要新增的节点
- 移除已经废弃的节点
- 移动或修改需要更新的节点

## Vue3的patch优化--patchFlag
当两个节点是同一类型时，两个节点虽然类型相同，但是可能新节点的属性发生变化，所以需要对节点属性进行遍历才能有效判断是否需要更新。
``` 
<div id="bar" :class="foo">Hello World</div>
```
上面代码id是静态属性，而class是动态属性，只需要关注class是否改变即可。Vue3利用patchFlag做到这点，生成AST树后，转换器遍历各个节点，会根据节点的特点打上对应的patchFlag。在patch过程中，仅仅会处理class这一个props，并不是全量比较。减少遍历props的次数，从而实现性能提升。

## patch怎么工作
``` 
const patch: PatchFn = (
  n1,
  n2,
  container,
  anchor = null,
  parentComponent = null,
  parentSuspense = null,
  isSVG = false,
  slotScopeIds = null,
  optimized = false
) => {
  // patching & 不是相同类型的 VNode，则从节点树中卸载
  if (n1 && !isSameVNodeType(n1, n2)) {
    anchor = getNextHostNode(n1)
    unmount(n1, parentComponent, parentSuspense, true)
    n1 = null
  }
	// PatchFlag 是 BAIL 类型，则跳出优化模式
  if (n2.patchFlag === PatchFlags.BAIL) {
    optimized = false
    n2.dynamicChildren = null
  }

  const { type, ref, shapeFlag } = n2
  switch (type) { // 根据 Vnode 类型判断
    case Text: // 文本类型
      processText(n1, n2, container, anchor)
      break
    case Comment: // 注释类型
      processCommentNode(n1, n2, container, anchor)
      break
    case Static: // 静态节点类型
      if (n1 == null) {
        mountStaticNode(n2, container, anchor, isSVG)
      }
      break
    case Fragment: // Fragment 类型
      processFragment(/* 忽略参数 */)
      break
    default:
      if (shapeFlag & ShapeFlags.ELEMENT) { // 元素类型
        processElement(
          n1,
          n2,
          container,
          anchor,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        )
      } else if (shapeFlag & ShapeFlags.COMPONENT) { // 组件类型
        processComponent(/* 忽略参数 */)
      } else if (shapeFlag & ShapeFlags.TELEPORT) { // TELEPORT 类型
        ;(type as typeof TeleportImpl).process(/* 忽略参数 */)
      }
  }
}
```
上面代码中n1和n2是待比较的两个节点，n1位旧节点，n2位新节点。container 是新节点的容器，而 anchor 是一个锚点，用来标识当我们对新旧节点做增删或移动等操作时，以哪个节点为参照物。  
- 第一个 if 条件，当两个节点的类型不同，则直接卸载旧节点。  
- 第二个 if 分支条件，新节点的 patchFlag 的值是 BAIL ，优化模式会被关闭。
- 接下来 patch 函数会通过 switch case 来判断节点类型，并分别对不同节点类型执行不同的操作。
## 元素的Patch过程--processElement
processElement 这个函数的逻辑是存在旧节点，则继续通过 patch 比较新旧两个节点，否则直接挂载新节点。
``` 
const processElement = (
  n1: VNode | null,
  n2: VNode,
  container: RendererElement,
  anchor: RendererNode | null,
  parentComponent: ComponentInternalInstance | null,
  parentSuspense: SuspenseBoundary | null,
  isSVG: boolean,
  slotScopeIds: string[] | null,
  optimized: boolean
) => {
  // 如果旧节点不存在
  if (n1 == null) {
    mountElement(
      n2,
      container,
      anchor
      /* 后续参数省略 */
    )
  } else {
    patchElement(
      n1,
      n2,
      parentComponent,
      parentSuspense,
      isSVG,
      slotScopeIds,
      optimized
    )
  }
}
```
### 比较子节点--patchElement
在元素类型的 patch 过程中，Vue3 首先会将新旧节点的 props 声明提取出来，因为之后需要对 props 进行 patch 比较。  
在比较开始之前会触发一些钩子，比如 VNode 自身的钩子：onVnodeBeforeUpdate，以及元素上绑定的指令的钩子 beforeUpdate。  
``` 
if ((vnodeHook = newProps.onVnodeBeforeUpdate)) {
  invokeVNodeHook(vnodeHook, parentComponent, n2, n1)
}
if (dirs) {
  invokeDirectiveHook(n2, n1, parentComponent, 'beforeUpdate')
}
```
### 更新属性
如果元素被标记patchFlag，则通过patchFlag进行比较，否则会全量diff元素中的props
``` 
if (patchFlag > 0) {
  if (patchFlag & PatchFlags.FULL_PROPS) {
    // 如果元素的 props 中含有动态的 key，则需要全量比较
    patchProps(
      el,
      n2,
      oldProps,
      newProps,
      parentComponent,
      parentSuspense,
      isSVG
    )
  } else {
    if (patchFlag & PatchFlags.CLASS) {
      if (oldProps.class !== newProps.class) {
        hostPatchProp(el, 'class', null, newProps.class, isSVG)
      }
    }

    if (patchFlag & PatchFlags.STYLE) {
      hostPatchProp(el, 'style', oldProps.style, newProps.style, isSVG)
    }

    if (patchFlag & PatchFlags.PROPS) {
      const propsToUpdate = n2.dynamicProps!
      for (let i = 0; i < propsToUpdate.length; i++) {
        const key = propsToUpdate[i]
        const prev = oldProps[key]
        const next = newProps[key]
        if (
          next !== prev ||
          (hostForcePatchProp && hostForcePatchProp(el, key))
        ) {
          hostPatchProp(
            el,
            key,
            prev,
            next,
            isSVG,
            n1.children as VNode[],
            parentComponent,
            parentSuspense,
            unmountChildren
          )
        }
      }
    }
  }

  if (patchFlag & PatchFlags.TEXT) {
    if (n1.children !== n2.children) {
      hostSetElementText(el, n2.children as string)
    }
  }
} else if (!optimized && dynamicChildren == null) {
  patchProps(
    el,
    n2,
    oldProps,
    newProps,
    parentComponent,
    parentSuspense,
    isSVG
  )
}
```
- patchFlag 为 FULL_PROPS 时,说明此时的元素中，可能包含了动态的 key ，需要进行全量的 props diff。
- patchFlag 为 CLASS 时,当新旧节点的 class 不一致时，此时会对 class 进行 patch，新旧节点的 class 属性完全一致时，不需要进行任何操作。这个 Flag 标记会在元素有动态的 class 绑定时加入。
- patchFlag 为 STYLE 时，会对 style 进行更新，这是每次 patch 都会进行的，这个 Flag 会在有动态 style 绑定时被加入。
-  patchFlag 为 PROPS 时，需要注意这个 Flag 会在元素拥有动态的属性或者 attrs 绑定时添加，不同于 class 和 style，这些动态的prop 或 attrs 的 key 会被保存下来以便于更快速的迭代。
   - PROPS 的比较会将新节点的动态属性提取出来，并遍历这个这个属性中所有的 key，当新旧属性不一致，或者该 key 需要强制更新时，则调用 hostPatchProp 对属性进行更新。
- patchFlag 为 TEXT 时，如果新旧节点中的子节点是文本发生变化，则调用 hostSetElementText 进行更新。这个 flag 会在元素的子节点只包含动态文本时被添加。
- 最后一个else，若当前不存在优化标记，并且动态子节点也不存在，则直接对 props 进行全量 diff，通过 patchProps 这个函数完成。

### 更新子节点--patchChildren
- 根据 patchFlag 进行判断
    -  如果 patchFlag 是存在 key 值的 Fragment：KEYED_FRAGMENT，则调用 patchKeyedChildren 来继续处理子节点。
    - 如果 patchFlag 是没有设置 key 值的 Fragment: UNKEYED_FRAGMENT，则调用 patchUnkeyedChildren 处理没有 key 值的子节点。
- 根据 shapeFlag （元素类型标记）进行判断
    - 如果新子节点是文本类型，而旧子节点是数组类型，则直接卸载旧节点的子节点。
    - 如果新旧节点类型一致，则直接更新新子节点的文本
    - 如果旧子节点类型是数组类型，新子节点也是数组类型，则调用 patchKeyedChildren 进行完整的 diff。
    - 如果旧子节点类型是数组类型，新子节点不是数组类型，说明不存在新子节点，直接从树中卸载旧节点即可。
    - 如果旧子节点是文本类型，由于已经在一开始就判断过新子节点是否为文本类型，那么此时可以肯定新子节点肯定不为文本类型，则可以直接将元素的文本置为空字符串。
    - 如果新子节点是类型为数组类型，而旧子节点不为数组，说明此时需要在树中挂载新子节点，进行 mount 操作即可。
## 子节点更新策略
如何高效的做 diff 算法，最重要的性能瓶颈就是如何更快速的对树中的子节点进行比较，得出需要进行什么具体操作，如果按正常思维去比较，那么时间复杂度至少为 O(n^3)，那么 1000 个子节点的树在进行比较时，至少需要 10 亿次比较，那么无疑这个操作是非常昂贵的。
### Vue2 的子节点优化策略
四类优化策略：
- 新子节点索引在最前面与旧子节点索引在最前面
- 新子节索引在末尾与旧子节点索引在末尾
- 新子节索引在末尾与旧子节点索引在最前面
- 新子节索引在末尾与旧子节点索引在末尾
### Vue3的节点更新
``` 
const patchKeyedChildren = (
  c1: VNode[],
  c2: VNodeArrayChildren,
  container: RendererElement,
  parentAnchor: RendererNode | null,
  parentComponent: ComponentInternalInstance | null,
  parentSuspense: SuspenseBoundary | null,
  isSVG: boolean,
  slotScopeIds: string[] | null,
  optimized: boolean
) => {
  let i = 0
  const l2 = c2.length
  let e1 = c1.length - 1 // prev ending index
  let e2 = l2 - 1 // next ending index
  /* 忽略后续逻辑 */
}
```
函数开始声明四个变量
- 遍历子节点的索引 i = 0
- 新子节点长度：l2
- 旧子节点的末尾索引：e1
- 新子节点的末尾索引：e2
### 新前与旧前
新子节点索引在最前面与旧子节点索引在最前面。  
待比较的两个新旧子节点，比较子节点会从两个子节点的起始索引开始：  
当 i = 0 时，比较第 0 个索引，发现 C1 的 A 节点 与 C2 节点的 A 节点 是同一类型的元素，则会对新旧的 A 节点进行 patch 操作，在这里进行 patch 能够递归的去访问 A 节点下的所有子节点，patch 完成后递增索引 i 。
发现 C1 的 B 节点与 C2 的 B 节点也是同一类型的元素，与之前一样对B 节点进行 patch 递归后递增 i 索引
比较第三个子节点时，会发现 C1 的 C 节点与 C2 的 D 节点并不是同一类型的节点，所以会 break 跳出新前与旧前的比较循环，于是新前与旧前的比较结束。此时已经完成了 C1 和 C2 的子节点的前两个节点的比较。
``` 
while (i <= e1 && i <= e2) {
  const n1 = c1[i]
  const n2 = (c2[i] = optimized
    ? cloneIfMounted(c2[i] as VNode)
    : normalizeVNode(c2[i]))
  // 比较 n1 与 n2 是否是同一类型的 VNode
  if (isSameVNodeType(n1, n2)) {
    patch(
      n1,
      n2,
      container,
      null,
      parentComponent,
      parentSuspense,
      isSVG,
      slotScopeIds,
      optimized
    )
  } else {
    // 如果 n1 与 n2 不是同一类型，则 break 出 while 循环
    break
  }
  // 递增 i
  i++
}
```
### 新后与旧后
新子节索引在末尾与旧子节点索引在末尾。  
比较的逻辑：
- 从末尾开始，C1 是 C 节点，而 C2 也是 C 节点，两个节点的类型相同，开始进行 patch 比较，待 patch 完成后，新旧子节点的末尾索引 - 1。
- 进行第二次比较，C1 的末尾是 B 节点，C2 的末尾是 B 节点，类型相同，进行 patch，之后递减尾部索引。
- 进行第三次比较，C1 的末尾节点是 A，C2 的末尾节点是 E，类型不同，break 跳出新后与旧后的比较循环。
``` 
while (i <= e1 && i <= e2) {
  const n1 = c1[e1]
  const n2 = (c2[e2] = optimized
    ? cloneIfMounted(c2[e2] as VNode)
    : normalizeVNode(c2[e2]))
  // 比较 n1 与 n2 是否是同一类型的 VNode
  if (isSameVNodeType(n1, n2)) {
    patch(
      n1,
      n2,
      container,
      null,
      parentComponent,
      parentSuspense,
      isSVG,
      slotScopeIds,
      optimized
    )
  } else {
    // 如果 n1 与 n2 不是同一类型，则 break 出 while 循环
    break
  }
  // 完成 patch 操作后，尾部索引递减
  e1--
  e2--
}
```
### 常规顺序的新子节点挂载
完成了前两轮的比较后，此时往往能在常规的序号上发现一些新子节点中存在，而旧子节点中没有的元素，此时就需要将这些新增的子节点插入。  
当新前与旧前的比较完成后，此时索引 i 已经递增的超过 C1 子节点的长度，此时 i = 2，并且 i 还小于等于 C2 子节点的长度，于是可以判定在新子节点中还有节点没有被遍历到，此时旧子节点已经全部遍历完，所以将未被遍历的子节点全部插入即可。  
``` 
// 当旧子节点被遍历完
if (i > e1) {
  // 新节点还有元素未被遍历完
  if (i <= e2) {
    const nextPos = e2 + 1
    // 确定好锚点元素
    const anchor = nextPos < l2 ? (c2[nextPos] as VNode).el : parentAnchor
    // 遍历剩余的新子节点
    while (i <= e2) {
    	// patch 时第一个参数传入 null，代表没有旧节点，直接将新节点插入即可
      patch(
        null,
        (c2[i] = optimized
          ? cloneIfMounted(c2[i] as VNode)
          : normalizeVNode(c2[i])),
        container,
        anchor,
        parentComponent,
        parentSuspense,
        isSVG,
        slotScopeIds,
        optimized
      )
      i++
    }
  }
}
```
### 常规顺序的移除多余节点
当新子节点已经全部遍历完时，如果此时旧子节点还有元素未被遍历，那么可以判定剩余的旧子节点已经不再需要了，所以直接将剩余旧子节点移除即可。  
C1 与 C2 在通过新后与旧后的比较完成后，此时旧子节点 C1 中还剩余 A 节点，新节点中已经没有需要比较的节点了，所以直接移除 A 节点即可。
``` 
// 如果新子节点已被遍历完
else if (i > e2) {
  // 就子节点未被遍历完
  while (i <= e1) {
    // 调用 unmount 卸载旧子节点
    unmount(c1[i], parentComponent, parentSuspense, true)
    // 递增索引
    i++
  }
}
```
### 未知顺序的子节点比较
通过新前与旧前，新后与旧后的比较后，还剩余中间部分的不确定顺序的节点了。旧子节点中的 CDE，新子节点中的 EDCH。  
patch处理步骤：
- 声明 s1、s2 两个变量，并将此时遍历的前序索引 i 赋值给 s1、s2。s1、s2 分别表示新旧子节点的起始索引。
- 以 s2 为起始节点，e2 为结束条件，遍历新子节点，用新子节点中子节点的 key 为键，索引 i 为值，生成一个 Map 对象, 存放原始索引。
    - 如果此时发现有子节点中有重复的键，就会发出一个所有 Vue 开发者都很熟悉的警告:Duplicate keys found during update xxx, Make sure keys are unique。
``` 
const s1 = i // 旧子节点的起始索引
const s2 = i // 新子节点的起始索引

// 对新子节点，创建一个索引的 map 对象
const keyToNewIndexMap: Map<string | number, number> = new Map()
for (i = s2; i <= e2; i++) {
  const nextChild = (c2[i] = optimized
    ? cloneIfMounted(c2[i] as VNode)
    : normalizeVNode(c2[i]))
  if (nextChild.key != null) {
    // 如果是 DEV 环境，且 keyToNewIndexMap 已经存在当前节点的 key 值，则警告。
    if (__DEV__ && keyToNewIndexMap.has(nextChild.key)) {
      warn(
        `Duplicate keys found during update:`,
        JSON.stringify(nextChild.key),
        `Make sure keys are unique.`
      )
    }
    // 以新子节点的 key 为键，索引为值，存入 map。
    keyToNewIndexMap.set(nextChild.key, i)
  }
}
```
- 声明变量 toBePatched，计算还有几个节点需要被 patch。声明变量 patched = 0，记录 patch 的节点数。
- 声明一个 newIndexToOldIndexMap 的数组，用于后续确定最长递增子序列，newIndexToOldIndexMap 数组大小为 toBePatched 的长度，并将数组内所有元素初始化为 0。
    - newIndexToOldIndexMap，形式是 Map<newIndex, oldIndex>
    - 里面存储的 oldIndex 是索引是偏移 +1 的
    - oldIndex = 0 是一个特殊值，表示新子节点中没有对应的旧子节点。
- 遍历旧子节点，将当前被遍历的子节点标记为 prevChild
    - 如果 patched 大于等于 toBePatched，说明需要被 patch 的节点已经全部比较完毕，则可以将剩余的 prevChild 移除。
    - 否则声明变量 newIndex
    - 如果 prevChild 的 key 不为空，则从 keyToIndexMap 中取 prevChild.key 的值，将获取到的值赋值给 newIndex。
    - 如果 newIndex 没有值，则说明在新子节点中没有对应的旧子节点，直接移除 prevChild 旧子节点。
    - 否则在 newIndexToOldIndexMap 中存下新的索引，并标记当前索引移动的最远位置或增加移动标记，并对新旧子节点进行 patch 比较。
    - 完成 patch 后，将 patched 计数递增。
``` 
/**
 * 遍历旧子节点，尝试 patch 比较需要被 patch 的节点，并且移除不会再出现的子节点
 */
let j
let patched = 0
const toBePatched = e2 - s2 + 1
let moved = false
// 用于跟踪是否有节点发生移动
let maxNewIndexSoFar = 0
// 用于确定最长递增子序列
const newIndexToOldIndexMap = new Array(toBePatched)
for (i = 0; i < toBePatched; i++) newIndexToOldIndexMap[i] = 0

for (i = s1; i <= e1; i++) {
  const prevChild = c1[i]
  if (patched >= toBePatched) {
    // 所有新节点都被 patch 了，所以剩下的只需要移除
    unmount(prevChild, parentComponent, parentSuspense, true)
    continue
  }
  let newIndex
  if (prevChild.key != null) {
    newIndex = keyToNewIndexMap.get(prevChild.key)
  } else {
    // 对于找不到 key 的节点，尝试去定位相同 type 的节点
		/* 忽略逻辑 */
  }
  // 如果旧子节点不能匹配到对应的新子节点，则移除该旧子节点
  if (newIndex === undefined) {
    unmount(prevChild, parentComponent, parentSuspense, true)
  } else {
    // 在 newIndexToOldIndexMap 记录下被 patch 的节点的索引
    newIndexToOldIndexMap[newIndex - s2] = i + 1
    // 如果 newIndex 的索引大于最远移动的索引，则更新
    if (newIndex >= maxNewIndexSoFar) {
      maxNewIndexSoFar = newIndex
    } else {
      // 否则标记 moved 为 true
      moved = true
    }
    // 对新旧子节点进行 patch
    patch(
      prevChild,
      c2[newIndex] as VNode,
      container,
      null,
      parentComponent,
      parentSuspense,
      isSVG,
      slotScopeIds,
      optimized
    )
    // patch 完毕后，递增 patched 计数。
    patched++
  }
}
```
在遍历完旧子节点后，我们已经移除了不必要的旧子节点，并且依旧存在于新节点中的对应的旧子节点都进行了 patch 比较，接下来我们需要遍历一下新子节点，仅仅只从后往前遍历需要被 patch 的部分，目的是对于新增的子节点进行插入，对于需要移动的子节点进行同级的移动。逻辑描述如下：
- 如果有 moved 标记，则从 newIndexToOldIndexMap 中找到最长递增子序列，并将 j 赋值为最长递增子序列数组的末尾索引。
- 从后往前的遍历新子节点，这样可以使我们确定锚点元素的位置。
- 声明 newIndex = s2 + i，即为最后一个需要被 patch 的节点。
- 获取锚点元素。
- 如果这个需要被 patch 的节点，i 索引在 newIndexToOldIndexMap 中的值为 0。还记得我之前提示的，0 是一个特殊值，代表该节点在旧子节点中没有对应的节点吧。那么对于没有对应节点的元素，我们就对它采用插入操作。
- newIndexToOldIndexMap 中有对应索引，但是存在 moved 标记，说明节点可能移动，应该继续判断。
    - 如果 j < 0，说明最长递增子序列中的所有节点都已经处理过。或者当索引 i 不等于最长增长子序列中索引 j 对应的值时，说明该节点并不处在一个相对稳定的位置，则需要进行移动操作。
    - 如果满足上述条件，j 索引递减，不用处理该节点。
``` 
/**
 * 移动和挂载
 */
// 当节点被移动时，创建最长递增子序列
const increasingNewIndexSequence = moved
  ? getSequence(newIndexToOldIndexMap)
  : EMPTY_ARR
j = increasingNewIndexSequence.length - 1
// 为了能方便的获取锚点，选择从后向前遍历
for (i = toBePatched - 1; i >= 0; i--) {
  const nextIndex = s2 + i
  const nextChild = c2[nextIndex] as VNode
  const anchor =
    nextIndex + 1 < l2 ? (c2[nextIndex + 1] as VNode).el : parentAnchor
  if (newIndexToOldIndexMap[i] === 0) {
		// 如果在 newIndexToOldIndexMap 中找不到对应的索引，则新增节点
    patch(
      null,
      nextChild,
      container,
      anchor,
      parentComponent,
      parentSuspense,
      isSVG,
      slotScopeIds,
      optimized
    )
  } else if (moved) {
    // 如果不是一个稳定的子序列，或者当前节点不在递增子序列上时，需要移动
    if (j < 0 || i !== increasingNewIndexSequence[j]) {
      move(nextChild, container, anchor, MoveType.REORDER)
    } else {
      j--
    }
  }
}

作者：Originalee
链接：https://juejin.cn/post/6968585717924495368
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

参考:  
[Vue3 源码解析（五）：Patch 算法](https://juejin.cn/post/6968585717924495368?content_source_url=https%3A%2F%2Fgithub.com%2Fvue3%2Fvue3-News)
