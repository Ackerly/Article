# VSCode IPC通信机制
## Electron 与 NW.js
### NW.js 内部架构
NW.js 是最早的 Node.js 桌面应用框架，在 NW.js 中，将 Node.js 和 Chromium 整合在一起使用，其中做了几件事情:  
1. Node.js 和 Chromium 都使用 V8 来处理执行的 JavaScript，因此在 NW.js 中它们使用相同的 V8 实例。
2. Node.js 和 Chromium 都使用事件循环编程模式，但它们用不同的软件库（Node.js 使用 libuv，Chromium 使用 MessageLoop/Message-Pump）。NW.js 通过使 Chromium 使用构建在 libuv 之上的定制版本的 MessagePump 来集成 Node.js 和 Chromium 的事件循环（如图2）。
3. 整合 Node.js 的上下文到 Chromium 中，使 Node.js 可用  
### Electron 内部架构
Electron 强调 Chromium 源代码和应用程序进行分离，因此并没有将 Node.js 和 Chromium 整合在一起。  
在 Electron 中，分为主进程(main process)和渲染器进程(renderer processes)：  
- 主进程：一个 Electron 应用中，有且只有一个主进程（package.json 的 main 脚本）
- 渲染进程：Electron 里的每个页面都有它自己的进程，叫作渲染进程。由于 Electron 使用了 Chromium 来展示 web 页面，所以 Chromium 的多进程架构也被使用到

在 Electron 中，可以通过以下方式来进行主进程和渲染器进程的通信：
1. 利用ipcMain和ipcRenderer模块进行 IPC 方式通信，它们是处理应用程序后端（ipcMain）和前端应用窗口（ipcRenderer）之间的进程间通信的事件触发。
2. 利用remote模块进行 RPC 方式通信。

> remote模块返回的每个对象（包括函数），表示主进程中的一个对象（称为远程对象或远程函数）。当调用远程对象的方法时，调用远程函数、或者使用远程构造函数 (函数) 创建新对象时，实际上是在发送同步进程消息

Electron 中从应用程序的后端部分到前端部分的任何状态共享（反之亦然），均通过ipcMain和ipcRenderer模块进行。这样，主进程和渲染器进程的 JavaScript 上下文将保持独立，但是可以在进程之间以显式方式传输数据。

## VSCode 的通信机制
### VSCode 多进程架构
VSCode 启动后主要有下面的几个进程：

- 主进程
- 渲染进程，多个，包括 Activitybar、Sidebar、Panel、Editor 等等
- 插件宿主进程
- Debug 进程
- Search 进程

在 VSCode 中，这些进程的通信方式同样包括 IPC 和 RPC 两种
### IPC 通信
主进程和渲染进程的通信基础还是 Electron 的webContents.send、ipcRender.send、ipcMain.on。
#### 频道
作为一个频道而言，它会有两个功能，一个是点播call，一个是收听，即listen。
``` 
/**
 * IChannel是对命令集合的抽象
 * call 总是返回一个至多带有单个返回值的 Promise
 */
export interface IChannel {
	call<T>(command: string, arg?: any, cancellationToken?: CancellationToken): Promise<T>;
	listen<T>(event: string, arg?: any): Event<T>;
}
```
#### 客户端与服务端
客户端和服务端的区分主要是：发起连接的一端为客户端，被连接的一端为服务端。在 VSCode 中，主进程是服务端，提供各种频道和服务供订阅；渲染进程是客户端，收听服务端提供的各种频道/服务，也可以给服务端发送一些消息（接入、订阅/收听、离开等）。  
不管是客户端和服务端，它们都会需要发送和接收消息的能力，才能进行正常的通信。  
在 VSCode 中，客户端包括ChannelClient和IPCClient，ChannelClient只处理最基础的频道相关的功能，包括：  
1. 获得频道getChannel。
2. 发送频道请求sendRequest。
3. 接收请求结果，并处理onResponse/onBuffer。

