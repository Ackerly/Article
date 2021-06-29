# WebAssembly入门
## 概念
WebAssembly是一种运行在现代网络浏览器中的新型代码，并且提供新的性能特性和效果。它设计的目的是为了诸如C、C++和Rust等低级源语言提供一个高效的编译目标。对于网络平台而言其意义--为客户端app提供一种网络平台以接近本地速度的方式运行多种语言编写的代码的方式；在此之前客户端app不可能做到。
## 特点
- 高效  
WebAssembly有一套完整的语义，实际上wasm是体积小且加载快的二进制格式，其目标就是充分发挥硬件能以以达到的原生执行效率。
- 安全  
WebAssembly运行在一个沙箱化的执行环境中，甚至可以在现有的JavaScript虚拟机中实现。在web环境中，WebAssembly将会严格遵守同源策略以及浏览器安全策略。
- 开放  
WebAssembly设计一个非常完整的文本格式用来调试、测试、实验，优化、学习、教学或者编写程序。可以以这种文本格式在web页面上查看wasm模块的源码。
- 标准  
WebAssembly在web中设计成无版本、特性可测试、向后姜蓉的。WebAssembly可以被JavaScript调用，进入JavaScript上下文，可以像Web API一样调用浏览器的功能。WebAssembly不仅可以运行在浏览器上，也可以运行在非web环境下。
## WebAssembly如何适应网络平台
网络平台可以分为两部分：
- 一个运行网络程序（Web app）代码，比如给你的程序提供能力的JavaScript的虚拟机
- 一系列网络程序能够调用从而控制网络浏览器/设备功能，并且能够让事物发生变化的网络（DOM、CSSOM、WebGL、IndexedDB、Web Audio APi等）。

JavaScript应用到3D游戏、虚拟现实、增强现实、计算机视觉、图像/视频编辑以及大量的要求原生性能的其他领域时会有性能问题。而且下载、解析以及编译巨大的JavaScript应用程序的成本是过高的。WebAssembly不同于JavaScript的语言，但它不是用来取代JavaScript的，它被设计为何JavaScript一起协同工作，从而使网络开发者能够利用两种语言优势。
- JavaScript是一门高级语言。它使动态类型，不需要编译环节以及一个巨大的能够提供强大框架、库和其他工具的生态系统。
- WebAssembly是一门低级的类汇编语言。有一种紧凑的二进制格式，能够以接近原生性能的速度运行，并且为诸如C++和Rust等拥有低级的内存模型语言提供一个编译目标以便他们能够再网络上运行（WebAssembly有一个在将来支持使用垃圾回收内存模型的语言的高级目标）  

JWebAssembly的avaScript API使用能够被正常调用的JavaScript函数封装了导出的WebAssembly代码，并且WebAssembly的模块在很多方面都和ES2015的模块时等价的。

## WebAssembly关键概念
- 模块  
表示一个已经被浏览器编译为可执行机器码的WebAssembly二进制代码。一个模块时无状态的，并且像一个二进制大对象（Blob）一样能够被缓存到indexedDB中或者在windows和workers之间进行共享（通过postMessage）。
- 内存  
ArrayBuffer，大小可变。本质是连续的字节数组，WebAssembly的低级内存取指令可以对它进行读写操作。
- 表格    
带类型数组，大小可变。表格中的项存储了**不能作为原始字节存储在内存里的对象**的引用（为了安全和可移植性的原因）
- 实例  
一个模块及其运行使用的所有状态，包括内存、表格和一系列导入值。一个实例就像一个已经被加载到一个拥有一组特定的全局变量的ES2015模块。

通过吧JavaScript函数导入到WebAssembly实例中，任意JavaScript函数都能被WebAssembly代码同步调用。JavaScript能够完全控制WebAssembly代码如何下载、编译运行。所以可以把WebAssembly想象为一个高效生成高性能函数的JavaScript特性。
## 如何在APP中使用WebAssembly
- 使用Emscripten移植一个C/C++应用程序
- 直接带汇编层，编写或生成WebAssembly代码
- 编写Rust程序，将WebAssembly作为它的输出
### 从C/C++移植
创建WASM代码的众多选项中有两个是在线WASM汇编程序或Emscripten。在线WASM汇编程序有：
- WasmFiddle
- WasmFiddle++
- WasmExplorer

Emscripten工具能够将一段C/C++代码，编译出：
- 一个.wasm模块
- 用来加载和运行该模块的JavaScript”胶水“代码
- 一个用来展示代码运行结果的HTML文档
工作流程：
1. Emscripten首先把C/C++提供给clang+LLVM——一个成熟的开源C/C++编译器工具链，比如，在OSX上是XCode的一部分。
2. Emscripten将clang+LLVM编译的结果转换为一个.wasm二进制文件。
3. WebAssembly当前不能直接的存取DOM；它只能调用JavaScript，并且只能传入整形和浮点型的原始数据类型作为参数。为了使用任何Web API，WebAssembly需要调用到JavaScript，然后由JavaScript调用Web API。

参考:  
[WebAssembly概念](https://developer.mozilla.org/zh-CN/docs/WebAssembly/Concepts)
[WebAssembly 中文网](http://webassembly.org.cn/)
