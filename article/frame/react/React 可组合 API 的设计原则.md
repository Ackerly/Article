# React 可组合 API 的设计原则
**什么是基于组合的 API**  
HTML 作为一种声明式编程，通过元素组合描述出网页上的具体内容。下面的示例是用 select 实现的下拉列表：  
```
<select id="cars" name="cars">
   <option value="audi">Audi</option>
   <option value="mercedes">Mercedes</option>
 </select>
```
把这种组合应用在 React 的开发中时，它被称作 “复合组件” 模式。它的核心思想是，将多个组件协同工作，以实现某个具体的功能。  
当我们谈到 API 时，可以将组件的 props 看作是组件的 公共 API，而组件可以看做是一个包 (package) 的 API。良好的 API 设计，就像那些困难且不明确的事情一样，需要经常根据反馈不断地花时间去迭代。  
其中的一个挑战就是，API 会有多种不同类型的消费者。一些消费者只需要简单的应用场景；另外一些则对灵活性有更高的要求；除此之外，还有一些消费者要对那些难以预见的应用场景进行深度的定制。对于那些常用的前端组件，基于组合的 API，可以很好的抵御那些不可预见的应用场景以及不断变化的需求。  

**设计可组合的组件**  
如何将组件分解到正确的级别？在纯的自底向上的方法中，我们可能会创建太多的小组件，实际上并不需要创建这么多。自顶向下的方法（这往往更常见）也是不够的，这会导致巨石组件，这样的组件会有太多的 props 和太多的功能，很快就变得难于管理和维护。  
当我们面临一个模棱两可的问题时，从最终用户开始着手去解决，以终为始，不失为一个好的办法。我们可以从消费者使用的组件 API 开始，去分解和设计组件。  
稳定依赖原则可以帮助我们设计组件的 API，它有两个核心思想：  
- 作为组件或包的消费者，我们希望我们依赖的东西能够保持稳定，不会随着时间的推移而发生变化，在此基础上完成我们的工作。
- 作为组件或包的开发者，我们希望将可能发生变化的东西封装起来，以保护消费者免于因任何变化而受到影响。

如何确定哪些组件是稳定的？  
假如我们对什么是 Tabs 没有完整的概念，那么用视觉上看到的主要元素来设计组件，在某种程度上也是靠谱的。与更抽象的实体相比，这也是为 UI 元素设计 API 的便利之处。  
在这种情况下，我们可以把 Tabs 想象成一个 list 列表（单击可以改变展示的内容）和 一个 content 内容区（根据当前选择的 tab 展示不同的内容）。基于此，我们把 Tabs 组件库的 API 设计成下面的样子（与 Reakit 和其他类似的开源组件库中的 API 相同）。  
```
import { Tabs, TabsList, Tab, TabPanel } from '@mycoolpackage/tabs'

 <Tabs>
   <TabsList>
     <Tab>first</Tab>
     <Tab>second</Tab>
   </TabsList>
   <TabPanel>
     hey there
   </TabPanel>
   <TabPanel>
     friend
   </TabPanel>
 </Tabs>
```
与 HTML 的 select 元素类似，这些组件组合在一起实现某个功能，并且将状态分配给各个组件处理。遵循稳定依赖原则，意味着当我们调整组件的内部实现时，使用这些组件的消费者不需要做任何处理。  
对消费者来说它既简单又灵活，但我们如何实现这些组件的底层逻辑呢？这才是我们需要面对的关键挑战。  

## 要解决的潜在的问题
**组件之间的内部逻辑**  
当我们完成对组件拆解之后，如何实现组件的交互逻辑，是我们面临的第一个问题。我们希望组件能够解耦，也就是说，它们感知不到彼此的存在。但我们也需要它们能够在一起实现特定的功能。  
比如，微服务面临的挑战之一，是在不产生耦合的情况下，连接所有节点以使它们协作。同理，微型前端也是如此。  
在我们的 Tabs 组件中，tab panels 按顺序嵌入到顶层 Tabs 中。我们需要实现组件内部交互逻辑，根据所选的 tab 来渲染出对应的内容。  

