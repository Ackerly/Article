# 如何利用 ChatGPT 提升日常编码效率
两个借助 ChatGPT 的能力提升编码效率的工具：  
- Cursor：一款基于 GPT-4 的智能代码编辑器
- Code GPT：一款基于 OpenAI API 的 VSCode 插件，可辅助编写代码

将从如下几个角度进行对比和分析：
- 环境配置
- 自动生成代码
- 解释代码
- 为代码添加注释
- 重构代码
- 寻找代码问题

## 环境配置
**Cursor 配置**
Cursor 是一款独立的编辑器，配置非常简单，因为内置了 ChatGPT 的能力，所以下载即用  
下载后即通过 Ctrl+k 调出对话框进行会话  
**Code GPT 配置**  
CodeGPT 是一款 VSCode 扩展，所以我们需要在 VSCode 扩展商店里找到并安装它  
CodeGPT 的 ChatGPT 并非内置的，所以需要我们自己的 OpenAI API 密钥，可以到 https://platform.openai.com/account/api-keys 创建一个密钥  
然后通过 cmd+shift+p 并搜索 “CodeGPT: Set API KEY” ，将自己的密钥设置上去  
打开 Settings，并搜索 CodeGPT 看看它的一些其他选项：  
- Max Tokens：在 API 处理提示之前，输入会被分解成 token，这个配置是是 API 应该取出并处理的最大 token 数量，因此它取决于你想要获取的响应长度。另外每个模型都有一个最大 token 数量限制。
- Model：CodeGPT 在处理查询时将使用的 OpenAI 模型，目前已经支持了最强大的 GPT-4
- Language：设置你希望与 API 交互的语言，类似于“解释”或“文档”的功能也将在选择的语言中进行
- Temperature：一个 0 到 1 之间的值，此设置可以确定所生成文本的随机性或“创意”水平。值越高，生成的输出随机性越高。

## 自动生成代码
**CodeGPT**  
在编辑器鼠标右键，点击 Ask CodeGPT,输入我们的问题,CodeGPT 给出的结果  

**Cursor**  
在 Cursor 中我们使用 command+k ，然后输入 prompts,Cursor 给出的结果  

**结果对比**  
- CodeGPT：创建了一个新的 .js 文件，但是回复展示的还是 MarkDown 语法，速度略慢（可能是没有采用流式输出的原因，直接将完整的结果贴了出来）
- Cursor：响应非常快，几乎输入的瞬间就开始给出结果（和 ChatGPT 一样的流式输出），而且直接在当前编辑器里输出，一些相关的说明使用注释进行展示，这一点体验更好一点

生成的函数在实现上基本一致，有一些细微的区别，比如在 CodeGPT 中使用的几个变量都是 let 声明的，实际上完全没有必要；而最后拼接数组  Cursor 使用的是 concat 方法，而 CodeGPT 使用的是更现代一点的扩展运算符，如果非要抠细节的话，扩展运算符的性能要略优于 concat 方法（因为 concat 会开辟新数组），不过在这个函数中只会合并一次，差别不大。  

## 解释代码
**CodeGPT**  
需要分析的代码，然后鼠标右键，点击 Explain CodeGPT  

**Cursor**  
选中需要分析的代码，按下 command+L，然后输入，'帮我解释这段代码'  

**结果对比**  
CodeGPT：
- 感觉仍然比较慢，并且还是开了一个新的 tab
- 按文本展示
- 展示的结果比较简单
- 使用内置的命令 Explain CodeGPT ，比较方便

Cursor：
- 在当前的编辑器弹出了一个新的窗口展示解释
- 按照 Markdown 语法进行展示
- 展示的结果明显要比 CodeGPT 更丰富一点，具体到了每个变量和辅助函数的实现
- 没有内置命令，需要手动输入指令

## 生成注释
**CodeGPT**  
CodeGPT 只有一个 Document CodeGPT 的功能可以为函数生成文档，没有内置注释能力，所以选中代码，然后鼠标右键，点击 Ask CodeGPT，输入请帮函数输入注释  

**Cursor**  
选中需要分析的代码，按下 command+L，然后输入，'帮这个函数生成注释'

**结果对比**  
- CodeGPT ：语言配置在生成注释上失效了，生成的注释是英文的（我们可以在改一下 prompts 为请帮我生成中文注释），但是注释生成的比较详细，函数、参数、返回值都有注释
- Cursor：语言上没有问题，注释生成的相对简洁一点，如果想要更详细的注释可以再继续引导

## 重构代码
测试重构代码的能力，使用一个判断字符串回文的函数：  
``` 
function isPalindrome(str) {
  const len = str.length;
  for (let i = 0; i < len / 2; i++) {
    if (str[i] !== str[len - i - 1]) {
      return false;
    }
  }
  return true;
}
```

**CodeGPT**  
需要分析的代码，然后鼠标右键，点击 Explain CodeGPT  

**Cursor**  
选中需要分析的代码，按下 command+L，然后输入，'帮我优化、重构这个函数'  

**结果对比**  
两者对代码的优化措施基本一致，但是如果你对结果不满意，可以进一步提示 Cursor 给出其他优化，这一点体验更好，而 CodeGPT 无法进一步引导。  

## 寻找代码 Bug
下面这段代码（含有数组越界的问题）来测试寻找代码 Bug 的能力：  
``` 
function calculateAverage(arr) {
    let sum = 0;
    for (let i = 0; i <= arr.length; i++) {
      sum += arr[i];
    }
    return sum / arr.length;
  }
```
**CodeGPT**  
选中需要分析的代码，然后鼠标右键，点击 Find Problems CodeGPT  
选中需要分析的代码，按下 command+L，然后输入，'帮我查找代码 Bug'  

**Cursor**  
Bug 比较简单，所以两者都很快给出了几乎相同的答案。  


## 总结
Cursor：
- 优点：响应更快、同一上下文可以方便的持续引导、展示的结果更友好，目前免费
- 缺点：脱离 VSCode 环境，这是硬伤，无法将强大的生态利用起来，所以注定无法去使用它来开发大型项目，只能打打辅助；另外没有内置的命令，每次只能人工输入命令

CodeGPT
- 优点：可以享受强大的 VSCode 生态，有很多内置的命令
- 缺点：响应稍微慢一点，展示结果不够友好

原文：  
[如何利用 ChatGPT 提升日常编码效率](https://mp.weixin.qq.com/s/--lvGQkum8DNkVbDDillQw)
