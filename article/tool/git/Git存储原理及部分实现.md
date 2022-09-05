# Git存储原理及部分实现
## GIT底层结构
目录结构  
``` 
├── COMMIT_EDITMSG  // 上一次提交的 msg
├── FETCH_HEAD // 远端的所有分支头指针 hash
├── HEAD // 当前头指针
├── ORIG_HEAD // 
├── config // 记录一些配置和远端映射
├── description // 仓库描述
├── hooks // commit lint规则 husky植入
│   ├── applypatch-msg
│   ├── applypatch-msg.sample
│   ├── commit-msg
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-applypatch
│   ├── post-checkout
│   ├── post-commit
│   ├── post-merge
│   ├── post-receive
│   ├── post-rewrite
│   ├── post-update
│   ├── post-update.sample
│   ├── pre-applypatch
│   ├── pre-applypatch.sample
│   ├── pre-auto-gc
│   ├── pre-commit
│   ├── pre-commit.sample
│   ├── pre-merge-commit
│   ├── pre-merge-commit.sample
│   ├── pre-push
│   ├── pre-push.sample
│   ├── pre-rebase
│   ├── pre-rebase.sample
│   ├── pre-receive
│   ├── pre-receive.sample
│   ├── prepare-commit-msg
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout
│   ├── push-to-checkout.sample
│   ├── sendemail-validate
│   ├── update
│   └── update.sample
├── index // 暂存区
├── info
│   ├── exclude
│   └── refs
├── logs // 顾名思义，记录我们的git log
│   ├── HEAD
│   └── refs
├── objects // git 存储的我们的文件
│   
├── lost-found //一些悬空的文件
│   ├── commit
│   └── other
├── packed-refs 打包好的指针头
└── refs // 所有的hash
    ├── heads
    ├── remotes
    └── tags
```
## GIT Hash
git中的对象一共有4种type，分别是 commit / tree / blob / tag。  
这四个type一定是要附带到我们sha1加密后的hash里面的，还有一些文本附加信息，整体的规则如下  
``` 
"{type} {content.length}\0{content}"
```
尝试下生成hash值，看看我们生成的，和git 生成的 是否一致。
``` 
echo -n "hello,world" | git hash-object --stdin

const crypto = require('crypto'),
const sha1 = crypto.createHash('sha1');
sha1.update("blob 11\0hello,world");
console.log(sha1.digest('hex'));
```
git 本质是一种类kv数据库的文件系统，通过sha1算法生成的hash作为key，对应到我们的git的几类对象，然后再去树状的寻址，最底层存储的是我们的文件内容  
衍生一下关于git使用sha1的目的以及前几年google碰撞sha1算法导致的 sha1算法不安全的问题，git使用sha1进行hash的目的，更多的是为了验证文件完整性 防损坏等目的  
## GIT 对象
**blob**  
这是最底层的对象，记录的是文件内容,上面计算hash的方式可以看出来，不管文件名怎么变化，我们所对应的那块内容没有改变，hash值就不会改变，找到的永远会是那个blob。  
这也是为什么 git是用来管理代码以及各种类型的文本的一种好方式  
在纯文本类型文件管理中，git只需要保存diff就行了，而如果我们代码中全是二进制文件，那简直是回溯噩梦，可能真实资源就两个pdf，一个word文件，但是版本太多，一个git仓库大小几个g也不是不可能。  
如果真的需要存储很多频繁变动的二进制文件，比如多媒体资源/ psd啥的，使用Git LFS，把我们的大文件变成文件指针存储在对象中，再去lfs拉取对应文件。  
**tree**  
blob对象是纯粹的内容，有些不对劲，我们内容需要索引，我怎么去找到它？  
随便点开一个objects下面的文件cat-file看看，可以看出来，整个对象的组织形式就是一棵多叉树。通过树级层级一层一层寻址，最后找到我们的内容块。  
**commit**  
个commit对应的信息其中只有几种  
- author与对应的时间点
- commit的时候我们输入的描述
- 这个commit所指向的tree
- 这个commit的parent 即父节点

git是以类似单向链表的形式将我们的一个个提交组织起来的，同时，同时一个节点至多有2个父节点。  
**tag**  
tag是对某个commit的描述，其实也是一种commit。
**小结**  
git的一个设计思路，git记录的是一个a → b过程的链表，通过链表，我们可以逐步回溯到a，在此之下呢，采用了一种多叉树形结构对我们的hash值进行分层记录，最底层，通过我们的hash值进行索引，对应到一个个压缩后的二进制objects。

