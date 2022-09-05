# 使用 React Testing Library 的 15 个常见错误
**没有使用 Testing Library 的 ESLint 插件**  
如果你想避免这些常见的错误，那么官方的 ESLint 插件可以给你带来很多帮助：  
- eslint-plugin-testing-library
- eslint-plugin-jest-dom

如果已经在用 create-react-library，那 eslint-plugin-testing-library 已经包含包在依赖中了  
建议：最好把这两个 ESLint 插件都装上。  
**还在用 Wrapper 作为 render 返回值的变量名**  
``` 
// bad
 const wrapper = render(<Example prop="1" />)
 wrapper.rerender(<Example prop="2" />)

 // good
 const {rerender} = render(<Example prop="1" />)
 rerender(<Example prop="2" />)
```
Wrapper 是以前 Enzyme 的过时用法，现在已经不需要它了。而且 render 的返回值里也并没有 Wraper 任何东西，它只是一些工具 API 的集合而已。所以，一般情况下可以不需要它了。  
建议：直接使用从 render 返回值解构出来的东西，或者将返回值命名为 view。  
**手动使用 cleanup**  
``` 
 // bad
 import {render, screen, cleanup} from '@testing-library/react'

 afterEach(cleanup)

 // good
 import {render, screen} from '@testing-library/react'
```
现在 cleanup 都是自动调用的，所以已经不再需要再考虑它了。  
建议：别手动调 cleanup  
**不用 screen**  
``` 
// bad
 const {getByRole} = render(<Example />)
 const errorMessageNode = getByRole('alert')

 // good
 render(<Example />)
 const errorMessageNode = screen.getByRole('alert')
```
screen 是在 DOM Testing Library v6.11.0 引入的 （就就是说，你可以在 @testing-library/react@>=9 这些版本中使用它）。直接在 render 引入的时候一并引入就可以了：  
``` 
import {render, screen} from '@testing-library/react'
```
使用 screen 的好处是：在添加 / 删除 DOM Query 时，不需要实时地解构 render 的返回值来获取内容。输入 screen，你的编辑器就能自动补全它里面的 API 了。  
除非一种情况：你在配置 container 或者 baseElement。  
也可以直接调 screen.debug 而不是 debug  
建议：用 screen 来做 Querying 和 Debugging  
**使用错误的断言 API**  
``` 
const button = screen.getByRole('button', {name: /disabled button/i})

 // bad
 expect(button.disabled).toBe(true)
 // error message:
 //  expect(received).toBe(expected) // Object.is equality
 //
 //  Expected: true
 //  Received: false

 // good
 expect(button).toBeDisabled()
 // error message:
 //   Received element is not disabled:
 //     <button />
```
上面的 toBeDisabled 来自 jest-dom 这个库。强烈建议使用 jest-dom，因为能获得更好的错误信息。  
建议：用 @testing-library/jest-dom 这个库  
**将不必要的操作放在 act 里**  
``` 
// bad
 act(() => {
   render(<Example />)
 })

 const input = screen.getByRole('textbox', {name: /choose a fruit/i})
 act(() => {
   fireEvent.keyDown(input, {key: 'ArrowDown'})
 })

 // good
 render(<Example />)
 const input = screen.getByRole('textbox', {name: /choose a fruit/i})
 fireEvent.keyDown(input, {key: 'ArrowDown'})
```
不少人像上面那样把一些操作放在 act 里，因为他们一看到 "act" 的 Warning，就把操作放在 act 里面，以此去掉 Warning。但他们不知道的是 render 和 fireEvent 已经包裹在 act 里了！所以这样么其实没用。  
如果看到这些 act 的 Warning，不是要让你无脑地干掉它们，是在告诉你：你的测试有问题了。  
建议：去了解什么时候应该用 act，别把啥东西都往 act 里放  
**使用错误的 Query**  
``` 
// bad
 // 假设你有这样的 DOM：
 // <label>Username</label><input data-testid="username" />
 screen.getByTestId('username')

 // good
 // 改成通过关联 label 以及设置 type 来访问 DOM
 // <label for="username">Username</label><input id="username" type="text" />
 screen.getByRole('textbox', {name: /username/i})
```
_使用 container 来查询元素_  
```` 
// bad
 const {container} = render(<Example />)
 const button = container.querySelector('.btn-primary')
 expect(button).toHaveTextContent(/click me/i)

 // good
 render(<Example />)
 screen.getByRole('button', {name: /click me/i})
