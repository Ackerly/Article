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

```



原文：
[叮~，你有几个系统级交互的React hooks待查收](https://mp.weixin.qq.com/s/twtT0xXZNVJmUDuvBrvgIQ)
