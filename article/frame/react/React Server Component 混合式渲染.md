# React Server Component: 混合式渲染
## 术语介绍
React 组件区分为以下三种：  
1. Server Component：服务端渲染组件，拥有访问数据库、访问本地文件等能力。无法绑定事件对象，即不拥有交互性。
2. Client Component：客户端渲染组件，拥有交互性。
3. Share Component：既可以在服务端渲染，又可以在客户端渲染。具体如何渲染取决于谁引入了它。当被服务端组件引入的时候会由服务端渲染当被客户端组件引入的时候会由客户端渲染。

React 官方暂定通过「文件名后缀」来区分这三种组件：  
- Server Component 需要以 .server.js(/ts/jsx/tsx) 为后缀
- Client Component 需要以 .client.js(/ts/jsx/tsx) 为后缀
- Share Component 以 .js(/ts/jsx/tsx) 为后缀

## 混合渲染
Server Component 是在服务端渲染的组件，而 Client Component 是在客户端渲染的组件。  
与类似 SSR , React 在服务端将 Server Component 渲染好后传输给客户端，客户端接受到 HTML 和 JS Bundle 后进行组件的事件绑定。不同的是：Server Component 只进行服务端渲染，不会进行浏览器端的 hyration (注水)，总的来说页面由 Client Component 和 Server Component 混合渲染。  

## React 是进行混合渲染的？
**渲染入口**  
浏览器请求到 HTML 后，请求入口文件 - main.js, 里面包含了 React Runtime 与 Client Root，Client Root 执行创建一个 Context，用来保存客户端状态，与此同时，客户端向服务端发出 /react 请求。  
``` 
// Root.client.jsx 伪代码

 function Root() {
     const [data, setData] = useState({});

     // 向服务端发送请求
     const componentResponse = useServerResponse(data);
     return (
         <DataContext.Provider value={[data, setData]}>
             componentResponse.render();
         </DataContext.Provider>
     );
 }
```
这里没有渲染任何真实的 DOM, 真正的渲染会等 response 返回 Component 后才开始。  

**请求服务端组件**  
Client Root 代码执行后，浏览器会向服务端发送一个带有 data 数据的请求，服务端接收到请求，则进行服务端渲染。  
服务端将从 Server Component Root 开始渲染，一颗混合组件树将在服务端渲染成一个巨大的 VNode。  
混合组件树会被渲染成这样一个对象，它带有 React 组件所有必要的信息。  
``` 
module.exports = {
     tag: 'Server Root',
     props: {...},
     children: [
         { tag: "Client Component1", props: {...}: children: [] },
         { tag: "Server Component1", props: {...}: children: [
             { tag: "Server Component2", props: {...}: children: [] },
             { tag: "Server Component3", props: {...}: children: [] },
         ]}
     ]
 }
```
不仅仅是这样一个对象，由于 Client Comopnent 需要 Hydration, React 会将这部分必须要的信息也返回回去。  
React 最终会返回一个可解析的 Json 序列 Map。  
``` 
M1:{"id":"./src/BlogMenu.client.js","chunks":["client0"],"name":"xxx"}
 J0:["$","main", null, ["]]
```
- M: 代表 Client Comopnent 所需的 Chunk 信息
- J: 代表 Server Compnent 渲染出的类 react element 格式的字符串

**React Runtime 渲染**  
组件数据返回给浏览器后，React Runtime 开始工作，将返回的 VNode 渲染出真正的 HTML。与此同时，发出请求，请求 Client Component 所需的 JS Bundle。  
当浏览器请求到 Js Bundle 后，React 就可以进行选择性 Hydration（Selective Hydration）。  
React 团队传输组件数据选择了流式传输，这也意味着 React Runtime 无需等待所有数据获取完后才开始处理数据。  

_启动流程_  
- 浏览器加载 React Runtime, Client Root 等 js 代码
- 执行 Client Root 代码，向服务端发出请求
- 服务端接收到请求，开始渲染组件树
- 服务端将渲染好的组件树以字符串的信息返回给浏览器
- React Runtime 开始渲染组件且向服务端请求 Client Component Js Bundle 进行选择性 Hydration（注水)

_Client <-> Server 如何通信？_  
Client Component 与 Server Component 有着天然的环境隔离，他们是如何互相通信的呢？  

