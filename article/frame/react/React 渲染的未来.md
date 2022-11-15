# React 渲染的未来
重要术语：
- Time To First Byte (TTFB) ：发出页面请求到接收到应答数据第一个字节所花费的时间；
- First Paint (FP) ：第一个像素对用户可见的时间。
- First Contentful Paint (FCP)：第一条内容可见所需的时间。
- Largest Contentful Paint (LCP) ：加载页面主要内容所需的时间。
- Time To Interactive (TTI)：页面变为交互并可靠响应用户事件的时间。

## 当前的渲染模式
在 React 中使用的最常见的模式就是客户端渲染和服务端渲染，以及由 Next.js 等框架提供的一些高级形式的服务端渲染，例如静态站点生成（SSG）、增量静态生成（ISR）。  

**客户端渲染（CSR）**  
在 Next.js 和 Remix 等元框架出现之前，客户端渲染（主要使用 create-react-app 或其他类似的脚手架）是构建 React 应用程序的默认方式。  
使用客户端渲染，服务端只需要为包含必要 <script> 和 <link> 标签的页面提供基本 HTML。一旦相关的 JavaScript 下载到浏览器。React 渲染树并生成所有 DOM 节点。路由和数据获取的所有逻辑也由客户端 JavaScript 处理。  
CSR 应用接收到应答数据第一个字节很快（TTFB），因为它们主要依赖于静态资源。但是，在下载相关 JavaScript 之前，用户必须盯着空白屏幕。在那之后，由于大多数应用都需要从 API 获取数据并向用户显示相关数据，这导致加载页面主要内容所需的时间（LCP）很长。  
CSR 的优点  
- 由于客户端渲染架构包含静态文件，因此可以非常轻松地通过 CDN 提供服务
- 所有渲染都是在客户端完成的，因此 CSR 允许我们在不刷新整个页面的情况下进行导航，从而提供良好的用户体验
- TTFB 时间很快，因此浏览器可以立即开始加载字体、CSS 和 JavaScript

CSR 的缺点  
- 由于所有内容都在客户端渲染，因此性能受到很大影响，因为用户首先需要下载并处理它才能看到页面上的内容。
- 客户端渲染应用通常会在组件挂载时获取所需的数据，这会导致糟糕的用户体验，因为在初始页面加载时会遇到很多 loaders。此外，如果子组件需要获取数据，情况可能会变得更糟，这样它们的父组件获取完所有数据后才会渲染它们，这可能会导致大量 loaders 和糟糕的 Network Waterfall。
- SEO 是客户端渲染应用的一个问题，因为网络爬虫可以轻松读取服务端渲染的 HTML，但它们可能不会等待下载完所有 JavaScript 包，执行它们并等待客户端数据获取瀑布流完成 ，这可能会导致不正确的索引。

**服务端渲染（SSR）**  
在 React 中服务端渲染的工作方式如下：  
- 通过 renderToString 获取相关数据并在服务端为页面运行客户端 JavaScript，这为我们提供了显示页面所需的所有 HTML
- 将此 HTML 提供给客户端，从而实现快速的 First Contentful Paint
- 这时还没有完成， 我们仍然需要下载并执行客户端 JavaScript 以将 JavaScript 逻辑连接到服务端生成的 HTML 以使页面具有交互性（这个过程就是 “注水”）

使用 SSR，可以获得良好的 FCP 和 LCP，但 TTFB 会受到影响，因为必须在服务端获取数据，然后将其转换为 HTML 字符串。  
这和 Next.js 的 SSG/ISR 有啥区别呢？它们也必须经历上面的过程。唯一的区别是，它们不会受到 TTFB 时间较长的影响。因为 HTML 要么是在构建时生成的，要么是在请求传入时以增量方式生成和缓存的。  
但是，SSG/ISR 更适合公共页面。对于根据用户登录状态或浏览器上存储的其他 cookie 更改的页面，必须使用 SSR。  

SSR 的优点  
- 与 CSR 不同，SEO 要好得多，因为所有 HTML 都是从服务端预先生成的，网络爬虫可以毫无问题地爬取它
- FCP 和 LCP 非常快。因此，用户可以很快看到内容，而不是像 CSR 应用那样查看空白屏幕。

SSR 的缺点  
- 由于我们在每次请求时首先在服务端渲染页面，并且必须等待页面的数据需求，这可能会导致 TTFB 速度变慢。这可能是由多种原因导致的，包括未优化的服务端代码或者许多并发的服务端请求。不过，使用像 Next.js 这样的框架可以提前生成页面并使用 SSG（静态站点生成）和 ISR（增量静态站点生成）等技术将它们缓存在服务端，从而在一定程度上解决了这个问题。
- 即使初始加载速度很快，用户仍然需要等待下载页面的所有 JavaScript 并对其进行处理，以便页面可以重新注水并变得可交互

