# VSCode 事件系统设计
## VS Code 中的事件都包括了哪些能力
``` 
// 主要定义了一些接口协议，以及相关方法
// 使用 namespace 的方式将相关内容包裹起来
export namespace Event {
	// 来看看里面比较关键的一些方法

	// 给定一个事件，返回另一个仅触发一次的事件
  export function once<T>(event: Event<T>): Event<T> {}

  // 给定一连串的事件处理功能（过滤器，映射等），每个事件和每个侦听器都将调用每个函数
  // 对事件链进行快照可以使每个事件每个事件仅被调用一次
  // 以此衍生了 map、forEach、filter、any 等方法此处省略
	export function snapshot<T>(event: Event<T>): Event<T> {}

	// 给事件增加防抖
	export function debounce<T>(event: Event<T>, merge: (last: T | undefined, event: T) => T, delay?: number, leading?: boolean, leakWarningThreshold?: number): Event<T>;

	// 触发一次的事件，同时包括触发时间
	export function stopwatch<T>(event: Event<T>): Event<number> {}

	// 仅在 event 元素更改时才触发的事件
	export function latch<T>(event: Event<T>): Event<T> {}

	// 缓冲提供的事件，直到出现第一个 listener，这时立即触发所有事件，然后从头开始传输事件
	export function buffer<T>(event: Event<T>, nextTick = false, _buffer: T[] = []): Event<T> {}

  // 可链式处理的事件，支持以下方法
	export interface IChainableEvent<T> {
		event: Event<T>;
		map<O>(fn: (i: T) => O): IChainableEvent<O>;
		forEach(fn: (i: T) => void): IChainableEvent<T>;
		filter(fn: (e: T) => boolean): IChainableEvent<T>;
		filter<R>(fn: (e: T | R) => e is R): IChainableEvent<R>;
		reduce<R>(merge: (last: R | undefined, event: T) => R, initial?: R): IChainableEvent<R>;
		latch(): IChainableEvent<T>;
		debounce(merge: (last: T | undefined, event: T) => T, delay?: number, leading?: boolean, leakWarningThreshold?: number): IChainableEvent<T>;
		debounce<R>(merge: (last: R | undefined, event: T) => R, delay?: number, leading?: boolean, leakWarningThreshold?: number): IChainableEvent<R>;
		on(listener: (e: T) => any, thisArgs?: any, disposables?: IDisposable[] | DisposableStore): IDisposable;
		once(listener: (e: T) => any, thisArgs?: any, disposables?: IDisposable[]): IDisposable;
	}
	class ChainableEvent<T> implements IChainableEvent<T> {}

  // 将事件转为可链式处理的事件
	export function chain<T>(event: Event<T>): IChainableEvent<T> {}

  // 来自 DOM 事件的事件
	export function fromDOMEventEmitter<T>(emitter: DOMEventEmitter, eventName: string, map: (...args: any[]) => T = id => id): Event<T> {}

  // 来自 Promise 的事件
	export function fromPromise<T = any>(promise: Promise<T>): Event<undefined> {}
}
```
Event中主要是一些对事件的处理和某种类型事件的生成,除了常见的once和 DOM 事件等兼容，还提供了比较丰富的事件能力：
- 防抖动
- 可链式调用
- 缓存
- Promise 转事件