## 实现
**识别命令参数**  
首先，让node环境能够读我们的一些命令，来干各种各样的事情，通过process的解析，我们能够获得输入的参数  
``` 
enum CommandEnum{
  Add= 'add',
  Init = 'init',
  ...
}
const chooseCommand = (command:CommandEnum) => {
  
  switch(command){
    case CommandEnum.Add:
      return add();
    case CommandEnum.Init:
      return init();
    ...
    default:
      break;
  }
  console.log("暂不支持此命令")
}

chooseCommand(process.argv[2] as CommandEnum);
```
**init**  
使用我们的命令，初始化一个git仓库  
``` 
const init = ()=>{
  
  fs.mkdirSync('.git');
  fs.mkdirSync('.git/refs')
  fs.mkdirSync('.git/objects')
  fs.writeFileSync('.git/HEAD','ref: refs/heads/master')
  fs.writeFileSync('.git/config',`
        [core]
    repositoryformatversion = 0
    filemode = true
    bare = false
    logallrefupdates = true
    ignorecase = true
    precomposeunicode = true
`);
  fs.writeFileSync('.git/description','');
}
```
**写入和读取**  
Git 中存在两个命令的概念，一个是底层命令(Plumbing) ，另一个就是我们日常会使用到的上层命令(Porcelain) ， 高层命令是基于底层命令的封装，让我们使用起来更为方便。  
引入一些npm包，定义一些结构体  
``` 
import fs from 'fs';
import zlib from 'zlib';
import crypto from 'crypto';

export enum GitObjectType{
  Commit = 'commit',
  Tree = 'tree',
  Blob = 'blob',
  Tag = 'tag'
}
```
实现一个简单的读取blob对象的方法，比较简陋，还不支持对content进行解析，将在后续完善。  
``` 
export const readObject = (sha1='f8eb512de72634ca12328d85f70b696414473914')=>{
  const data = fs.readFileSync(`.git/objects/${sha1.substring(0,2)}/${sha1.substring(2)}`);
  const a = zlib.inflateSync(data).toString('utf8');
  const typeIndex = a.indexOf(' ');
  const lengthIndex = a.indexOf(`\0`);

  const objType = a.substring(0,typeIndex);
  const length = a.substring(typeIndex +1,lengthIndex);
  const content = a.substring(lengthIndex+1);
  // console.log(a);
  return {objType,length,content};
}
```
有了读之后，还需要往里写  
``` 
export const createObject = (obj:GitObject)=>{
  const data = obj.serialize();
  const sha1 = crypto.createHash('sha1');
  sha1.update(data);
  const name = sha1.digest("hex");
  const zipData = zlib.deflateSync(data);
  console.log(name);
  const dirName = `.git/objects/${name.substring(0,2)}` 
  fs.existsSync(dirName) && fs.mkdirSync(dirName);
  fs.writeFileSync(`.git/objects/${name.substring(0,2)}/${name.substring(2)}`,zipData)
  return name;
}
```
cat-file命令和hash-object命令实际上实现起来就很简单了，只需要调用现有的方法就行了。
先是cat-file 对hash 名的一个寻址，同时解压缩对应的objects，支持四个参数，分别返回不同的结果。直接读对象就完事了。
``` 
export const catFile = ()=>{
  const type = process.argv[3];
  const sha1 = process.argv[4];
  const res = readObject(sha1);
  if(type ==='-t'){
    console.log(res.type);
  }
  if(type==='-s'){
    console.log(res.length);
  }
  if(type === '-e'){
    console.log(!!res?.type)
  }
  if(type === '-p'){
    console.log(res.content)
  }
}
```
接着是hash-object,这个也简单的实现下，就是返回对应路径的hash值就行了  
``` 
export const hashObject = ()=>{
  const path = process.argv[3];
  const data = fs.readFileSync(path);
  const sha1 = crypto.createHash('sha1');
  sha1.update(data);
  const name = sha1.digest("hex");
  console.log(name);
  }
```
基本结构已经搭起来了，需要的是commit 和tree将文件串联起来，调用我们的cat-file试试，现在应该对commit和blob的解析是正确的， 但是tree的content的解析似乎有些问题  
**内容解析完善**  
粗略的实现了一下读对象，能把内容块读出来了。接着我们来完善他，以便更好的服务于我们的四种对象,先改写下readObject。  
``` 
export const readObject = (sha1: string)=>{
  const data = fs.readFileSync(`.git/objects/${sha1.substring(0,2)}/${sha1.substring(2)}`);
  const buf = zlib.inflateSync(data)
  const a = buf.toString('utf8');
  const typeIndex = a.indexOf(' ');

  const lengthIndex = a.indexOf(`\0`);
  // console.log(a);
  const objType = a.substring(0,typeIndex);
  // 去掉校验， 其实这里需要记录长度和真实长度对比是否有错
  // const length = a.substring(typeIndex +1,lengthIndex);
  let obj;
  if(objType===GitObjectType.Blob){
    obj = new GitBlob(a.substring(lengthIndex+1));
  }
  if(objType===GitObjectType.Commit){
    obj = new GitCommit(a.substring(lengthIndex+1))
  }
  if(objType===GitObjectType.Tree){
    obj = new GitTree(buf.slice(lengthIndex+1))
  }
  return obj;
}
```
Commit对象实现起来稍微复杂一点，我们需要解析commit对象中的一些键值对，将他们都记住，同时把commit内容单独存起来。通过一个map将commit对象存储。  
Commit object  
``` 
class GitCommit extends GitObject{
  type = GitObjectType.Commit;
  data = '';
  length = 0;
  content:any;
  map;
  constructor(data:string){
  super();
    if(data){
      this.data=data;
      this.length = data.length;
    }
  }
  serialize = ()=>{
    return `${this.type} ${this.length}\0${this.data}`;
  }
  deserialize= ()=>{
    console.log(this.recursiveParse(this.data))
    return this.data;
  }
  recursiveParse = (data:string,map?:any):any=>{
    if(!map){
        map = new Map();
    }
    const space = data.indexOf(' ');

    const nl = data.indexOf(`\n`);
    console.log(space,nl);
    if(space<0 || nl<space){
       map.set("content",data);
        return map;
    }
    const key = data.substring(0,space);

    let end =0;
    while(true){
        end = data.indexOf(`\n`,end+1)
        if(data[end+1] !== ' ') break;
    }
    const value = data.substring(space+1,end);
    if(key ==='parent'){
        map.has('parent') ? map.set(key+'1',value) : map.set(key,value);
    } else {
        map.set(key,value)
    }

    const restData = data.substring(end+1);
    return this.recursiveParse(restData,map);
  }
}
```
**Tree Object解析**  
Tree Object 相对来说是解析起来最为复杂的一个对象，它不像前两个一样，能够通过直接toString就能拿到正常的文本，直接去解析就行了。Tree Object本身其实就是一个二进制对象  
``` 
class GitTree extends GitObject{
  type = GitObjectType.Commit;
  data:Buffer = Buffer.from('');
  length = 0;
  constructor(data:Buffer){
  super();
    if(data){
      this.data=data;
      this.length = data.length;
      
    }
  }
  serialize = ()=>{
    return `${this.type} ${this.length}\0${this.data}`;
  }
  parseTreeOneLine = (data:Buffer,start:number)=>{
    const x = data.indexOf(' ',start);
    if(x<0){
      return start+21;
    }
    const mode = data.slice(start,x).toString('ascii');
    // const type = 
    const y = data.indexOf(`\x00`,x)
    if(y<0){return x+21};
    const path = data.slice(x+1,y).toString('ascii');
    const sha1 = data.slice(y+1,y+21).toString('hex');
    console.log(mode,path,sha1);
    return y+21;
  }
  deserialize= ()=>{
    const buffer = this.data;
    let pos = 0;
    let max = buffer.length;
   
    while(pos<max){
      pos = this.parseTreeOneLine(buffer,pos);
    }
    return this.data;
  }
}
```
分支和ref：  
分支名和ref其实也是键值对，分支名作为文件名存储在ref目录下，文件内容则是一串sha1值，这串sha1值来自于commit 头结点的hash值，可以通过这个commit 对象回溯到当时的场景  
log与reflog  
单纯的文本文件，记录一些commit对象以及时间点等

原文: 
[Git存储原理及部分实现](https://mp.weixin.qq.com/s/j4DBwQlVxfixgPdpfW9dQA)