## 新的渲染模式
- 在 CSR 应用中，用户必须下载所有必要的 JavaScript 并执行它以查看 / 与页面交互
- 在 SSR 应用中，我们通过在服务端生成 HTML 来解决了其中的一些问题。然而这并不是最优的，因为首先我们必须等待服务端获取所有数据并生成 HTML。然后客户端必须下载整个页面的 JavaScript。最后，我们必须执行 JavaScript 以连接服务端生成的 HTML 和 JavaScript 逻辑，以便页面可以交互。所以主要问题是我们必须要等待每一步完成，然后才能开始下一步。

**流式 SSR**  
浏览器可以通过 HTTP 流接收 HTML。流式传输允许 Web 服务端通过单个 HTTP 连接将数据发送到客户端，该连接可以无限期保持打开状态。因此可以通过网络以多个块的形式在浏览器上加载数据，这些数据在渲染时按顺序加载。  

**React 18 之前的流式渲染**  
流式渲染并不是 React 18 中全新的东西。事实上，它从 React 16 开始就存在了。React 16 有一个名为 renderToNodeStream 的方法，与 renderToString 不同，它将前端渲染为浏览器的 HTTP 流。  
这允许在渲染它的同时以块的形式发送 HTML，从而为用户提供更快的 TTFB 和 LCP，因为初始 HTML 更快地到达浏览器。  

**React 18 中的流式 SSR**  
React 18 弃用了 renderToNodeStream API，取而代之的是一个名为 renderToPipeableStream 的新 API，它通过 Suspense 解锁了一些新功能，允许将应用分解为更小的独立单元，这些单元可以独立完成我们在 SSR 中看到的步骤。这是因为 Suspense 添加了两个主要功能：  
- 服务端流式渲染；
- 客户端选择性注水。

**服务端流式渲染**  
React 18 之前的 SSR 是一种全有或全无的方法。首先，需要获取页面所需的数据，并生成 HTML，然后将其发送到客户端。由于 HTTP 流，情况不再如此。  
在 React 18 中想要使用这种方式，可以包装可能需要较长时间才能加载且在 Suspense 中不需要立即显示在屏幕上的组件。  
假设 Comments API 很慢，所以我们将 Comments 组件包装在 Suspense 中：  
``` 
<Layout>
   <NavBar />
   <Sidebar />
   <RightPane>
     <Post />
     <Suspense fallback={<Spinner />}>
       <Comments />
     </Suspense>
   </RightPane>
 </Layout>
```
初始 HTML 中就不存在 Comments，返回的只有占位的 Spinner：  
``` 
<main>
   <nav>
     <!--NavBar -->
     <a href="/">Home</a>
    </nav>
   <aside>
     <!-- Sidebar -->
     <a href="/profile">Profile</a>
   </aside>
   <article>
     <!-- Post -->
     <p>Hello world</p>
   </article>
   <section id="comments-spinner">
     <!-- Spinner -->
     <img width=400 src="spinner.gif" alt="Loading..." />
   </section>
 </main>
```
当数据准备好用于服务端的 Comments 时，React 将发送最少的 HTML 到带有内联 <script> 标签的同一流中，以将 HTML 放在正确的位置：  
``` 
<div hidden id="comments">
   <!-- Comments -->
   <p>First comment</p>
   <p>Second comment</p>
 </div>
 <script>
   // 简化了实现
   document.getElementById('sections-spinner').replaceChildren(
     document.getElementById('comments')
   );
 </script>
```
这解决了第一个问题，因为现在不需要等待服务端获取所有数据，浏览器可以开始渲染应用的其余部分，即使某些部分尚未准备好。  

**客户端选择性注水**  
即使 HTML 被流式传输，页面也不会可交互的，除非页面的整个 JavaScript 被下载完。这就是选择性注水的用武之地。  
在客户端渲染期间避免页面上出现大型包的一种方法就是通过 React.lazy 进行代码拆分。它指定了应用的某个特定部分不需要同步加载，并且打包工具会将其拆分为单独的 <script> 标签  
React.lazy 的限制是它不适用于服务端渲染。但在 React 18 中，<Suspense> 除了允许流式传输 HTML 之外，它还可以为应用的其余部分注水  
现在 React.lazy 在服务端开箱即用。当你将 lazy 组件包裹在 <Suspense> 中时，不仅告诉 React 你希望它被流式传输，而且即使包裹在 <Suspense> 中的组件仍在被流式传输，也允许其余部分注水。这也解决了我们在传统服务端渲染中看到的第二个问题。在开始注水之前，不再需要等待所有 JavaScript 下载完毕。  
把 Comments 包含在 Suspense 中，并使用新的 Suspense 架构，来看看应用的生命周期：  
``` 
 <Layout>
   <NavBar />
   <Sidebar />
   <RightPane>
     <Post />
     <Suspense fallback={<Spinner />}>
       <Comments />
     </Suspense>
   </RightPane>
 </Layout>
```
对于 Suspense，很多连续发生的事情现在可以并行发生  
这不仅有助于我们在 HTML 被流式传输后更快地 TTFB，而且用户不必等待所有 JavaScript 被下载才能开始与应用交互。除此之外，它还有助于在页面开始流式传输时立即加载其他资源（CSS、JavaScript、字体等），有助于并行更多请求。  
如果有多个组件包裹在 Suspense 中并且还没有在客户端上注水，但是用户开始与其中一个交互，React 将优先考虑给该组件注水。  

