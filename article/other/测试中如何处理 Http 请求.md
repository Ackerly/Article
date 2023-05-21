# 测试中如何处理 Http 请求
先来看下面这段测试代码有什么问题：  
``` 
// __tests__/checkout.js
 import * as React from 'react'
 import {render, screen} from '@testing-library/react'
 import userEvent from '@testing-library/user-event'
 import {client} from '~/utils/api-client'

 jest.mock('~/utils/api-client')

 test('clicking "confirm" submits payment', async () => {
   const shoppingCart = buildShoppingCart()
   render(<Checkout shoppingCart={shoppingCart} />)

   client.mockResolvedValueOnce(() => ({success: true}))

   userEvent.click(screen.getByRole('button', {name: /confirm/i}))

   expect(client).toHaveBeenCalledWith('checkout', {data: shoppingCart})
   expect(client).toHaveBeenCalledTimes(1)
   expect(await screen.findByText(/success/i)).toBeInTheDocument()
 })
```
第一个问题就是把 client 给 Mock 掉了，问问自己：你怎么知道 client 是一定会被正确调用的呢？当然，你可能会说：client 可以用别的单测来做保障呀。但你又怎么能保证 client 不会把返回值里的 body 改成 data 呢？哦，你是想说你用了 TypeScript 是吧？但由于我们把 client Mock 了，所以肯定不会完全保证 client 的功能正确性。你可能还会说：我还有 E2E 测试！  
如果我们在这里能真的调用一下 client 不是更能提高我们对 client 的信心么？好过一直猜来猜去嘛。  
不过，我们肯定也不是想真的调用 fetch 函数，所以我们会选择把 window.fetch 给 Mock 了：  
``` 
// __tests__/checkout.js
 import * as React from 'react'
 import {render, screen} from '@testing-library/react'
 import userEvent from '@testing-library/user-event'

 beforeAll(() => jest.spyOn(window, 'fetch'))

 // Jest 的 rsetMocks 设置为 true
 // 我们就不用担心要 cleanup 了
 // 这里假设你用了类似 `whatwg-fetch` 的库来做 fetch 的 Polyfill

 test('clicking "confirm" submits payment', async () => {
   const shoppingCart = buildShoppingCart()
   render(<Checkout shoppingCart={shoppingCart} />)

   window.fetch.mockResolvedValueOnce({
     ok: true,
     json: async () => ({success: true}),
   })

   userEvent.click(screen.getByRole('button', {name: /confirm/i}))

   expect(window.fetch).toHaveBeenCalledWith(
     '/checkout',
     expect.objectContaining({
       method: 'POST',
       body: JSON.stringify(shoppingCart),
     }),
   )
   expect(window.fetch).toHaveBeenCalledTimes(1)
   expect(await screen.findByText(/success/i)).toBeInTheDocument()
 })
```
这里的缺点在于：它不能测 headers 里是否会带有 Content-Type: application/json。没有这一步，我们也不能确定服务器是否真的能处理发出去的请求。还有一个问题，你怎么能确定用户鉴权的信息是不是真的也被带上呢？  
Mock 类似 fetch 函数的东西，最终会在所有地方把整个后端的逻辑都重新实现一遍。 这通常发生在多个测试之间，非常烦人。特别是在一些测试中，我们要假定后端要返回的内容的时候，就不得不在所有地方都要 Mock 一次。在这种情况下，就会给你和要做测试的东西设置了很多障碍。  
我们的测试策略就会变成这样：
- 把 client Mock 了（第一个例子），然后依赖一些 E2E 测试来保障 client 正确执行，以此给予我们心灵上一丢丢信心。但这也导致了一旦遇到后端的东西，就要在所有地方都要重新实现一遍后端逻辑  
- 把 window.fetch Mock 了（第二个例子）。这会好点，但这也会遇到第 1 点类似的问题

很长一段时间里我的解决方法是：声明一个假的 fetch 函数，把后端要 Mock 的内容都放里面。  
``` 
// 把它放在 Jest 的 setup 文件中，就会在所有测试文件前被引入了
 import * as users from './users'

 async function mockFetch(url, config) {
   switch (url) {
     case '/login': {
       const user = await users.login(JSON.parse(config.body))
       return {
         ok: true,
         status: 200,
         json: async () => ({user}),
       }
     }
     case '/checkout': {
       const isAuthorized = user.authorize(config.headers.Authorization)
       if (!isAuthorized) {
         return Promise.reject({
           ok: false,
           status: 401,
           json: async () => ({message: 'Not authorized'}),
         })
       }
       const shoppingCart = JSON.parse(config.body)
       // 可以在这里添加购物车的逻辑
       return {
         ok: true,
         status: 200,
         json: async () => ({success: true}),
       }
     }
     default: {
       throw new Error(`Unhandled request: ${url}`)
     }
   }
 }

 beforeAll(() => jest.spyOn(window, 'fetch'))
 beforeEach(() => window.fetch.mockImplementation(mockFetch))
```
测试就可以写成这样了：  
``` 
// __tests__/checkout.js
 import * as React from 'react'
 import {render, screen} from '@testing-library/react'
 import userEvent from '@testing-library/user-event'

 test('clicking "confirm" submits payment', async () => {
   const shoppingCart = buildShoppingCart()
   render(<Checkout shoppingCart={shoppingCart} />)

   userEvent.click(screen.getByRole('button', {name: /confirm/i}))

   expect(await screen.findByText(/success/i)).toBeInTheDocument()
 })
```
这个测试方法不需要你做多余的事，就是写 case。这里还可以给它再多加一个失败的 Case  
## msw
msw 全称 “Mock Service Worker”。 现在 Service Worker 还只是浏览器中的功能，不能在 Node 端使用。但是，msw 可以支持 Node 端所有测试场景。  
它的工作原理是这样的：创建一个 Mock Server 来拦截所有的请求，然后你就可以像是在真的 Server 里去处理请求。  
用 json 来初始化数据库，或者用 faker（现在别用了） 和 test-data-bot 来构造数据。然后用 Server Handler（类似 Express 的写法）和 Mock DB 交互并返回 Mock 数据。这就可以更容易和快速地写测试了（配置好 Handler 后）。  
msw 还有一个优势：可以将这些 “Server Handler” 用在前端本地开发上，适用于以下场景：  
- API 还没实现完
- API 崩了的时候
- 网速太慢或者没联网

