# JavaScript Containers
> 提出了一种设想：未来将出现新的类似容器的抽象来简化服务器，大多数 Web 服务可通过 JavaScript 容器而非 Linux 容器进行简化。在这个新兴的服务器抽象层中，JavaScript 取代了 Shell。

大多数服务器程序是 Linux 程序。它们包括一个文件系统、一些可执行文件，可能还有一些共享库，它们可能与 systemd 或 nsswitch 等系统软件交互。  
Docker 普及了 Linux 容器的使用；操作系统级别的虚拟化，为分发服务器软件提供了一种极好的机制。每个容器镜像都是一个无依赖的可立即运行的软件包。  
由于服务器软件通常依赖于许多系统资源和配置，因此在过去部署它一直充满挑战。Linux 容器解决了这个问题。  
在浏览器 JavaScript 中, 我们可以找到类似的封闭环境，尽管它处于更高的抽象级别中。Cloudflare 的 Zack Bloom 早在 2018 年就启发了我们去思考 JavaScript 本身是否可以提供一种新型的自包含服务器容器。  
我们越能去除不必要的抽象，就越能接近“网络就是计算机”的概念。Cloudflare Workers 本质上是 Cloudflare 网络中这一概念的实现。   
**通用脚本语言**  
脚本语言对许多服务器端问题有着很多的含义。大多数正在编写的代码不受计算力限制（compute bound），而是受生产力的限制：可以编写的速度和开发人员的金钱成本。脚本语言允许更快、更便宜地编写业务逻辑。脚本语言（Python、Ruby、Lua、Shell、Perl、Smalltalk、JavaScript）非常相似。除了语法和 API 存在差异，其余的就没有什么不同了。任何使用过 Rust 或 C 的人都了解脚本语言的感受。  
**Shell : 可执行文件 :: JavaScript : WebAssembly**  
服务器软件出现了一个新的更高级别的容器：JavaScript 沙箱本身。  
这个容器并不是为了解决 Linux 容器所针对的同样广泛的问题。它的出现只是我们追求简单性的结果。它最大限度地减少了编写 Web 服务端业务逻辑的样板工作。它与浏览器共享概念并减少程序员需要了解的概念。（例如：在编写 Web 服务时，很可能任何 systemd 配置都只是不必要的样板。）  
每个 Web 工程师都已经知道 JavaScript 浏览器 API。因为 JS 容器抽象是建立在相同的浏览器 API 之上的，所以工程师需要的经验总量减少了。Javascript 的普遍性降低了该容器在理解和使用上的复杂性。  
Shell 是用于调用 Unix 程序的解释型脚本语言。它可以编写条件，循环御酒，它有变量......但不幸的是它的能力是有限的，最终是满足不了复杂编程的需要。它真正的功能归于可执行文件。  
JavaScript 沙箱可以调用 Wasm，而不是像 shell 那样调用 Linux 可执行文件。如果你有一些繁重的计算工作，比如图像大小调整，使用 Wasm 而不是用 JS 编写它可能更有意义。就像你不会在 bash 中编写图像大小调整代码一样，你会借助于 imagemagick。  
脚本语言的未来是浏览器中的 JavaScript。Node.js 的根本错误在于不跟随ECMAScript API 的标准化,而是与之背道而驰，发明了太多东西。在 2010 年，我们还没有 ES 模块，但一旦标准化，它就应该被引入 Node.js 中。对于 promises、async/await、fetch、streams 等也可以这样说。像 CommonJS require、package.json、node_modules、NPM 这样过时的非标准，全局进程对象等要么应该被标准化并添加到浏览器中，或者应该被与 Web 对齐的替代品所取代。  
这个更高级别的容器还有待标准化。

参考:
[JavaScript Containers](https://mp.weixin.qq.com/s/fPcdVCqWvkPqAdVK7JHacg)