``` 
// 客户端
export class ChannelClient implements IChannelClient, IDisposable {
	getChannel<T extends IChannel>(channelName: string): T {
		const that = this;
		return {
			call(command: string, arg?: any, cancellationToken?: CancellationToken) {
				return that.requestPromise(channelName, command, arg, cancellationToken);
			},
			listen(event: string, arg: any) {
				return that.requestEvent(channelName, event, arg);
			}
		} as T;
	}
	private requestPromise(channelName: string, name: string, arg?: any, cancellationToken = CancellationToken.None): Promise<any> {}
	private requestEvent(channelName: string, name: string, arg?: any): Event<any> {}
	private sendRequest(request: IRawRequest): void {}
	private send(header: any, body: any = undefined): void {}
	private sendBuffer(message: VSBuffer): void {}
	private onBuffer(message: VSBuffer): void {}
	private onResponse(response: IRawResponse): void {}
	private whenInitialized(): Promise<void> {}
	dispose(): void {}
}
```
服务端包括ChannelServer和IPCServer，ChannelServer也只处理与频道直接相关的功能，包括：  
1. 注册频道registerChannel。
2. 监听客户端消息onRawMessage/onPromise/onEventListen。
3. 处理客户端消息并返回请求结果sendResponse。  
``` 
// 服务端
export class ChannelServer<TContext = string> implements IChannelServer<TContext>, IDisposable {
	registerChannel(channelName: string, channel: IServerChannel<TContext>): void {
		this.channels.set(channelName, channel);
	}
	private sendResponse(response: IRawResponse): void {}
	private send(header: any, body: any = undefined): void {}
	private sendBuffer(message: VSBuffer): void {}
	private onRawMessage(message: VSBuffer): void {}
	private onPromise(request: IRawPromiseRequest): void {}
	private onEventListen(request: IRawEventListenRequest): void {}
	private disposeActiveRequest(request: IRawRequest): void {}
	private collectPendingRequest(request: IRawPromiseRequest | IRawEventListenRequest): void {}
	public dispose(): void {}
}
```
作为频道的直接连接对象，ChannelClient和ChannelServer的发送和接收基本上是一一对应的，像sendRequest和sendResponse等等。但ChannelClient只能发送请求，而ChannelServer只能响应请求    
对消息的发送和接受，ChannelClient和ChannelServer会进行序列化(serialize)和反序列化(deserialize)：  
``` 
// 以 deserialize 举例：
function deserialize(reader: IReader): any {
	const type = reader.read(1).readUInt8(0);
	switch (type) {
		case DataType.Undefined: return undefined;
		case DataType.String: return reader.read(readSizeBuffer(reader)).toString();
		case DataType.Buffer: return reader.read(readSizeBuffer(reader)).buffer;
		case DataType.VSBuffer: return reader.read(readSizeBuffer(reader));
		case DataType.Array: {
			const length = readSizeBuffer(reader);
			const result: any[] = [];

			for (let i = 0; i < length; i++) {
				result.push(deserialize(reader));
			}

			return result;
		}
		case DataType.Object: return JSON.parse(reader.read(readSizeBuffer(reader)).toString());
	}
}
```
#### 连接
有了频道直接相关的客户端部分ChannelClient和服务端部分ChannelServer，但是它们之间需要连接起来才能进行通信。  
``` 
interface Connection<TContext> extends Client<TContext> {
	readonly channelServer: ChannelServer<TContext>; // 服务端
	readonly channelClient: ChannelClient; // 客户端
}
```
连接的建立，则由IPCServer和IPCClient负责。其中：  
- IPCClient基于ChannelClient，负责简单的客户端到服务端一对一连接
- IPCServer基于channelServer，负责服务端到客户端的连接，由于一个服务端可提供多个服务，因此会有多个连接

``` 
// 客户端
export class IPCClient<TContext = string> implements IChannelClient, IChannelServer<TContext>, IDisposable {
	private channelClient: ChannelClient;
	private channelServer: ChannelServer<TContext>;
	getChannel<T extends IChannel>(channelName: string): T {
		return this.channelClient.getChannel(channelName) as T;
	}
	registerChannel(channelName: string, channel: IServerChannel<TContext>): void {
		this.channelServer.registerChannel(channelName, channel);
	}
}

// 由于服务端有多个服务，因此可能存在多个连接
export class IPCServer<TContext = string> implements IChannelServer<TContext>, IRoutingChannelClient<TContext>, IConnectionHub<TContext>, IDisposable {
	private channels = new Map<string, IServerChannel<TContext>>();
	private _connections = new Set<Connection<TContext>>();
	// 获取连接信息
	get connections(): Connection<TContext>[] {}
	/**
	 * 从远程客户端获取频道。
	 * 通过路由器后，可以指定它要呼叫和监听/从哪个客户端。
	 * 否则，当在没有路由器的情况下进行呼叫时，将选择一个随机客户端，而在没有路由器的情况下进行侦听时，将监听每个客户端。
	 */
	getChannel<T extends IChannel>(channelName: string, router: IClientRouter<TContext>): T;
	getChannel<T extends IChannel>(channelName: string, clientFilter: (client: Client<TContext>) => boolean): T;
	getChannel<T extends IChannel>(channelName: string, routerOrClientFilter: IClientRouter<TContext> | ((client: Client<TContext>) => boolean)): T {}
	// 注册频道
	registerChannel(channelName: string, channel: IServerChannel<TContext>): void {
		this.channels.set(channelName, channel);
		// 添加到连接中
		this._connections.forEach(connection => {
			connection.channelServer.registerChannel(channelName, channel);
		});
	}
}
```

参考：  
[VSCode IPC通信机制](https://godbasin.github.io/front-end-playground/front-end-basic/deep-learning/vscode-ipc.html#vscode-%E5%A4%9A%E8%BF%9B%E7%A8%8B%E6%9E%B6%E6%9E%84)
