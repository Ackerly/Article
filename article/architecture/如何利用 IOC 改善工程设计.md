# 如何利用 IOC 改善工程设计
## 什么是 IOC 控制反转
考虑这样一个组件，它层层向下传递user和avatarSize属性，从而让深度嵌套的Link和Avatar组件读取到这些属性  
``` 
<Page user={user} avatarSize={avatarSize} />
<PageLayout user={user} avatarSize={avatarSize} />
<NavigationBar user={user} avatarSize={avatarSize} />
<Link href={user.permalink}>
    <Avatar user={user} size={avatarSize} />
</Link>
```
如果在最后只有Avatar组件真的需要user和avatarSize，那么层层传递这两个props就显得非常荣与，一旦Avatar组件需要更多从来自顶层组件到props，还需在中间层级一个一个加上去，这将变到非常麻烦  
一种无需context到解决方案是将Avatar组件自身传递下去，因为中间组件无需知道user或者avatarSize等props  
``` 
<Page user={user} avatarSize={avatarSize} />
<PageLayout user={user} avatarSize={avatarSize} />
<NavigationBar user={user} avatarSize={avatarSize} />
<Link href={user.permalink}>
    <Avatar user={user} size={avatarSize} />
</Link>
```
这种变化下，只有最顶部到Page组件需要知道Link和Avatar组件是如何使用user和avatarSize的。  
这种对组件对控制反转减少了在


原文:  
[如何利用 IOC 改善工程设计](https://mp.weixin.qq.com/s/AvoFeACu4wCjQhYeLkP2bw)
