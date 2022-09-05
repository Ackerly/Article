# JavaScript在线解压ZIP文件
## 浏览器解压方案
JSZip 是一个用于创建、读取和编辑 .zip 文件的 JavaScript 库，该库支持大多数浏览器，具体的兼容性如下：  

|  Opera   | Firefox  | Safari  | Chrome  | Internet Explorer  | Node.js  |
|  ----  | ----  | ----  | ----  | ----  | ----  |
|  Yes  | Yes  | Yes  | Yes  | Yes  | Yes  |
|  Tested with the latest version  | Tested with 3.0 / 3.6 / latest version  | Tested with the latest version  | Tested with the latest version  | Tested with IE 6 / 7 / 8 / 9 / 10  | Tested with node.js 0.10 / latest version  |

### 定义工具类
下载 ZIP 文件和解析 ZIP 文件的逻辑封装在 ExeJSZip 类中：  
``` 
class ExeJSZip {
  // 用于获取url地址对应的文件内容
  getBinaryContent(url, progressFn = () => {}) {
    return new Promise((resolve, reject) => {
      if (typeof url !== "string" || !/https?:/.test(url))
        reject(new Error("url 参数不合法"));
      JSZipUtils.getBinaryContent(url, { // JSZipUtils来自于jszip-utils这个库
        progress: progressFn,
        callback: (err, data) => {
          if (err) {
            reject(err);
          } else {
            resolve(data);
          }
        },
      });
    });
  }
  
  // 遍历Zip文件
  async iterateZipFile(data, iterationFn) {
    if (typeof iterationFn !== "function") {
      throw new Error("iterationFn 不是函数类型");
    }
    let zip;
    try {
      zip = await JSZip.loadAsync(data); // JSZip来自于jszip这个库
      zip.forEach(iterationFn);
      return zip;
    } catch (error) {
      throw new error();
    }
  }
}
```
### 在线解压 ZIP 文件
利用 ExeJSZip 类的实例，实现在线解压 ZIP 文件的功能：  
**html 代码**  
``` 
<p>
  <label>请输入ZIP文件的线上地址：</label>
  <input type="text" id="zipUrl" />
</p>
<button id="unzipBtn" onclick="unzipOnline()">在线解压</button>
<p id="status"></p>
<ul id="fileList"></ul>
```
**JS 代码**  
``` 
const zipUrlEle = document.querySelector("#zipUrl");
const statusEle = document.querySelector("#status");
const fileList = document.querySelector("#fileList");
const exeJSZip = new ExeJSZip();

// 执行在线解压操作
async function unzipOnline() {
  fileList.innerHTML = "";
  statusEle.innerText = "开始下载文件...";
  const data = await exeJSZip.getBinaryContent(
    zipUrlEle.value,
    handleProgress
  );
  let items = "";
  await exeJSZip.iterateZipFile(data, (relativePath, zipEntry) => {
    items += `<li class=${zipEntry.dir ? "caret" : "indent"}>
      ${zipEntry.name}</li>`;
  });
  statusEle.innerText = "ZIP文件解压成功";
  fileList.innerHTML = items;
}

// 处理下载进度
function handleProgress(progressData) {
  const { percent, loaded, total } = progressData;
  if (loaded === total) {
    statusEle.innerText = "文件已下载，努力解压中";
  }
}
```
能否预览解压后的文件呢？答案是可以的，因为 JSZip 这个库提供了 file API，通过这个 API 我们就可以读取指定文件中的内容。比如这样使用:  
``` 
zip.file("amount.txt").async("arraybuffer")
```
之后可以执行对应的操作来实现文件预览的功能。  
基于 JSZip 的方案并不是完美的，它存在一些限制。比如它不支持解压加密的 ZIP 文件，当解压较大的文件时，在 IE 10 以下的浏览器可能会出现闪退问题。