**渲染任意的子元素**  
如何处理那些包裹在我们组件外面的组件
```
<Tabs>
    <TabsList>
      <CustomTabComponent />
      // etc ...
      <ToolTip message="cool">
        <Tab>Second</Tab>
      </ToolTip>
    </TabsList>
    // etc ...
    <AnotherCustomThingHere />
    <Thing>
      <TabPanel>
      // etc ..
      </TabPanel>
    </Thing>
    //...
  </Tabs>
```
由于 Tabs 和它所包含的组件要按顺序在子树中进行关联，因此我们需要记录不同的索引，以便能够正确的选择上一项和下一项  
我们也需要处理焦点管理、键盘导航之类的事情，有多少组件来包裹我们的组件是不确定的。  
消费者可以在组件之间插入任意数量组件或元素，这对于我们也是个挑战。  
需要忽略那些不是我们的组件，来保持正确的相对顺序。这里有两种实现方式：
1. 在 React 中跟踪所有我们的组件  
    将元素及其相对顺序存储在某个数据结构中。但是这种处理方法有点复杂。因为在默认情况下，我们不知道组件在树中的位置，所以我们需要以某种方式跟踪所有子组件。  
    一个可行的方法是，在子组件装载和卸载时，对它们进行 “注册” 和 “注销”，这样顶级的父组件可以直接访问它们。Reach UI 和 Chakra 组件库采用的就是这种方法，这种方法的底层实现比较复杂，但对消费者来说很灵活。  
2. 从 DOM 读取
    在组件底层的 HTML 上附加唯一 ID 或 data-attribute。我们使用这些存储在 HTML 属性中的索引来查询 DOM，就可以获取下一个或上一个元素  
    将在 React 中访问 DOM，虽然它与常用的 React 代码风格背道而驰，但是胜在实现起来比较简单  
    我们需要深入了解规则，然后知道什么时候可以打破规则。封装可以让消费者不必关注实现的细节，消费者只需将组件组合在一起，就可以实现想要的功能  
    封装和稳定依赖原则的最大好处是，把所有混乱的细节进行隐藏。  

## 完成我们的 Tabs 组件开发  
### 处理逻辑  
**元素复制**  
React 提供了一个用于元素复制的 API，可方便我们传递一些 props。我们的 Tabs 组件可以把所有属性和事件传递给子组件，而组件的使用者不会看到任何代码  
```
React.Children.map(children, (child, index) => (
     React.cloneElement(child, {
       isSelected: index === selectedIndex,
       // other stuff we can pass as props "under the hood"
       // so consumers of this can simply compose JSX elements
     })
   ))
```
**元素复制的局限**  
元素复制的主要问题是不够灵活，例如它不能适用于组件被包裹的场景：  
```
<Tabs>
   <TabsList>
     // can't do this
     <ToolTip message="oops"><Tab />One</Tooltip>
   </TabsList>
 </Tabs>
```
这种场景下我们复制的是 ToolTip 组件而不是 Tab 组件。我们也可以深度拷贝所有的元素，但是需要遍历所有子组件才能找到正确的 Tab 组件，增加了实现成本，而且对于那些不是以子组件渲染的场景也不适用：  
```
const MyCustomTab = () => <Tab>argh</Tab>

 <TabsList>
     <Tab>hey</Tab>
     <MyCustomTab />
 </TabsList>
```
如果项目使用了 Typescript，cloneElement 也不能保证类型安全。这个方法尽管是最简单的，由于不够灵活，所以我们在这里不使用它。  
**Render props**  
props 将所有必要的数据和属性（例如内部的 onChange）暴露给消费者，消费者可以使用这些属性来 “连接” 那些自定义的组件  
这是在实践中使用了控制反转的一个例子，两个独立的组件能够灵活地一起工作，最终实现了某些功能，换句话说这就是 -- 组合  
假设我们有一个通用的内联可编辑的功能组件（InlineEdit），该字段具有只读视图（readView），单击时会切换到编辑视图（editView），消费者可以在其中插入自定义组件：  
```
<InlineEdit
     editView={(props) => (
       // expose the necessary attributes for consumers
       // so they can place them directly where they need to go
       // in order for things to work together
       <SomeWrappingThing>
         <AnotherWrappingThing>
           <TextField {...props }/>
         </AnotherWrappingThing>
       </SomeWrappingThing>
     )}}
     // ... etc
   />
```
 对于这种的独立组件来说，这是一种很好的方法。对于我们的 Tabs 复合组件，也是可行的；但对于那些想要简单明了的 Tabs 组件的消费者来说，这些是额外的负担。  
值得注意的是，Reach UI 的 tab API 允许常规 children 和函数作为 children，以实现其 API 的最大程度的灵活性。  