做类似事情的 Mirage它不是用 Service Worker 在客户端实现的，所以你不能在开发者的 Network Tab 里看到 HTTP 请求，但是 msw 则可以。 两者对比可以看这里。  
**示例**  
现在来看看 msw 是如何 Mock Server 的：  
``` 
// server-handlers.js
 // 放在这里，不仅可以给测试用也能给前端本地使用
 import {rest} from 'msw' // msw 支持 GraphQL
 import * as users from './users'

 const handlers = [
   rest.get('/login', async (req, res, ctx) => {
     const user = await users.login(JSON.parse(req.body))
     return res(ctx.json({user}))
   }),
   rest.post('/checkout', async (req, res, ctx) => {
     const user = await users.login(JSON.parse(req.body))
     const isAuthorized = user.authorize(req.headers.Authorization)
     if (!isAuthorized) {
       return res(ctx.status(401), ctx.json({message: 'Not authorized'}))
     }
     const shoppingCart = JSON.parse(req.body)
     // do whatever other things you need to do with this shopping cart
     return res(ctx.json({success: true}))
   }),
 ]

 export {handlers}
 
 // test/server.js
 import {rest} from 'msw'
 import {setupServer} from 'msw/node'
 import {handlers} from './server-handlers'

 const server = setupServer(...handlers)
 export {server, rest}
 
 / test/setup-env.js
 // 加到 Jest 的 setup 文件上，可以在所有测试前执行
 import {server} from './server.js'

 beforeAll(() => server.listen())
 // 如果你要在特定的用例上使用特定的 Handler，这会在最后把它们重置掉
 // （对单测的隔离性很重要）
 afterEach(() => server.resetHandlers())
 afterAll(() => server.close())
```
测试就可以改成：  
``` 
// __tests__/checkout.js
 import * as React from 'react'
 import {render, screen} from '@testing-library/react'
 import userEvent from '@testing-library/user-event'

 test('clicking "confirm" submits payment', async () => {
   const shoppingCart = buildShoppingCart()
   render(<Checkout shoppingCart={shoppingCart} />)

   userEvent.click(screen.getByRole('button', {name: /confirm/i}))

   expect(await screen.findByText(/success/i)).toBeInTheDocument()
 })
```
比起 Mock fetch，使用这种方案的理由是：  
- 不用管 fetch 函数里的实现细节
- 当调用 fetch 时有报错，那么真实的 Server Handler 不会被调用，而且我的测试也会失败，可以避免提交有问题的代码
- 可以在前端本地开发时复用这些 Server Handler！

**Colocation 和 error/edge case testing**  
唯一值得担心的是：你可能会把所有 Server Handler 放在同一个地方，而依赖它们的测试文件又会被放在不同地方，这可能会导致文件放置不集中。  
只有那些对你测试很重要，很独特的东西才应该尽可能靠近测试文件。  
不需要在所有测试文件中都要重复 setup 一次，只需要 setup 独特的东西就可以了。 所以，最简单的方式就是：把常用的部分放在 Jest 的 setup 文件里。 不然你会有很多的干扰项，也很难对真正要测的东西进行隔离。  
对于自定义的场景，msw 可以在运行时允许你在测试用例中添加自定义的 Server Handler，也可以一键重置成你原来的 Handler，以此保留隔离性。 比如：  
``` 
// __tests__/checkout.js
 import * as React from 'react'
 import {server, rest} from 'test/server'
 import {render, screen} from '@testing-library/react'
 import userEvent from '@testing-library/user-event'

 // 啥也不需要改
 test('clicking "confirm" submits payment', async () => {
   const shoppingCart = buildShoppingCart()
   render(<Checkout shoppingCart={shoppingCart} />)

   userEvent.click(screen.getByRole('button', {name: /confirm/i}))

   expect(await screen.findByText(/success/i)).toBeInTheDocument()
 })

 // 边界情况、错误情况，需要添加自定义的 Handler
 // 注意 afterEach(() => server.resetHandlers())
 // 可以确保在最近移除新增特殊的 Handler
 test('shows server error if the request fails', async () => {
   const testErrorMessage = 'THIS IS A TEST FAILURE'
   server.use(
     rest.post('/checkout', async (req, res, ctx) => {
       return res(ctx.status(500), ctx.json({message: testErrorMessage}))
     }),
   )
   const shoppingCart = buildShoppingCart()
   render(<Checkout shoppingCart={shoppingCart} />)

   userEvent.click(screen.getByRole('button', {name: /confirm/i}))

   expect(await screen.findByRole('alert')).toHaveTextContent(testErrorMessage)
 })
```



原文:  
[测试中如何处理 Http 请求？](https://mp.weixin.qq.com/s/4p01lRklMCve36qXpyoGbg)
