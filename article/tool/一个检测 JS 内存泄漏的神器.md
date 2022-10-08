# 一个检测 JS 内存泄漏的神器
**导致 Web 应用内存过高的原因**  
因为内存泄漏通常不是很明显，在开发过程中，以及做 Code Review 的时候都很难发现，而且在生产环境中通常也很难找到根本原因。虽然主流的 JavaScript 运行时都有垃圾回收机制，那么为什么还会有内存泄漏呢？  
JavaScript 代码中可能会有很多隐藏对象的引用，而隐藏的引用会以许多意想不到的方式导致内存泄漏。  
例如：  
```
var obj = {};
console.log(obj);
obj = null;
```
在 Chrome 中，即使将引用设置为 null ，这段代码也会泄漏 obj 。  
发生这种情况是因为 Chrome 需要保留对打印对象的内部引用，以便以后可以在 Web 控制台中对其进行检查（即使在 Web 控制台没打开的情况下）  
在某些情况下，内存在技术上并没有发生泄漏，而是在用户会话期间线性增长而且没有限制。最常见的原因是客户端缓存没有内置任何释放的逻辑，无限滚动列表没有任何虚拟化的功能，无法在添加新内容时从列表中删除较早的内容。  
没有适当的自动化系统和流程来控制内存，因此防止此类问题的唯一防御措施就是专家通过 Chrome DevTools 定期挖掘内存泄漏，一些大型的项目几乎每天都会有发布和变更，这样的工作方式是不可持续的。  

**MemLab 的工作原理**  
MemLab 通过预定义的测试场景运行无头浏览器并比较和分析 JavaScript 堆快照来发现内存泄漏的问题  
这个过程可以分为下面六个步骤：  
1. 浏览器交互: MemLab 使用 Puppeteer 自动化浏览器，在目标页面上查找泄露的对象
2. 区分堆: 导航到一个页面然后离开它，正常情况下该页面分配的大部分内存也应该被释放，如果没有，可能暗示着存在内存泄漏。MemLab 通过区分 JavaScript 堆并记录在页面 B 上分配的一组对象，这些对象没有在页面 A 上分配，但在重新加载页面 A 时仍然存在，从而发现潜在的内存泄漏
3. 细化内存泄漏列表: 内存泄漏检测器进一步结合了特定框架的知识来细化泄漏对象的列表。例如，React 分配的 Fiber 节点（React 用于渲染虚拟 DOM 的内部数据结构）应该在我们访问多个选项卡后清理时释放
4. 生成 retainer traces: 遍历堆并为每个泄漏的对象生成 retainer traces 。trace 显示了泄漏对象为何以及如何在内存中保持活动状态。打破引用链意味着泄漏的对象将不再可以从 GC 的根访问，因此可以进行垃圾回收。通过一步步地跟踪，就可以找到应该设置为 null 的引用
5. 聚合 retainer traces: 将所有 retainer traces 聚集在一起，并为每个共享相似 retainer traces 的泄漏对象聚合显示为一个跟踪，其中还包括调试信息，例如支配节点和保留大小
6. 报告泄漏: 定期运行 MemLab，以持续收集 retainer traces，任何新的 traces 都会记录到内部仪表板，开发者可以查看每个内存泄漏的 retainer traces 上的对象属性