_Server Component -> Client Component_  
在服务端的渲染都是从 Server Root Component 开始的，Server Component 可以简单的通过 props 向 Client Component 传递数据。  
``` 
import ClientComponent from "./ClientComponent";

 const ServerRootComponent = () => {
     return <ClientComponent title="xxx" />
 };
```
> 但需要注意的是：这里传递的数据必须是可序列化的，也就是说你无法通过传递 Function 等数据。

_Client Component -> Server Component_  
Client Component 组件通过 HTTP 向服务端组件传输信息。Server Component 通过 props 的信息接收数据，当 Server Component 拿到新的 props 时会进行重新渲染，之后通过网络的手段发送给浏览器，通过 React Runtime 渲染在浏览器渲染出最新的 Server Component UI。  
这也是 Server Component 非常明显的劣势：渲染流程极度依赖网络。  
``` 
// Client Component
 function ClientComponent() {
     const sendRequest = (props) => {
         const payload = JSON.stringify(props);
         fetch(`http://xxxx:8080/react?payload=${payload}`)
     }
     return (
         <button
            onclick = {() => sendRequest({ messgae: "something" })}
         >
             Click me, send some to server
         </button>
     )
 }
 // Serve Component
 const ServerRootComponent = ({ messgae: "something" }) => {
     return <ClientComponent title="xxx" />
 };
```

### Server Component 所带来的优势
**更小的 Bundle 体积**  
通常情况下，我们在前端开发上使用很多依赖包，但实际上这些依赖包的引入会增大代码体积，增加 bundle 加载时间，降低用户首屏加载的体验。  
例如在页面上渲染 MarkDown ，我们不得不引入相应的渲染库，以下面的 demo 为例，不知不觉我们引入了 240 kb 的 js 代码，而且往往这种大型第三方类库是没办法进行 tree-shaking。  
``` 
// NOTE: *before* Server Components

 import marked from 'marked'; // 35.9K (11.2K gzipped)
 import sanitizeHtml from 'sanitize-html'; // 206K (63.3K gzipped)

 function NoteWithMarkdown({text}) {
   const html = sanitizeHtml(marked(text));
   return (/* render */);
 }
```
为了某一个计算任务，需要将大型 js 第三方库传输到用户浏览器上，浏览器再进行解析执行它来创造计算任务的 runtime, 最后才是计算。从用户的角度来讲：「我还没见到网页内容，你就占用了我较大带宽和 CPU 资源，是何居心」。  
然而这一切都是可以省去的，可以利用 SSR 让 React 在服务端先渲染，再将渲染后的 html 发送给用户。从这一方面看，Server Component 和 SSR 很类似，但不同的是 SSR 只能适用于首页渲染，Server Component 在用户交互的过程中也是服务端渲染，Server Component 传输的也不是 html 文本，而是 json。  
Server Component 在服务端渲染好之后会将一段类 React 组件 json 数据发送给浏览器，浏览器中的 React Runtime 接收到这段 json 数据 后，将它渲染成 HTML。  
举一个更加极端的例子：若用户无交互性组件，所以组件都可以在服务端渲染，那么所有 UI 渲染都将走「浏览器接收 "类 react element 文本格式" 的数据，React Runtime 渲染」的形式进行渲染。  
那么除了一些 Runtime， 我们无需其他 JS Bundle。而 Runtime 的体积是不会随着项目的增大而增大的，这种常数系数级体积也可以称为 "Zero-Bundle-Size"。因此官方这称为: "Zero-Bundle-Size Components"。  

**更好的使用服务端能力**  
为了获取数据，前端通常需要请求后端接口，这是因为浏览器是没办法直接访问数据库的。但既然我们都借助服务端的能力了，那我们当然可以直接访问数据库，React 在服务器上将数据渲染进组件。  
通过自由整合后端能力，我们可以解决:网络往返过多和数据冗余问题。甚至我们可以根据业务场景自由地决定数据存储位置，是存储在内存中、还是存储在文件中或者存储在数据库。  
除了数据获取，还可以再开一些 "脑洞"。  
- 我们可以在 Server Component 的渲染过程中将一些高性能计算任务交付给其他语言，如 C++，Rust。这不是必须的，但你可以这么做

Nodejs 拥有什么样的能力，组件就能拥有什么能力。

**更好的自动化 Code Split**  
在过去，可以通过 React 提供的 lazy + Suspense 进行代码分割。这种方案在某些场景 (如 SSR) 下无法使用，社区比较成熟的方案是使用第三方类库 @loadable 。然而无论是使用哪一种，都会有以下两个问题：  
- Code Split 需要用户进行手动分割，自行确认分割点。
- 与其说是 Code Split，其实更偏向懒加载。也就是说，只有加载到了代码切割点，才会去即时加载所切割好的代码。这里还是存在一个加载等待的问题，削减了 code split 给性能所带来的好处

React Server Component 将所有 Client Component 的导入视为潜在的分割点。也就是说，你只需要正常的按分模块思维去组织你的代码。React 会自动帮你分割  
``` 
import ClientComponent1 from './ClientComponent1';

 function ServerComponent() {
     return (
         <div>
             <ClientComponent1 />
         </div>
     )
 }