**Server components (Alpha)**  
Server Components RFC旨在补充服务端渲染，允许拥有仅在服务端渲染且没有交互性的组件。  
它们的工作方式就是可以使用 .server.js/jsx/ts/tsx 扩展创建非交互式服务端组件，然后它们可以无缝集成并将 props 传递给客户端组件（使用 .client.js/jsx/ts/tsx 扩展），它可以处理页面的交互部分。以下是它提供的功能的：

_不影响客户端包_  
服务端组件仅在服务端渲染，不需要注水。它允许我们在服务端渲染静态内容，同时对客户端包大小没有影响。如果使用的是繁重的库并且没有交互性，这可能特别有用，并且它可以完全渲染在服务端，而不会影响客户端包。RFC 中的 Notes 预览就是一个很好的例子：  
``` 
// NoteWithMarkdown.js
 // 在 Server Components 之前

 import marked from 'marked'; // 35.9K (11.2K gzipped)
 import sanitizeHtml from 'sanitize-html'; // 206K (63.3K gzipped)

 function NoteWithMarkdown({text}) {
   const html = sanitizeHtml(marked(text));
   return (/* render */);
 }
```
``` 
// NoteWithMarkdown.server.js - Server Component === 包大小为0

 import marked from 'marked'; // 包大小为0
 import sanitizeHtml from 'sanitize-html'; // 包大小为0

 function NoteWithMarkdown({text}) {
   const html = sanitizeHtml(marked(text));
   return (/* render */);
 }
```

_服务端组件不具有交互性，但可以与客户端组件组合_ 
由于它们只在服务端渲染，它们只是接收 props 并渲染视图的 React 组件。因此，它们不能像常规客户端组件中那样拥有状态、effects 和事件处理程序之类的东西。  
尽管它们可以导入具有交互性的客户端组件，并且在客户端上渲染时注水，正如我们在普通 SSR 中看到的那样。客户端组件与服务端组件类似，使用 .client.jsx 或 .client.tsx 后缀定义。  
这种可组合性使开发人员在页面上节省大量的包大小，例如具有大部分静态内容和很少交互元素的详情页。例如：  
``` 
// Post.server.js

 import { parseISO, format } from 'date-fns';
 import marked from 'marked';
 import sanitizeHtml from 'sanitize-html';

 import Comments from '../Comments.server.jsx'
 // 导入客户端组件
 import AddComment from '../AddComment.client.jsx';

 function Post({ content, created_at, title, slug }) {
   const html = sanitizeHtml(marked(content));
   const formattedDate = format(parseISO(created_at), 'dd/MM/yyyy')

   return (
     <main>
         <h1>{title}</h1>
         <span>Posted on {formattedDate}</span>
         {content}
         <AddComment slug={slug} />
         <Comments slug={slug} />
     </main>
   )
 }
```
``` 
// AddComment.client.js

 function AddComment({ hasUpvoted, postSlug }) {

   const [comment, setComment] = useState('');

   function handleCommentChange(event) {
     setComment(event.target.value);
   }

   function handleSubmit() {
     // ...
   }

   return (
     <form onSubmit={handleSubmit}>
       <textarea name="comment" onChange={handleCommentChange} value={comment}/>
       <button type="submit">
         Comment
       </button>
     </form>
   )
 }
```
- Post 服务端组件主要包含静态数据，包括文章标题、内容和发布日期
- 由于服务端组件不能有任何交互性，我们导入了一个名为 AddComment 的客户端组件，它允许用户添加评论

在服务端组件中导入的所有日期和 markdown 解析库都不会在客户端下载。我们在客户端下载的唯一 JavaScript 就是 AddComment 组件  