**使用 React Context**  
这是一种灵活而直接的方法。其中子组件从共享 Context 中读取数据。我们可以采用这种方法，但是前提是要解决 Context 的渲染问题。要将 state 分解为合适的大小，来优化重新渲染。在拆解组件的时候，我们要先回答 “每个组件的完整且最小的状态是什么”。  
**开发组件**  
从 state 开始，将 state 分解到每个单独的 Context 中  
```
const TabContext = createContext(null)
 const TabListContext = createContext(null)
 const TabPanelContext = createContext(null)

 export const useTab = () => {
   const tabData = useContext(TabContext)
   if (tabData == null) {
     throw Error('A Tab must have a TabList parent')
   }
   return tabData
 }

 export const useTabPanel = () => {
   const tabPanelData = useContext(TabPanelContext)
   if (tabPanelData == null) {
     throw Error('A TabPanel must have a Tabs parent')
   }
   return tabPanelData
 }

 export const useTabList = () => {
   const tabListData = useContext(TabListContext)
   if (tabListData == null) {
     throw Error('A TabList must have a Tabs parent')
   }
   return tabListData
 }
```
将这些 state 分解成较小的 state，而不是单一的大型 context，我们可以获得以下几点收益：  
- 较小的状态更容易优化重新渲染。
- 状态管理的边界比较清晰（单一职责）。
- 如果消费者需要实现一个完全自定义的 Tab 版本，他们可以导入这些状态管理 Hooks，以便进行逻辑复用。因此，至少我们可以复用公共状态管理逻辑。

这些 context provider 中的每一个都将提供数据和可访问性属性，这些属性将传递到 UI 组件中，以连接所有内容并实现我们的 Tabs 组件。  
Tabs 为 TabPanel 提供了数据，TabsList 为 Tab 提供了 context。因此，在示例中，需要确保在预期的父 context 中渲染各个组件。  

**Tab 和 TabPanel**  
Tab 和 TabPanel 是简单的 UI 组件，它们从 context 中获得必要的状态并渲染子组件  
```
 export const Tab = ({ children }) => {
   const tabAttributes = useTab()
   return (
     <div {...tabAttributes}>
       {children}
     </div>
   )
 }

 export const TabPanel = ({ children }) => {
   const tabPanelAttributes = useTabPanel()
   return (
     <div {...tabPanelAttributes}>
       {children}
     </div>
   )
 }
```
**TabsList**  
TabsList 组件的简化版本。它负责管理 Tabs 中的列表，用户可以与它交互来更改展现的内容。  
```
export const TabsList = ({ children }) => {
   // provided by top level Tabs component coming up next
   const { tabsId, currentTabIndex, onTabChange } = useTabList()
   // store a reference to the DOM element so we can select via id
   // and manage the focus states
   const ref = createRef()

   const selectTabByIndex = (index) => {
     const selectedTab = ref.current.querySelector(
       `[id=${tabsId}-${index}]`
     )
     selectedTab.focus()
     onTabChange(index)
   }
   // we would handle keyboard events here
   // things like selecting with left and right arrow keys
   const onKeyDown = () => {
    // ...
   }
   // .. some other stuff - again we're omitting styles etc
   return (
     <div role="tablist" ref={ref}>
       {React.Children.map(children, (child, index) => {
           const isSelected = index === currentTabIndex
           return (
             <TabContext.Provider
               // (!) in real life this would need to be restructured
               // (!) and memoized to use a stable references everywhere
               value={{
                 key: `${tabsId}-${index}`,
                 id: `${tabsId}-${index}`,
                 role: 'tab',
                 'aria-setsize': length,
                 'aria-posinset': index + 1,
                 'aria-selected': isSelected,
                 'aria-controls': `${tabsId}-${index}-tab`,
                 // managing focussability
                 tabIndex: isSelected ? 0 : -1,
                 onClick: () => selectTabByIndex(index),
                 onKeyDown,
               }}
             >
               {child}
             </TabContext.Provider>
           )
         }
       )}
     </div>
   )
 }
```

