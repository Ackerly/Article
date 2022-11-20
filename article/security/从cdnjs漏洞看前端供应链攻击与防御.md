# 从cdnjs漏洞看前端供应链攻击与防御
**cdnjs**  
在写前端的时候，常常会碰到许多要使用第三方 library 的场合，例如说 jQuery 或者是 Bootstrap 之类的（前者在 npm 上每周 400 万次下载，后者 300 万次）。先撇开现在其实大多数都会用 webpack 自己打包这点不谈，在以往像这种需求，要嘛就是自己下载一份文件，要嘛就是找现成的 CDN 来载入,而 cdnjs 就是其中一个来源。  
除了 cdnjs，也有其他提供类似服务的网站，例如说在 jQuery 官网上可以看见他们自己的 code.jquery.com ，而 Bootstrap 则是使用了另一个叫做 jsDelivr 的服务。  
假设我现在做的网站需要用到 jQuery，我就要在页面中用 <script> 标签引入 jQuery 这个库，而这个来源可以是：  
- 自己的网站
- jsDelivr: https://cdn.jsdelivr.net/npm/jquery@3.6.0/dist/jquery.min.js
- cdnjs: https://cdnjs.cloudflare.com/ajax/libs/jquery/3.6.0/jquery.min.js
- jQuery 官方：https://code.jquery.com/jquery-3.6.0.min.js

假设我最后选择了 jQuery 官方提供的网址，就会写下这一段 HTML：  
``` 
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
```
为什么要选择 CDN，而不是选择下载下来，放在自己的网站上呢？可能有几个理由：  
- 懒惰，直接用别人的最快
- 预算考量，放别人网站可以节省自己网站流量花费跟负荷
- 速度考量

第三点速度考量值得特别说明一下，如果载入的函数库是来自于 CDN，下载的速度可能会比较快。  
较快的第一个理由是他们本来就是做 CDN 的，所以在不同国家可能都有节点。假设你主机放在美国，那若是放自己网站，台湾的使用者就要连到美国的服务器去抓这些 library，但如果是用 CDN 提供的网址，可能只要连到台湾的节点就好，省去一些延迟（latency）。  
第二个理由是如果大家都在用这个 CDN，那它被快取住的机率就提高了。例如说，假设 Facebook 也用了 cdnjs 来载入 jQuery 3.6.0 版，那如果我的网站也用了同样的服务载入了同个 library，对于访问过 Facebook 的浏览器来说，它就不需要再次下载文件，因为已经下载过，被快取住了。  

> 現在的浏览器对于快取多加了一个限制，也就是跨网站（更详细一点说是根据 eTLD+1 來判断）的快取将会分开。所以就算 Facebook 已经载入 jQuery 3.6.0，用户访问你的网站时还是需要再下载一次。

使用第三方 CDN 的优点，那缺点是什么呢？
1. 如果 CDN 挂了，你的网站可能会跟着一起挂，就算不是挂掉，连接缓慢也是一样。例如说我网站从 cdnjs 载入了 jQuery，可是 cdnjs 突然变得很慢，那我的网站也会变得很慢，一起被牵连。
2. 如果 CDN 被黑客入侵了，你引入的库被植入恶意代码，那你的网站就会跟着一起被入侵。

**解析 cdnjs 的 RCE 漏洞**  
2021 年 7 月 16 号，一名资安研究员 @ryotkak 在他的博客上发布了一篇文章，名为：Remote code execution in cdnjs of Cloudflare  
Remote code execution 简称为 RCE，这种漏洞可以让攻击者执行任意代码，是风险等级很高的漏洞。而作者发现了一个 cdnjs 的 RCE 漏洞，若是有心利用这个漏洞的话，可以控制整个 cdnjs 的服务。  
首先呢，Cloudflare 有把 cdnjs 相关的代码开源在 GitHub 上面，而其中有一个自动更新的功能引起了作者的注意。这个功能会自动去抓 npm 上打包好的 package 文件，格式是压缩档.tgz，解压缩之后把文件做一些处理，复制到合适的位置。  
作者知道在 Go 里面如果用 archive/tar 来解压缩的话可能会有漏洞，因为解压缩出来的文件没有经过处理，所以文件路径可以长得像是这样：../../../../../tmp/temp  
长成这样有什么问题呢？  
假设今天有一段代码是复制文件，然后做了类似底下的操作：  
- 用目的地 + 文件名拼凑出目标位置，建立新文件
- 读取原本文件，写入新文件