## 服务器解压方案
服务器解压方案就是允许用户通过文件 ID 或文件名进行在线解压,接下来将基于 koa 和 node-stream-zip 这两个库来介绍如何实现服务器在线解压 ZIP 文件的功能  
``` 
const path = require("path");
const Koa = require("koa");
const cors = require("@koa/cors");
const Router = require("@koa/router");
const StreamZip = require("node-stream-zip");

const app = new Koa();
const router = new Router();
const ZIP_HOME = path.join(__dirname, "zip"); // ZIP文件的根目录
const UnzipCaches = new Map(); // 保存已解压的文件信息

router.get("/", async (ctx) => {
  ctx.body = "服务端在线解压ZIP文件示例（阿宝哥）";
});

// 注册中间件
app.use(cors());
app.use(router.routes()).use(router.allowedMethods());

app.listen(3000, () => {
  console.log("app starting at port 3000");
});
```
在以上代码中，使用了 @koa/cors 和 @koa/router 两个中间件并创建了一个简单的 Koa 应用程序。  
** 根据文件名解压指定 ZIP 文件**
``` 
// app.js
router.get("/unzip/:name", async (ctx) => {
  const fileName = ctx.params.name;
  let filteredEntries;
  try {
    if (UnzipCaches.has(fileName)) { // 优先从缓存中获取
      filteredEntries = UnzipCaches.get(fileName);
    } else {
      const zip = new StreamZip.async({ file: path.join(ZIP_HOME, fileName) });
      const entries = await zip.entries();
      filteredEntries = Object.values(entries).map((entry) => {
        return {
          name: entry.name,
          size: entry.size,
          dir: entry.isDirectory,
        };
      });
      await zip.close();
      UnzipCaches.set(fileName, filteredEntries);
    }
    ctx.body = {
      status: "success",
      entries: filteredEntries,
    };
  } catch (error) {
    ctx.body = {
      status: "error",
      msg: `在线解压${fileName}文件失败`,
    };
  }
});
```
通过 ZIP_HOME 和 fileName 获得文件的最终路径，然后使用 StreamZip 对象来执行解压操作。为了避免重复执行解压操作，定义一个 UnzipCaches 缓存对象，用来保存已解压的文件信息。  
**在线解压 ZIP 文件**  
``` 
// html 代码
<p>
  <label>请输入ZIP文件名：</label>
  <input type="text" id="fileName" value="kl_161828427993677" />
</p>
<button id="unzipBtn" onclick="unzipOnline()">在线解压</button>
<p id="status"></p>
<ul id="fileList"></ul>
```
``` 
// JS 代码
const fileList = document.querySelector("#fileList");
const fileNameEle = document.querySelector("#fileName");

const request = axios.create({
  baseURL: "http://localhost:3000/",
  timeout: 10000,
});

async function unzipOnline() {
  const fileName = fileNameEle.value;
  if(!fileName) return;
  const response = await request.get(`unzip/${fileName}`);
  if (response.data && response.data.status === "success") {
    const entries = response.data.entries;
    let items = "";
    entries.forEach((zipEntry) => {
      items += `<li class=${zipEntry.dir ? "caret" : "indent"}>${
        zipEntry.name
      }</li>`;
    });
    fileList.innerHTML = items;
  }
}
```
利用 zip 对象提供的 entryData(entry: string | ZipEntry): Promise<Buffer> 方法就可以读取指定路径下文件的内容。  
**预览 ZIP 文件中指定路径的文件**  
``` 
// app.js
router.get("/unzip/:name/entry", async (ctx) => {
  const fileName = ctx.params.name; // ZIP压缩文件名
  const entryPath = ctx.query.path; // 文件的路径
  try {
    const zip = new StreamZip.async({ file: path.join(ZIP_HOME, fileName) });
    const entryData = await zip.entryData(entryPath);
    await zip.close();
    ctx.body = {
      status: "success",
      entryData: entryData,
    };
  } catch (error) {
    ctx.body = {
      status: "error",
      msg: `读取${fileName}中${entryPath}文件失败`,
    };
  }
});
```
通过 zip.entryData 方法来读取指定路径的文件内容，它返回的是一个 Buffer 对象。当前端接收到该数据时，还需要把接收到的 Buffer 对象转换为 ArrayBuffer 对象，对应的处理方式如下所示：  
``` 
function toArrayBuffer(buf) {
  let ab = new ArrayBuffer(buf.length);
  let view = new Uint8Array(ab);
  for (let i = 0; i < buf.length; ++i) {
    view[i] = buf[i];
  }
  return ab;
}
```
定义完 toArrayBuffer 函数之后，可以通过调用 app.js 定义的 API 来实现预览功能，具体的代码如下所示：
``` 
async function previewZipFile(path) {
  const fileName = fileNameEle.value; // 获取文件名
  const response = await request.get(
    `unzip/${fileName}/entry?path=${path}`
  );
  if (response.data && response.data.status === "success") {
    const { entryData } = response.data;
    const entryBuffer = toArrayBuffer(entryData.data);
    const blob = new Blob([entryBuffer]);
    // 使用URL.createObjectURL或blob.text()读取文件信息
  }
}
```
原文: 
[JavaScript 如何在线解压 ZIP 文件](https://juejin.cn/post/6971197120250396680?utm_source=gold_browser_extension)