**Tabs**  
它将渲染 tab 列表和当前选中的 tab 面板。它将必要的数据传递给 TabsList 和 TabPanel 组件。  
```
export const Tabs = ({ id, children, testId }) => {
   const [selectedTabIndex, setSelectedTabIndex] = useState(0)
   const childrenArray = React.Children.toArray(children)
   // with this API we expect the first child to be a list of tabs
   // followed by a list of tab panels that correspond to those tabs
   // the ordering is determined by the position of the elements
   // that are passed in as children
   const [tabList, ...tabPanels] = childrenArray
   // (!) in a real impl we'd memoize all this stuff
   // (!) and restructure things so everything has a stable reference
   // (!) including the values pass to the providers below
   const onTabChange = (index) => {
     setSelectedTabIndex(index)
   }
   return (
     <div data-testId={testId}>
       <TabListContext.Provider
         value={{ selected: selectedTabIndex, onTabChange, tabsId: id }}
       >
         {tabList}
       </TabListContext.Provider>
       <TabPanelsContext.Provider
         value={{
           role: 'tabpanel',
           id: `${id}-${selectedTabIndex}-tab`,
           'aria-labelledby': `${id}-${selectedTabIndex}`,
         }}
       >
         {tabPanels[selectedTabIndex]}
       </TabPanelsContext.Provider>
     </div>
   )
 }
```
需要考虑的其他方面：  
- 添加一个 “受控” 版本，消费者可以连接到该版本，以实现自己的 onChange 事件
- 扩展 API 以包括默认选定的 tab、tab 对齐样式等
- 优化重新渲染
- 处理 RTL 样式和国际化
- 确保类型安全
- 缓存被访问过的 tab 的选项（当我们点选其他 tab 时，将卸载当前选定的 tab）

假设 TabsList 是 Tabs 的第一个子元素。这里假定 Tabs 始终位于顶层。虽然灵活性受限，但如果不加以管理，过多的灵活性会成为一种诅咒。这里需要有一个很好的平衡。  
如果这是一个组织中更大系统中的一部分，那么在保持体验视觉的一致性是很重要的。因此，我们可能希望强制 Tabs 始终位于顶部。  
如果要追求完全的灵活（如 Reach），需要做更多的工作，同时灵活性会带来用户体验上的变化。这算是一种权衡。  
将讨论如将分层与组合结合到一起工作，在理想情况下，我们将有一个灵活的基础组件库，用于构建更特定类型的组件。就像 tabs 总是在顶部一样。  
对于一个简单的 tabs 组件来说，这需要做很多的工作。是的，组件 API 设计可能是一个艰难的平衡。在构建真正可访问和可重用的组件时，就是要在底层实现上做很多事情。  
这也是为什么 Web Components 有如此大潜力的原因之一，它可以使我们不必不断地重新实现这些类型的组件。  

## 如何扩展  
**团队之间共享代码**  
团队需要使用已有的组件，但有一些变化。如果这些组件在底层没有组合得很好，那么通常很难进行重构和扩展以支持新的场景。  
为了减少风险，通常对代码复制粘贴会更加保险，然后按需作出调整，就可以使用这些代码了。这会导致大量相似但又略有不同的组件。这现象是一种常见的反模式，被称为 “散弹式修改”。  
**性能**  
首先是 bundle 包的大小。使用边界清晰的、较小的独立组件，可以更容易地对不立即需要的组件进行代码拆分，或者在交互中按需加载这些组件。消费者应该只为他们使用的东西付费。  
其次是运行时性能（React 重新渲染）。与每次重新渲染一个巨石组件相比，独立的组件更容易被 React 看到那些内容需要被重新渲染。  

## 向上组合
我们的想法是从一个顶级组件开始，然后朝着它自底向上地构建。在自底而上构建之前，我们需要先确定一个目标，以了解我们正在朝着什么方向构建。  
我们的 Tabs 组件是由更小的组件组成的。这种模式可以一直应用到应用程序的顶部。功能源于不同组件之间的组合；应用程序则是不同功能的组合。组合不断地上升。  
通过分层的方式，来理解 React 应用程序中更高层级的组合：  
- 基础层：一系列通用设计标记、常量和变量，这些会被可共享复用的组件库使用。
- 基础组件：组件库使用的类库，这些类库是基础层的组合，用于帮助构建组件库中可用的组件。比如，Pressable 组件被 Button 和 Link 使用。
- 可共享（可复用）组件库：共享类库和基础组件的组合，以提供一组常用的 UI 元素，如 Button 和 Tabs 等。这些是支撑更上层的 “地基”。
- 特定产品中通用可共享组件的适配：例如，产品中通常使用的 “有机体”，可能将几个组件库组件包装在一起。在组织内的共享功能。
- 特定产品中的专用组件：例如，在我们的产品中，Tabs 组件可能需要调用 API 来确定要渲染的 tab 和内容。可以将其包装为 ＜ProductTabs/＞，在它里面使用我们的 Tab 组件。




原文:  
[React 可组合 API 的设计原则](https://mp.weixin.qq.com/s/hW5VcbjpFCoGOp3BPsnAZQ)