```
框架侧可以介入 Server Component 的渲染结果，因此上层框架可以根据当前请求的上下文来预测用户的下一个动作，从而去预加载对应的 js 代码。  
**避免高度抽象所带来的性能负担**  
React server component 通过在服务器上的实时编译和渲染，将抽象层在服务器进行剥离，从而降低了抽象层在客户端运行时所带来的性能开销。  
举个例子，如果一个组件为了可配置行，被多个 wrapper 包了很多层。但事实上，这些代码最终只是渲染为一个 <div>。如果把这个组件改造为 server component 的话，那么我们只需要往客户端返回一个 <div> 字符串即可。下面例子，我们通过把这个组件改造为 server component，那么，我们大大降低网络传输的资源大小和客户端运行时的性能开销：  
``` 
 // Note.server.js
 // ...imports...

 function Note({id}) {
   const note = db.notes.get(id);
   return <NoteWithMarkdown note={note} />;
 }

 // NoteWithMarkdown.server.js
 // ...imports...

 function NoteWithMarkdown({note}) {
   const html = sanitizeHtml(marked(note.text));
   return <div ... />;
 }

 // client sees:
 <div>
   <!-- markdown output here -->
 </div>
```
可以通过在 Server Component ，将 HOC 组件进行渲染，可能渲染到最后只是一个 <div> 我们就无需将 bundle 传输过去，也无需让浏览器消耗性能去渲染。  

### Sever Component 可能存在的劣势
**弱网情况下的交互体验**  
React Server Component 的逻辑， 他的渲染流程依靠网络。服务端渲染完毕后将类 React 组件字符串的数据传输给浏览器，浏览器中的 Runtime React 再进行渲染。显然，在弱网环境下，数据传输会很慢，渲染也会因为网速而推迟，极大的降低了用户的体验。  
Server Component 比较难能可贵的是，它跟其他技术并不是互斥的，而是可以结合到一块。例如：我们完全可以将 Server Component 的计算渲染放在边缘设备上进行计算，在一定程度上能给降低网络延迟带来的问题。  

**开发者的心智负担**  
在 React Server Component 推出之后，开发者在开发的过程中需要去思考: 我这个组件是 Server Component 还是 Client Component，在这一方面会给开发者增加额外的心智负担  
Nextjs 前一段时间发布了 v13，目前已实现了 Server & Client Component 。参考 Next13 的方案，默认情况下开发者开发的组件都是 Server Component ，当你判断这个组件需要交互或者调用 DOM, BOM 相关 API 时，则标记组件为 Client Component。  
> 「默认走 Server Component，若有交互需要则走 Client Component」 通过这种原则，相信在一定程度上能给减轻开发者的心智负担。

### 应用场景：文档站
从上面我们可以知道 Server Component 在轻交互性的场景下能够发挥它的优势来，轻交互的场景一般我们能想到文档站。  
特点：  
- 极小的 Js bundle。
- 文件修改无需 Bundle

当然像文档站等偏向静态的页面更适合 SSR, SSG，但就像前面所说的它并不与其他的技术互斥，我们可以将其进行结合，更况且他不仅仅能应用于这样的静态场景。  

原文:  
[React Server Component: 混合式渲染](https://mp.weixin.qq.com/s/BIxGyA1cnmWjFNNpL5NLmw)
