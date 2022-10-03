# WebAssembly 打造定制 JS Runtime
lightdm-webkit2-greeter 这个 lightdm 插件支持在显示管理器中使用 web 技术去自定义登录界面，与操作系统的交互是通过 Runtime 中的一组 JS API来实现登录、关机、睡眠等功能。  
把 webkit 搬过来渲染系统界面，然后通过定制的 JS Runtime 与操作系统交互，相当于对浏览器本身进行了改造，关键的实现点是把系统调用封装成了Native函数，并在JS Runtime中进行绑定，以实现浏览器界面控制操作系统。  
这种方式和 Electron 的本质区别在于，无需让浏览器与另外一个进程通信，它直接拓展了 JS 的运行时环境，与 Node 的做法十分相像，不过这次我们越过中间商赚差价，自己实现 Runtime ，可以做的更小巧和定制化  

**思考**  
直接在浏览器上去定制 Runtime 这个想法确实很酷，但显然难度属于地狱级，这相当于我们直接去爆改 V8、JavaScriptCore 这种成熟稳定又复杂的JS引擎来是实现 JS API层面的嵌入和拓展，但 JS 引擎并不只是浏览器独有，真要改的话，可以找一个轻量、好改、好移植的。  
但是OS binding怎么办？总不能直接把浏览器里的JS引擎整个替换成这个不复杂，又好改，又好移植的吧？暂且先不做 OS binding，改做 Web binding，让Web Assembly来跑 Runtime，然后在 Runtime 里再跑JS，有点套娃了，但它依旧有一些应用的场景。  

## 实践
**起一个 JS 引擎**  
- 要方便移植，要好改，方便我们快速的定制
- Native 与 JS 的交互足够简单（包括数据类型的转换，通信的实现，事件循环等）
- 因为是编译到 WebAssembly 在 Web上跑，所以传输体积越小越好，同时运行时内存占用也最好不要太大。

选择了 Figma 曾经的方案 - Duktape  
- Duktape是一个嵌入式Javascript引擎，专注于可移植性和低空间占用。Duktape易于集成到C/C++项目中：将duktape.c, duktape.h，和duk_config.h添加到的构建项目中，然后使用Duktape API实现 C 代码与 Ecmascript 函数的双向调用。它最低打包编译体积低于 64kB ，可以运行在 192kB的内存上，同时支持诸多功能
- 
    - 完整的 ES5 支持
    - 支持垃圾回收
    - 字节码缓存
    - 支持调试功能

```
extern "C" char* runScript(char* script){
        duk_context *ctx = duk_create_heap_default();


        duk_eval_string(ctx, script);
        duk_pop(ctx);  /* pop eval result */

        duk_destroy_heap(ctx);

        return "ok";
}
```
**拓展一些 Runtime API**  
- IO 功能实现
    ```
    * Being an embeddable engine, Duktape doesn't provide I/O
     * bindings by default.  Here's a simple one argument print()
     * function.
     */
    static duk_ret_t native_print(duk_context *ctx) {
            duk_push_string(ctx, " ");
            duk_insert(ctx, 0);
            duk_join(ctx, duk_get_top(ctx) - 1);
            printf("%s\n", duk_safe_to_string(ctx, -1));
            return 0;
    }
    ```
- 绑定到 Runtime
  ```
    duk_push_c_function(ctx, native_print, DUK_VARARGS);
    duk_put_global_string(ctx, "print");
  ```

我们实现了一个基本的JS引擎，它可以完成 ES5 代码的解析和执行，在全局对象上注入了一个 print 方法，它是一个 Native的实现，通过引擎内部的堆栈与 JS 交互，最后 使用Duktape提供的注册方式暴露到 JS Runtime中  

## 编译成 WASM
这里编译器的实现选用 emscripten，用它直接生成相应的 WebAssembly 文件和相应的 JS 胶水代码  
- 把刚刚实现的 JS 执行函数暴露到 宿主环境中（另一个JS Runtime）
    ```
    int main() {
        EM_ASM("console.log('wasm js runtime is ready!')");
        EM_ASM("window.runScript = Module.cwrap('runScript', 'string', ['string'])");
        return 0;
    }
    ```
- 在编译的时候，指定导出函数
    ```
    CCOPTS += -s EXPORTED_FUNCTIONS=['_runScript','_main']
    ```
- 完整的Makefile 如下
    ```
    DUKTAPE_SOURCES = ./engine/duktape.c

    CC = emcc
    CCOPTS = -s DISABLE_EXCEPTION_CATCHING=0 -s ALLOW_MEMORY_GROWTH=1 -O3 --bind
    CCOPTS += -s EXPORTED_RUNTIME_METHODS=["cwrap"]
    CCOPTS += -s EXPORTED_FUNCTIONS=['_runScript','_main']
    CCOPTS += -I./engine  # for combined sources
    DEFINES =

    BUILD = wasm/index.html

    all: $(DUKTAPE_SOURCES) main.cpp
            ${CC} $(CFLAGS) $(CPPFLAGS) ${LDFLAGS} -o ${BUILD} ${DEFINES} ${CCOPTS} ${DUKTAPE_SOURCES} main.cpp ${CCLIBS}

    run:
            cd wasm && python3 -m http.server 8080
    ```
**简单测试**  
```
make
make run
```
看一下 WASM 体积，胶水代码+ WASM本体不 600KB 出头，基本在一张大图的范围内，可以接受  
借助这两个项目，至此我们完成了一整个 JS Runtime 定制的流程，目前看起来它完全是可用的：  
- 它足够小巧，随取随用
- 它与宿主 JS Runtime 完全隔离，足够安全
- WASM 实现相对来说在Web上是性能较好的，不会影响浏览器中JS线程

**应用场景**  
- JS 沙箱
- 打造插件系统
- 把WASM 产物一移植到 WASI 以实现真正的 OS Binding

原文:  
[使用 WebAssembly 打造定制 JS Runtime](https://mp.weixin.qq.com/s/jMFdha5FNqo2lPmyx10SIQ)