## VS Code 中的事件的触发和监听的实现 
``` 
// 这是事件发射器的一些生命周期和设置
export interface EmitterOptions {
	onFirstListenerAdd?: Function;
	onFirstListenerDidAdd?: Function;
	onListenerDidAdd?: Function;
	onLastListenerRemove?: Function;
	leakWarningThreshold?: number;
}

export class Emitter<T> {
  // 可传入生命周期方法和设置
	constructor(options?: EmitterOptions) {}

	// 允许大家订阅此发射器的事件
	get event(): Event<T> {
		// 此处会根据传入的生命周期相关设置，在对应的场景下调用相关的生命周期方法
	}

	// 向订阅者触发事件
	fire(event: T): void {}

  // 清理相关的 listener 和队列等
	dispose() {}
}
```
Emitter以Event为对象，以简洁的方式提供了事件的订阅、触发、清理等能力
## 项目中的时间管理
``` 
class WindowManager {
  public static readonly INSTANCE = new WindowManager();
  // 注册一个事件发射器
	private readonly _onDidChangeZoomLevel = new Emitter<number>();
  // 将该发射器允许大家订阅的事件取出来
	public readonly onDidChangeZoomLevel: Event<number> = this._onDidChangeZoomLevel.event;

	public setZoomLevel(zoomLevel: number, isTrusted: boolean): void {
		if (this._zoomLevel === zoomLevel) {
			return;
		}
		this._zoomLevel = zoomLevel;
		// 当 zoomLevel 有变更时，触发该事件
		this._onDidChangeZoomLevel.fire(this._zoomLevel);
	}
}
```
事件的使用方式主要包括：  
- 注册事件发射器
- 对外提供定义的事件
- 在特定时机向订阅者触发事件

通过挂载全局对象的方式来提供使用：
``` 
// 对外提供一个调用全局实例的方法
export function onDidChangeZoomLevel(callback: (zoomLevel: number) => void): IDisposable {
	return WindowManager.INSTANCE.onDidChangeZoomLevel(callback);
}

// 其他地方的调用方式
import { onDidChangeZoomLevel } from 'vs/base/browser/browser';
let zoomListener = onDidChangeZoomLevel(() => {
  // 该干啥干啥
});
```
通过创建实例调用来直接监听相关事件:  
``` 
const instance = new WindowManager(opts);
instance.onDidChangeZoomLevel(() => {
  // 该干啥干啥
});
```
### 事件满天飞，会不会导致性能问题
果在某个组件里做了事件订阅这样的操作，当组件销毁的时候是需要取消事件订阅的。否则该订阅内容会在内存中一直存在，除了一些异常问题，还可能引起内存泄露。
``` 
// 只选取局部关键代码摘要
export class Scrollable extends Disposable {
	private _onScroll = this._register(new Emitter<ScrollEvent>());
	public readonly onScroll: Event<ScrollEvent> = this._onScroll.event;

	private _setState(newState: ScrollState): void {
		const oldState = this._state;
		if (oldState.equals(newState)) {
			return;
		}
    this._state = newState;
    // 状态变更的时候，触发事件
		this._onScroll.fire(this._state.createScrollEvent(oldState));
	}
}
```
使用了this._register(new Emitter<T>())这样的方式注册事件发射器，该方法继承自Disposable。  
Disposable的实现
``` 
export abstract class Disposable implements IDisposable {
  // 用一个 Set 来存储注册的事件发射器
	private readonly _store = new DisposableStore();

	constructor() {
		trackDisposable(this);
	}

  // 处理事件发射器
	public dispose(): void {
		markTracked(this);

		this._store.dispose();
	}

  // 注册一个事件发射器
	protected _register<T extends IDisposable>(t: T): T {
		if ((t as unknown as Disposable) === this) {
			throw new Error('Cannot register a disposable on itself!');
		}
		return this._store.add(t);
	}
}
```
每个继承Disposable类都会有管理事件发射器的相关方法，包括添加、销毁处理等。这个Disposable并不只是服务于事件发射器，它适用于所有支持dispose()方法的对象: 
> Dispose 模式主要用来资源管理，资源比如内存被对象占用，则会通过调用方法来释放。