**MemLab 有哪些能力**  
_内存泄漏检测_  
对于浏览器内存泄漏的检测，MemLab 需要开发者提供的唯一输入就是一个测试场景文件，这个文件定义了如何通过使用 Puppeteer API 和 CSS 选择器覆盖三个回调来与网页交互。MemLab 会自动区分 JavaScript 堆、优化内存泄漏并聚合结果。  
_JavaScript 堆的 Graph-view API_  
MemLab 支持一个自定义的泄漏检测器，作为筛选器回调，应用于每个由目标交互分配的泄漏候选对象，但之后从不释放。泄漏过滤器回调函数可以遍历堆并确定哪些对象是内存泄漏。例如，我们的内置检漏器会跟踪 React Fiber 节点的返回链路，检查 Fiber 节点是否与 React Fiber 树分离  
为了分析每个可能内存泄漏的上下文，MemLab 提供了一个 JavaScript 堆的内存效率图。这可以在不了解 V8 堆快照文件结构的任何领域知识的情况下查询和遍历 JavaScript 堆  
堆中的每个 JavaScript 对象或原生对象都是一个图节点，堆中的每个 JavaScript 引用都是一个图的边。实际应用程序的堆大小通常很大，因此图视图需要在提供直观的面向对象堆遍历 API 的同时提高内存效率。因此，图节点被设计成了虚拟的，不通过 JavaScript 引用进行连接。当分析代码遍历堆时，虚拟图会部分地即时构建图的接触部分。图的任何部分都可以很容易地释放，因为这些虚拟节点彼此之间没有 JavaScript 引用。  
堆视图可以从基于 Chromium 的浏览器、Node.js、Electron 和 Hermes 获取的 JavaScript 堆快照加载。这允许分析复杂的模式并回答诸如 “有多少 React Fiber 节点是备用的 Fiber 节点，它们用于不完整的并发渲染？”之类的问题。  
``` 
import {getHeapFromFile} from '@memlab/heap-analysis';
const heapGraph = await getHeapFromFile(heapFile);
heapGraph.nodes.forEach(node => {
  // heap node traversal
  node.type
  node.references
);
```
_内存断言_  
Node.js 程序或 Jest 测试也可以使用 graph-view API 来获取其自身状态的堆视图，进行自内存检查，并编写各种内存断言  
``` 
import type {IHeapSnapshot} from '@memlab/core';
import {config, takeNodeMinimalHeap, tagObject} from '@memlab/core';

test('memory test', async () => {
  config.muteConsole = true;
  const o1 = {};
  let o2 = {};

  // tag o1 with marker: "memlab-mark-1", does not modify o1 in any way
  tagObject(o1, 'memlab-mark-1');
  // tag o2 with marker: "memlab-mark-2", does not modify o2 in any way
  tagObject(o2, 'memlab-mark-2');

  o2 = null;

  const heap: IHeapSnapshot = await takeNodeMinimalHeap();

  // expect object with marker "memlab-mark-1" exists
  expect(heap.hasObjectWithTag('memlab-mark-1')).toBe(true);

  // expect object with marker "memlab-mark-2" can be GCed
  expect(heap.hasObjectWithTag('memlab-mark-2')).toBe(false);

}, 30000);
```
_内存工具箱_  
除了内存泄漏检测，MemLab 还包括一组内置的 CLI 命令和 API，用于寻找可能的内存优化机会

**Meta 使用 MemLab 的实践**  
在过去的几年中，Meta 一直在使用 MemLab 检测和诊断内存泄漏，并收集了很多有助于优化内存、减少 OOM 崩溃并改善用户体验的手段  
在 2021 年上半年， Facebook.com 上的 OOM 崩溃减少了 50%。  
_React Fiber 节点清理_  
为了渲染组件，React 构建了 Fiber 树 — 一个 React 用于渲染虚拟 DOM 的内部数据结构。虽然 Fiber 树看起来像一棵树，但它是一个双向图，将所有 Fiber 节点、React 组件实例和关联的 HTML DOM 元素强连接起来。理想情况下，React 维护对组件 Fiber 树的根的引用，并防止 Fiber 树被垃圾回收。当一个组件被卸载时，React 会断开组件的根与 Fiber 树的其余部分之间的连接，然后这些部分就可以被垃圾回收了。  
拥有这样的强连接图的缺点是，如果有任何外部引用指向图的任何部分，就无法对整个图进行垃圾回收。例如，下面 export 语句在模块范围级别缓存 React 组件，因此相关的 Fiber 树和分离的 DOM 元素永远不会被释放  
``` 
export const Component = (( 
  <List> ... </List> 
): React.Element<typeof List>);
```
也不仅仅是 React 数据结构要 keep alive ，Hooks 和它们的闭包也可以让各种其他对象保活。这意味着单个 React 组件泄漏可能会导致页面对象的重要部分泄漏，从而导致巨大的内存泄漏  
为了防止 Fiber 树中内存泄漏的级联效应，MemLab 添加了一个树的完整遍历，当组件在 React 18 中卸载时会进行清理。这可以让垃圾回收器在清理未挂载的树方面做得更好一点。这个优化将 Facebook 上的平均内存使用量减少了近 25%，其他使用 React 的站点在升级时也有了很大的改进。你可能会担心这种比较激进的清理方式可能会减慢 React 组件的卸载速度，但令人惊讶的是，由于内存的减少，性能也有显着的提升。  

_string interning_  
通过利用 MemLab 中的 heap analysis API，Meta 团队发现字符串占据了 70% 的堆内存，其中一半的字符串至少有一个重复的实例。（V8 对 string interning 支持的不是很好，这是一种对具有相同值的字符串实例进行重复数据删除的优化。）  
另外很大一部分字符串内存被 Relay 中缓存的键字符串消耗。通过与 Relay 和 React Apps 团队合作，可以在客户端插入和缩短过长的字符串键来优化 Relay 缓存键字符串  
这种优化使 Relay 能够缓存更多数据，允许站点向用户显示更多内容，尤其是在客户端 RAM 有限的情况下。内存 p99 和 OOM 崩溃减少了 20%，页面渲染速度更快，用户体验得到改善，在收入上也有一定提升。

 原文:  
 [推荐一个检测 JS 内存泄漏的神器](https://mp.weixin.qq.com/s/3pWjbpQ_EHBGTdjTnlvh7A)
