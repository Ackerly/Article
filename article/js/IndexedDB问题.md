# IndexedDB 问题
报错信息为 QuotaExceededError，分类为未分类，
## localforage使用
localforage通过异步存储来改进应用程序的离线体验。能存储多种类型的数据，而不仅仅是字符串。采用优雅降级，优先使用 IndexedDB 或 WebSQL，若浏览器不支持，则使用 localStorage。还可设置过期时间expire，在getItem之前会做是否清除判断。  
localforage适用的业务场景:  
1. 本地储存的数据需要设置过期时间，超过指定时间后清除重新调用最新数据
2. 可能会有大量数据需要本地存储
3. IndexedDB为异步操作，不会阻塞主线程

## 原因
不同于浏览器会为localStorage预留5MB左右的储存空间，而IndexedDB的配额是动态计算的，虽然浏览器实现各不相同，但可用存储量通常取决于设备上可用的存储量。  
**如何查看有多少可用存储空间**  
``` 
if (navigator.storage && navigator.storage.estimate) {
  const quota = await navigator.storage.estimate();
  // quota.usage -> Number of bytes used.
  // quota.quota -> Maximum number of bytes available.
  const percentageUsed = (quota.usage / quota.quota) * 100;
  console.log(`You've used ${percentageUsed}% of the available storage.`);
  const remaining = quota.quota - quota.usage;
  console.log(`You can write up to ${remaining} more bytes.`);
}
```
当手机内存为1GB时，webview的剩余储存空间为500MB左右，随着手机内存的不断减少，浏览器的配额也会减少，直到手机内存为300MB左右时，配额降为0，当写入的数据量大于剩余配额时，便会抛出QuotaExceededError。

## 总结
1. 内存严重不足的极端情况会导致IndexedDB使用异常。
2. 使用新API时应该做一些异常处理。虽然localforage有降级机制，但是在支持IndexedDB的端遇到IndexedDB报错时并不会自动降为localStorage，需要在开发过程中手动处理。


原文: 
[IndexedDB 使用引起的线上事故](https://juejin.cn/post/7033303792443473934)