``` 
export interface IDisposable {
	dispose(): void;
}
export class DisposableStore implements IDisposable {
	private _toDispose = new Set<IDisposable>();
	private _isDisposed = false;

  // 处置所有注册的 Disposable，并将其标记为已处置
  // 将来添加到此对象的所有 Disposable 都将在 add 中处置。
	public dispose(): void {
		if (this._isDisposed) {
			return;
		}

		markTracked(this);
		this._isDisposed = true;
		this.clear();
	}

	// 丢弃所有已登记的 Disposable，但不要将其标记为已处置
	public clear(): void {
		this._toDispose.forEach(item => item.dispose());
		this._toDispose.clear();
	}

  // 添加一个 Disposable
	public add<T extends IDisposable>(t: T): T {
    markTracked(t);
    // 如果已处置，则不添加
		if (this._isDisposed) {
			// 报错提示之类的
		} else {
      // 未处置，则可添加
			this._toDispose.add(t);
		}
		return t;
	}
}
```
VS Code 中是这样管理事件：  
- 抹平 DOM 事件等差异，提供标准化的Event和Emitter能力。
- 通过注册Emitter，并对外提供类似生命周期的方法onXxxxx的方式，来进行事件的订阅和监听。
- 通过提供通用类Disposable，统一管理多个事件发射器（或其他资源）的注册、销毁

## 对于订阅者来说，要怎么销毁订阅的 Listener 
``` 
// 1. 注册事件发射器
export class Button extends Disposable {
  // 注册一个事件发射器，可使用 this._onDidClick.fire(xxx) 来触发事件
	private _onDidClick = this._register(new Emitter<Event>());
  get onDidClick(): BaseEvent<Event> { return this._onDidClick.event; }
}

// 2. 订阅某个事件
export class QuickInputController extends Disposable {
  // 省略很多其他非关键代码
  private getUI() {
    const ok = new Button(okContainer);
    ok.label = localize('ok', "OK");
    // 注册一个 Disposable，用来订阅某个事件
		this._register(ok.onDidClick(e => {
			this.onDidAcceptEmitter.fire();
    }));
  }
}
```
当某个类被销毁时，会发生以下事情：
- 它所注册的事件发射器会被销毁，而事件发射器中的 Listener、队列等都会被清空。
- 它所订阅的一些事件会被销毁，订阅中的 Listener 同样会被移除。

订阅事件的 Listener 的移除：
``` 
export class Emitter<T> {
  get event(): Event<T> {
		if (!this._event) {
			this._event = (listener: (e: T) => any, thisArgs?: any, disposables?: IDisposable[] | DisposableStore) => {
        // 若无队列，则新建一个
				if (!this._listeners) {
					this._listeners = new LinkedList();
				}
        // 往队列中添加该 Listener，同时返回一个移除该 Listener 的方法
				const remove = this._listeners.push(!thisArgs ? listener : [listener, thisArgs]);

        let result: IDisposable;
        // 返回一个带 dispose 方法的结果，dispose 执行时会移除该 Listener
				result = {
					dispose: () => {
						result.dispose = Emitter._noop;
						if (!this._disposed) {
							remove();
						}
					}
				};
				if (disposables instanceof DisposableStore) {
					disposables.add(result);
				} else if (Array.isArray(disposables)) {
					disposables.push(result);
				}

				return result;
			};
		}
		return this._event;
  }
}
```
VS Code 中事件相关的管理的设计：
- 提供标准化的Event和Emitter能力
- 通过注册Emitter，并对外提供类似生命周期的方法onXxxxx的方式，来进行事件的订阅和监听
- 通过提供通用类Disposable，统一管理相关资源的注册和销毁
- 通过使用同样的方式this._register()注册事件和订阅事件，将事件相关资源的处理统一挂载到dispose()方法中


参考：  
[VSCode源码解读 事件系统设计](https://godbasin.github.io/front-end-playground/front-end-basic/deep-learning/vscode-event.html#q1-vs-code-%E4%B8%AD%E7%9A%84%E4%BA%8B%E4%BB%B6%E7%AE%A1%E7%90%86%E4%BB%A3%E7%A0%81%E5%9C%A8%E5%93%AA)