````
实际上我们更希望用户能直接和 UI 进行交互，然而，如果你用 querySelector 这些来做查询的话，不仅我们不能模仿用户的 UI 交互行为，测试代码也会变得很难读，而且容易崩。  
_没有用文本来做查询_  
``` 
// bad
 screen.getByTestId('submit-button')

 // good
 screen.getByRole('button', {name: /submit/i})
```
如果不用真实的文本来查询，那你要做很多额外的工作，因为你要确保你的地区语言的翻译转换是正确的。这里肯定有多人会吐槽说：要是别人改了文本的内容，你的测试不就崩了么？首先，如果有人将 “UserName” 更改为 “Email”，这是我绝对想知道的变更（因为我需要更改我的实现了）。而且，就算有人因为改了个名搞崩了测试，修复测试也用不了多长时间，马上就能修好了。  
总的来说，修复的成本是很低的，而好处则是可以增加你对翻译正确性信心，而且写出来的测试也是容易阅读和修改的。  
**多数情况下没有使用 ByRole**  
name 选项可以让你通过元素的 "Accessible Name" 查询元素，这也是 Screen Reader 会对每个元素读取的内容。好处是：即使元素的文本内容被其它不同元素分割了，它还是能够以此做查询。比如：  
``` 
// 假如现在我们有这样的 DOM：
 // <button><span>Hello</span> <span>World</span></button>

 screen.getByText(/hello world/i)
 // bad 报错:
 // Unable to find an element with the text: /hello world/i. This could be
 // because the text is broken up by multiple elements. In this case, you can
 // provide a function for your text matcher to make your matcher more flexible.

 screen.getByRole('button', {name: /hello world/i})
 // good 成功!
```
不使用 *ByRole 做查询的原因之一是他们不熟悉在元素上的隐式 Role。  
如果不能通过指定好的 Role 找到元素，它不仅会像 get* 以及 find* API 一样把整个 DOM 树都打印出来，而且还会把当前能访问的 Role 都打印出来  
``` 
// 假设我们有这样的 DOM
 // <button><span>Hello</span> <span>World</span></button>
 screen.getByRole('blah')
```
上面会报这样的错误：  
``` 
TestingLibraryElementError: Unable to find an accessible element with the role "blah"

 Here are the accessible roles:

   button:

   Name "Hello World":   <button />

   -------------------------------------------------- <body>
   <div>
     <button>
       <span>
         Hello       </span>

       <span>
         World       </span>
     </button>
   </div>
 </body>
```
建议：阅读并根据 “Which Query Should I Use" Guide” 里的推荐顺序来使用 Query  
**错误地添加可访问属性：aria-，role**  
``` 
// bad
 render(<button role="button">Click me</button>)

 // good
 render(<button>Click me</button>)

```
上面那样随意添加 / 修改可访问属性（Accessibility Attributes）不仅没有必要，而且还会把 Screen Reader 和用户搞懵。只有当无法满足当前的 HTML 语义时（比如你写了一个非原生的 UI 组件，同时也要让它 像 AutoComplete 一样可访问），你才应该使用可访问属性。假如这就是你现在要开发的东西，那可以用现有的第三库根据 WAI-ARIA 实践来实现可访问性。它们一般会有一些 很好的样例来参考。  
注意：如果要让 input 可以通过 role 来访问，你需要指定对应的 type 属性值！  
建议：避免错误地添加不必要的或不正确的可访问属性  
**没有使用 @testing-library/user-event**  
``` 
// bad
 fireEvent.change(input, {target: {value: 'hello world'}})

 // good
 userEvent.type(input, 'hello world')
```
@testing-library/user-event 是在 fireEvent 基础上实现的，但它提供了一些更接近用户交互的方法。上面这个例子中，fireEvent.change 其实只触发了 Input 的一个 Change 事件。但是 type 则可以对每个字符都会触发 keyDown、keyPress 和 keyUp 一系列事件。这能更接近用户的真实交互场景。好处是可以很好地和你当前那些没有监听 Change 事件的库一起使用。  
现在还在进行 @testing-library/user-event 这个库的开发，来保证它能像它承诺的那样：能够触发用户在执行特定操作时会触发的所有相同事件。不过，现在它还没完全做到这一点，这也是为什么它还没有合入 @testing-library/dom （可能在未来的某个时候会合入）。但是，我对它有足够的信心，建议你多关注和使用它，而不是 fireEvent。  
建议：尽可能地使用 @testing-library/user-event，而不是 fireEvent  
**没有用 query 来断言元素不存在**  
``` 
// bad
 expect(screen.queryByRole('alert')).toBeInTheDocument()

 // good
 expect(screen.getByRole('alert')).toBeInTheDocument()
 expect(screen.queryByRole('alert')).not.toBeInTheDocument()
```
把暴露 query* 相关的 API 出来的唯一原因是：可以在找不到元素的情况下不会抛出异常（返回 null）。唯一的好处是可以用来判断这个元素是否没有被渲染到页面上。这是很重要的，因为类似 get 和 find 相关的 API 在找不到元素时都会自动抛出异常 —— 这样你就可以看到渲染的内容以及为什么找不到元素的原因。然而，query* 只会返回 null，所以 toBeInTheDocument 在这里最好的用法就是：判断 null 不在 Document 上。  
建议：query* API 只用于断言当前元素不能被找到  
**用 waitFor 等待 find 的查询结果**  
``` 
// bad
 const submitButton = await waitFor(() =>
   screen.getByRole('button', {name: /submit/i}),
 )

 // good
 const submitButton = await screen.findByRole('button', {name: /submit/i})
```
上面两段代码几乎是等价的（find* 其实也是在内部用了 waitFor），但是第二种使用方法更清晰，而且抛出的错误信息会更友好。  
建议：当查询那些不能立马能访问到的元素时，使用 find*  
**给 waitFor 传空 callback**  
``` 
// bad
 await waitFor(() => {})
 expect(window.fetch).toHaveBeenCalledWith('foo')
 expect(window.fetch).toHaveBeenCalledTimes(1)

 // good
 await waitFor(() => expect(window.fetch).toHaveBeenCalledWith('foo'))
 expect(window.fetch).toHaveBeenCalledTimes(1)
```
waitFor 的目的是可以让你等一些指定的事情发生。如果传了空的 callback，可能它在今天还能 Work，因为你只是想在 Event Loop 等一个 Tick 就好了。但这样你也会留下一个脆弱的测试用例，一旦改了某些异步逻辑它很可能就崩了。  
建议：在 waitFor 里等待指定的断言，不要传空 callback  
**一个 waitFor callback 里有多个断言**  
``` 
// bad
 await waitFor(() => {
   expect(window.fetch).toHaveBeenCalledWith('foo')
   expect(window.fetch).toHaveBeenCalledTimes(1)
 })

 // good
 await waitFor(() => expect(window.fetch).toHaveBeenCalledWith('foo'))
 expect(window.fetch).toHaveBeenCalledTimes(1)

```
如果 window.fetch 调用了两次，那么 waitFor 就会失败，但是我们就得等到超时了才能看到具体报错。而如果 waitFor 里只有一个断言，我们则可以等待 UI 渲染到断言的同时，也可以在其中一个断言失败时更快地获得报错信息。  
建议：waitFor 的 callback 里只放一个断言  
**在 waitFor 中使用副作用**  
``` 
// bad
 await waitFor(() => {
   fireEvent.keyDown(input, {key: 'ArrowDown'})
   expect(screen.getAllByRole('listitem')).toHaveLength(3)
 })

 // good
 fireEvent.keyDown(input, {key: 'ArrowDown'})
 await waitFor(() => {
   expect(screen.getAllByRole('listitem')).toHaveLength(3)
 })
```
waitFor 适用的情况是：在执行的操作和断言之间存在不确定的时间量。因此，callback 可在不确定的时间和频率（在间隔以及 DOM 变化时调用）被调用（或者检查错误）。所以这也意味着你的副作用可能会被多次调用！  
同时，这也意味着你不能在 waitFor 里面使用快照断言（SnapShot Assertion）。如果你想要用快照断言，首先要等待某些断言走完了，然后再拍快照。  
建议：把副作用放在 waitFor 回调的外面，回调里只能有断言  
**用 get 来做断言**  
``` 
// bad
 screen.getByRole('alert', {name: /error/i})

 // good
 expect(screen.getByRole('alert', {name: /error/i})).toBeInTheDocument()
```
如果 get* API 找不到元素，它就会抛出异常，打印整个 DOM 树结构（语法高亮），在 Debug 的时候很有用。也因为这点，断言是永远不可能失败的（因为如果找不到元素，查询在断言之前抛出异常）。  
建议：如果你想断言某个东西是否存在，那么就做显式的断言操作  

原文: 
[使用 React Testing Library 的 15 个常见错误](https://mp.weixin.qq.com/s/gssYOb7xgSx2HsAeRGTgxA)