如果目的地是 /packages/test，文件名是 abc.js，那最后就会在 /packages/test/abc.js 产生新的文件  
若是目的地一样，文件名是 ../../../tmp/abc.js，就会在 /package/test/../../../tmp/abc.js 也就是 /tmp/abc.js 底下写入文件。  
透过这样的手法，可以写入文件到任何有权限的地方！而 cdnjs 的代码就有类似的漏洞，能够写入文件到任意位置。如果能利用这漏洞，去覆盖掉原本就会定时自动执行的文件的话，就可以达成 RCE 了。  
Git repo 的自动更新，有一段复制文件的代码，长这个样子：  
``` 
func MoveFile(sourcePath, destPath string) error {
     inputFile, err := os.Open(sourcePath)
     if err != nil {
         return fmt.Errorf("Couldn't open source file: %s", err)
     }
     outputFile, err := os.Create(destPath)
     if err != nil {
         inputFile.Close()
         return fmt.Errorf("Couldn't open dest file: %s", err)
     }
     defer outputFile.Close()
     _, err = io.Copy(outputFile, inputFile)
     inputFile.Close()
     if err != nil {
         return fmt.Errorf("Writing to output file failed: %s", err)
     }
     // The copy was successful, so now delete the original file
     err = os.Remove(sourcePath)
     if err != nil {
         return fmt.Errorf("Failed removing original file: %s", err)
     }
     return nil
 }
```
看起来没什么，就是复制文件而已，新建一个新文件，把旧文件的内容复制进去。  
但如果这个原始文件是个 symbolic link 的话，就不一样了。在继续往下之前，先简单介绍一下什么是 symbolic link。  
Symbolic link 的概念有点像是以前在 Windows 上看到的 “捷径”，这个捷径本身只是一个连结，连到真正的目标去。  
在类 Unix 系统里面可以用 ln -s 目标文件 捷径名称 去建立一个 symbolic link，这边直接举一个例子会更好懂。  
先建立一个文件，内容是 hello，位置是 /tmp/hello。接著在当前目录底下建立一个 symbolic link，指到刚刚建立好的 hello 档案：ln -s /tmp/hello link_file  
如果打印出 link_file 的内容，会出现 hello，因为其实就是在打印出 /tmp/hello 的内容。如果我对 link_file 写入数据，实际上也是对 /tmp/hello 写入。  
试试看用 Node.js 写一段复制文件的代码，看看会发生什么事：  
``` 
node -e 'require("fs").copyFileSync("link_file", "test.txt")'
```
执行完成之后，发现目录底下多了一个 test.txt 的文件，内容是 /tmp/hello 的 w 恩建内容。  
用程序在执行复制文件时，并不是 “复制一个 symbolic link”，而是 “复制指向的文件内容”。  
刚刚提到的 Go 复制文件的代码，如果有个文件是指向 /etc/passwd 的 symbolic link，复制完以后就会产生出一个内容是 /etc/passwd 的文件。  
可以在 Git 的文件里面加一个 symbolic link 名称叫做 test.js，让它指向 /etc/passwd，这样被 cdnjs 福建过后，就会产生一个 test.js 的文件，而且裡面是 /etc/passwd 的内容！  
如此一来，就得到了一个任意文件读取（Arbitrary File Read）的漏洞。  
作者一共找到两个漏洞，一个可以写文件一个可以读文件，写文件如果不小心覆盖重要文件会让系统挂掉，因此作者决定从读文件开始做 POC，自己建了一个 Git 仓库然后发布新版本，等 cdnjs 去自动更新，最后触发文件读取的漏洞，在 cdnjs 发布的 JS 上面就可以看到读到的文件内容。  
而作者读的文件是 /proc/self/environ（他本来是想读另一个 /proc/self/maps），这里面有着环境变数，而且有一把 GitHub 的 api key 也在里面，这把 key 对 cdnjs 底下的 repo 有写入权限，所以利用这把 key，可以直接去改 cdnjs 或是 cdnjs 网站的程序，进而控制整个服务。  
**身为前端工程师，该如何防御？**  
浏览器其实有提供一个功能：“如果文件被窜改过，就不要载入”，这样尽管 cdnjs 被入侵，jQuery 的文件被窜改，我的网站也不会载入新的 jQuery 文件，免于文件污染的攻击。  
在 cdnjs 上面，当你决定要用某一个 library 的时候，你可以选择要复制 URL 还是复制 script tag，若是选择后者，就会得到这样的内容：  
``` 
 <script
     src="https://cdnjs.cloudflare.com/ajax/libs/react/17.0.2/umd/react.production.min.js"
     integrity="sha512-TS4lzp3EVDrSXPofTEu9VDWDQb7veCZ5MOm42pzfoNEVqccXWvENKZfdm5lH2c/NcivgsTDw9jVbK+xeYfzezw=="
     crossorigin="anonymous"
     referrerpolicy="no-referrer">
 </script>
```
上面的另一个标签 integrity 才是防御的重点，这个属性会让浏览器帮你确认要载入的资源是否符合提供的 hash 值，如果不符合的话，就代表文件被窜改过，就不会载入资源。所以，就算 cdnjs 被入侵了，黑客替换掉了我原本使用的 react.js，浏览器也会因为 hash 值不合，不会载入被污染过的程序。  
利用这个网站 https://www.srihash.org/ 可以对远程资源进行 hash 值计算，并把相关值作为integrity属性值。  
不过这种方法只能防止 “已经引入的 script” 被窜改，如果碰巧在黑客窜改档案之后才复制 script，那就没有用了，因为那时候的文件已经是窜改过的文件了。  
如果要完全避免这个风险，就是不要用这些第三方提供的服务，把这些 library 放到自己家的 CDN 上面去，这样风险就从第三方的风险，变成了自己家服务的风险。除非自己家的服务被打下来，不然这些 library 应该不会出事。  
现在许多网站因为 library 都会经由 webpack 这类型的 bundler 重新切分，所以没有办法使用第三方的 library CDN，一定会放在自己家的网站上，也就排除了这类型的供应链攻击。  
要注意的是，仍然避免不了其他供应链攻击的风险。因为尽管没有用第三方的 library CDN，还是需要从别的地方下载这些库对吧？例如说 npm，你的库来源可能是这里，意思就是如果 npm 被入侵了，上面的文件被窜改，还是会影响到你的服务。这就是供应链攻击，不直接攻击你，而是从其他上游渗透进来。  
不过这类型的风险可以在 build time 的时候透过一些静态扫描的服务，看能不能抓出被窜改的文件或是恶意代码，或也有公司会在内部架一个 npm registry，不直接与外面的 npm 同步，确保使用到的库不会被窜改。  
**额外风险：CSP 的绕过**  
除了上面提到的供应链安全风险以外，其实使用第三方 JS 还有另一个潜在风险，就是 CSP (Content Security Policy) 的绕过。现在有许多网站都会设置 CSP，阻挡不信任的来源，例如说只允许某个 domain 的 JS 文件，或是不开放 inline event 跟 eval 等等。  
如果你的网站有用到 cdnjs 的脚本，你的 CSP 里面势必会有 https://cdnjs.cloudflare.com 这个网址。比起完整的路径，比较多人会倾向允许整个 domain 的东西，因为你可能用到多个 library，懒得一个一个新增上去。  
这时候若是网站有着 XSS 漏洞，一般情况下 CSP 应该会有防御作用，阻止这些不信任的代码的执行。但很遗憾地，CSP 中 https://cdnjs.cloudflare.com 的这个路径，让攻击者可以轻松绕过 CSP。  
原理就是 cdnjs 上除了你想要用的 library 之外，还有千千万万个不同的 library，而有些 library 本身提供的功能，让攻击者不需要执行 JS，也能执行任意程序。  
例如说 AngularJS，在旧版本中有着 Client-Side Template Injection 的漏洞，只需要 HTML 就可以执行代码，像是这类 “利用其他合法的 script 帮助你执行攻击代码” 的手法，叫做 script gadgets，想知道更多可以参考：security-research-pocs/script-gadgets  
假设现在的 CSP 只允许 https://cdnjs.cloudflare.com，找到这两个很棒的资源：  
- Bypassing path restriction on whitelisted CDNs to circumvent CSP protections - SECT CTF Web 400 writeup
- H5SC Minichallenge 3: "Sh＊t, it's CSP!"

只要利用 AngularJS + Prototype 这两个 library，就可以在符合 CSP（只引入 cdnjs 底下的脚本）的情况下进行 XSS  
想要避免这种 CSP bypass，就只能把 CSP 中 cdnjs 的路径写死，把整个脚本的 URL 写上去，而不是只写 domain。否则，这类型的 CSP 其实会帮助攻击者更容易突破 CSP 的限制，进而执行 XSS 攻击。

原文:  
[从cdnjs 的漏洞来看前端的供应链攻击与防御](https://mp.weixin.qq.com/s/bBdO1GSH3Zr5VASCbyhjxQ)
