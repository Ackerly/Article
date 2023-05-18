# 几个系统级交互的React hooks
## 监听网络状态
**定义**  
这个hook主要借助了navigator全局属性和offline/online事件监听  
``` 
import { useEffect, useState } from "react"

export const useNetwork = () => {

  const [state, setState] = useState<boolean>(navigator.onLine)

  useEffect(() => {
    window.addEventListener('offline', () => setState(false))
    window.addEventListener('online', () => setState(true))
    return () => {
      window.removeEventListener('offline', () => setState(false))
      window.removeEventListener('online', () => setState(true))
    }
  }, [])

  return state
}
```
**使用**  
``` 
const onlineState = useNetwork()

return onlineState ? <App /> : <OfflineTip />
```
类似的方法还可以探索很多有意思的事件属性，例如复制时加版权标识

## 复制加版权标识
**定义**  
``` 
import { useEffect } from "react"

export const useCopy = () => {

  useEffect(() => {

    const onCopy = () => navigator.clipboard.readText()
      .then((text) => {
        navigator.clipboard.writeText(text + ': @copyright萌萌哒草头将军')
      })
      
    addEventListener('copy', () => onCopy())
    
    return removeEventListener('copy', () => onCopy())
  }, [])
}
```
**使用**  
``` 
useCopy() // 复制：abc
// 粘贴：abc ：@copyright萌萌哒草头将军
```
## 监听窗口大小变化
**定义**  
``` 
import { useEffect, useState } from 'react';

export const useResize = () => {
  const [width, setWidth] = useState<number>(() =>
    window.document.body.offsetWidth);

  useEffect(() => {
    window.addEventListener('resize', (e) =>
      setWidth((e?.target as any).innerWidth),
    );
    return window.removeEventListener('resize', (e) =>
      setWidth((e?.target as any).innerWidth),
    );
  }, []);

  return width;
};
```
**使用**  
``` 
const width = useResize()

return width > 1200 ? <PcApp /> : width > 720 ? <PadApp /> : <PhoneApp />
```
**优化**  
为了防止因为频繁触发监听事件导致宽度也频繁变化，这里可以使用上期文章提到的useDeferredValue优化
``` 
const [width, setWidth] = useState<number>(() =>
    window.document.body.offsetWidth);
 // 延时更新⏰   
 const newWidth = useDeferredValue(value);
 
 useEffect(() => {
    window.addEventListener('resize', (e) =>
      setWidth((e?.target as any).innerWidth),
    );
    return window.removeEventListener('resize', (e) =>
      setWidth((e?.target as any).innerWidth),
    );
  }, []);
  
return newWidth;
```
或者添加防抖函数，不管触发多少次，在一定时间内，只更新最后一次  
``` 
function debounce(fn, delay) {
  let timer = null;
  return function (...arg) {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, arg);
    }, delay);
  }
}
```
然后
``` 
const [width, setWidth] = useState<number>(() =>
    window.document.body.offsetWidth);
 
 useEffect(() => {
    const fn = debounce((e) =>
      setWidth((e?.target as any).innerWidth),
    ), 2000)
    window.addEventListener('resize', (e) => fn(e);
    return window.removeEventListener('resize', (e) => fn(e)
 }, []);
  
 return width;

```
## 监听系统主题色变化
**定义**  
``` 
import { useEffect, useState } from "react";

export const useTheme = () => {

  const themeMedia = window.matchMedia("(prefers-color-scheme: light)");

  const [value, setValue] = useState<string | null>(themeMedia.matches ? 'light' : 'dark');

  useEffect(() => {
    
    themeMedia.addEventListener('change', e => 
      setValue(e.matches ? 'light' : 'dark')
    );

    return () => themeMedia.removeEventListener("change", e => 
      setValue(e.matches ? 'light' : 'dark')
    );
  }, []);

  return value;
};
```
**使用**  
``` 
const theme = useTheme()

return (
  <ConfigProvider
    theme={{
      algorithm: theme === 'light'
        ? theme.defaultAlgorithm
        : theme.darkAlgorithm,
    }}
  >
    <App />
  </ConfigProvider>
)
```
## 监听storage变化
**定义**  
错误定义
``` 
import { useEffect, useState } from "react";

export const useStorage = () => {

  const [value, setValue] = useState<string | null>();

  useEffect(() => {
    window.addEventListener("storage", (ev) => setValue(ev.newValue));
    return () =>
      window.removeEventListener("storage", (ev) => setValue(ev.newValue));
  }, []);
  
  return value;
};
```
storage仅仅对同源下的不同页面起作用，作为单页面应用SPA，还得再想办法。  
在不大动大改的前提下，重写了window.localStorage下的方法，命名保持一直，这样所有之前使用过的页面只需要引入我定义好的localStorage，同时去掉window.  
``` 
export const localStorage = {
  getItem: (key: string) => window.localStorage.getItem(key),
  setItem: (key: string, value: any) => window.localStorage.setItem(key, value),
  clear: () => return window.localStorage.clear(),
  removeItem: (key: string) => window.localStorage.removeItem(key),
  key: window.localStorage.key,
  length: window.localStorage.length,
}
```
接下来让每次的修改、删除、清空都可以被监听到。这里借助中介者模式，负责监听storage的变化。所以正确的定义方法如下  
``` 
import { useState } from "react"
// 中介者
const mediator = (function () {
  let topics: {[key: string]: {callback: (value: any) => void,
    uuid: number}[]} = {},
    uuid = 0;

  function subscribe (topic: string, callback: (value: any) => void) {
      uuid ++
      topics[topic] = topics[topic]
          ? [...topics[topic], { callback, uuid }]
          : [{ callback, uuid }]
  }

  function publish (topic: string, value: any) {

      if (topics[topic]) {
          topics[topic].map(item => item.callback(value))
      }
  }
  return {
      install: function (obj: any) {
          obj.uuid = uuid
          obj.publish = publish
          obj.subscribe = subscribe
          return obj
      } 
  }
})()
// 创建中介者函数
const createMediator = (obj: object) => mediator.install(obj)
// 记录所有监听的key
const keys: string[] = []
// 重新 window.localStorage
export const localStorage = {
  getItem: (key: string) => {
    return window.localStorage.getItem(key)
  },

  setItem: (key: string, value: any) => {
    // 防止重复发布
    if (!keys.includes(key)) keys.push(key)

    const sub = createMediator({})
    // 被修改就发布事件
    sub.publish(key, value)
    
    return window.localStorage.setItem(key, value)
  },
  clear: () => {

    const sub = createMediator({})
    // 被删除就每个key发布事件
    keys.map(key => sub.publish(key, undefined))
    // 发布后清空记录key的数组
    keys.length = 0

    return window.localStorage.clear()
  },
  removeItem: (key: string) => {

    keys.splice(keys.indexOf(key), 1)
    
    const sub = createMediator({})
    // 被移除就发布 undefined
    sub.publish(key, undefined)

    return window.localStorage.removeItem(key)
  },
  key: window.localStorage.key,
  length: window.localStorage.length,
}
// 监听key最新变化
export const useStorage = (key: string) => {
  // 默认初始值
  const [value, setValue] = useState<null | string>(window.localStorage.getItem(key))

  const sub = createMediator({})
  // 为指定的key订阅变更事件
  sub.subscribe(key, (value: any) => setValue(value))
  
  return value
}

```
使用
``` 
import { localStorage, useStorage } from './useStorage.ts'

export const App = () => {
  
  const random = useStorage('random')

  useEffect(() => console.log(random), [random])

  return (
    <div
      onClick={() =>
        localStorage.setItem('random', Math.random().toString())
      }
    >random: {random} </div>
  )
}

```


原文：
[叮~，你有几个系统级交互的React hooks待查收](https://mp.weixin.qq.com/s/twtT0xXZNVJmUDuvBrvgIQ)