_服务端组件可以直接访问后端_  
它们仅在服务端渲染，因此可以使用它们直接从组件访问数据库和其他仅限后端的数据源，如下所示：  
``` 
// Post.server.js

 import { parseISO, format } from 'date-fns';
 import marked from 'marked';
 import sanitizeHtml from 'sanitize-html';

 import db from 'db.server';

 // 导入客户端组件
 import Upvote from '../Upvote.client.js';

 function Post({ slug }) {
   // 直接从数据库中读取数据
   const { content, created_at, title } = db.posts.get(slug);
   const html = sanitizeHtml(marked(content));
   const formattedDate = format(parseISO(created_at), 'dd/MM/yyyy');

   return (
     <main>
       <h1>{title}</h1>
       <span>Posted on {formattedDate}</span>
       {content}
       <AddComment slug={slug} />
       <Comments slug={slug} />
     </main>
   );
 }
```
在传统的服务端渲染中也可以实现这一点。例如，Next.js 可以直接在 getServerSideProps 和 getStaticProps 中访问服务端数据。没错，但区别在于，传统的 SSR 是一种全有或全无的方法，只能在顶级页面上完成，但服务端组件可以在每个组件的基础上执行此操作。  

_自动代码拆分_  
代码拆分是一个概念，它允许将应用分成更小的块，向客户端发送更少的代码。对应用进行代码拆分的最常见方式就是按路由进行拆分。这也是 Next.js 等框架默认拆分包的方式。  
除了自动代码拆分之外，React 还允许使用 React.lazy API 在运行时延迟加载不同的模块。这又是一个来自 RFC 的很好的例子，说明这可能特别有用：  
``` 
// PhotoRenderer.js
 // 在 Server Components 之前

 import React from 'react';
 const OldPhotoRenderer = React.lazy(() => import('./OldPhotoRenderer.js'));
 const NewPhotoRenderer = React.lazy(() => import('./NewPhotoRenderer.js'));

 function Photo(props) {
   if (FeatureFlags.useNewPhotoRenderer) {
     return <NewPhotoRenderer {...props} />;
   } else {
     return <OldPhotoRenderer {...props} />;
   }
 }
```
这种技术通过在运行时只动态导入需要的组件来提高性能，但它确实有一些问题。例如，这种方法会延迟应用开始加载代码的时间，从而抵消了加载更少代码的好处。  
正如我们之前在客户端组件如何与服务器组件组合中看到的那样，它们通过将所有客户端组件导入视为潜在的代码拆分点，并允许开发人员选择要在服务端更早渲染的内容，从而使客户端能够更早下载。下面是 RFC 中使用服务端组件的相同 PhotoRenderer 示例：  
``` 
// PhotoRenderer.server.js - Server Component

 import React from 'react';
 import OldPhotoRenderer from './OldPhotoRenderer.client.js';
 import NewPhotoRenderer from './NewPhotoRenderer.client.js';

 function Photo(props) {
   if (FeatureFlags.useNewPhotoRenderer) {
     return <NewPhotoRenderer {...props} />;
   } else {
     return <OldPhotoRenderer {...props} />;
   }
 }
```
服务端组件可以在保留客户端状态的同时重新加载：我们可以随时从客户端重新获取服务端树，以从服务端获取更新的状态，而不会破坏本地客户端状态、焦点甚至正在进行的动画。  
这是可能的，因为接收到的 UI 描述是数据而不是纯 HTML，这允许 React 将数据合并到现有组件中，从而使客户端状态不会被破坏。  

_服务端组件与 Suspense 集成_  
服务器组件可以通过 <Suspense> 逐步流式传输，正如在上面中看到的那样，这允许我们在等待页面剩余部分加载时创建加载状态并快速显示重要内容。  
这次 Sidebar 和 Post 是服务端组件，而 Navbar 和 Comments 是客户端组件。将 Post 包裹在 Suspense 中。  
``` 
<Layout>
   <NavBar />
   <SidebarServerComponent />
   <RightPane>
     <Suspense fallback={<Spinner />}>
       <PostServerComponent />
     </Suspense>
     <Suspense fallback={<Spinner />}>
       <Comments />
     </Suspense>
   </RightPane>
 </Layout>
```
它的 Network 图与使用 Suspense 的流式渲染非常相似，但 JavaScript 更少。因此，服务端组件甚至是解决我们一开始的问题的进一步措施，它不仅可以下载更少的 JavaScript，而且还显着改善了开发者体验。  
React 团队还在 RFC 常见问题解答中提到，他们在 Facebook 的单个页面上对少数用户进行了实验，产品代码大小减少了约 30%。  

_什么时候可以开始使用这些功能_  
服务端组件仍处于 alpha 阶段，而具有新 Suspense 架构的流式 SSR 所需的用于数据获取的 Suspense 还没有正式发布，将在 React 18 的小更新中发布


原文:  
[React 渲染的未来](https://mp.weixin.qq.com/s/d0Sh0tanTJ6x0jsXcA4PFQ)
