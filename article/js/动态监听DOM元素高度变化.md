# 动态监听DOM元素高度变化
在我们的网页中有一个固定区域，这个区域会用于渲染从后端拉取的含有图片等资源的富文本字符串。  
他需要在内容不超过一个最大高度的时候完全显示所有内容，超过最大内容后仅展示最大高度范围内的内容，超出部分隐藏，并通过一个按钮 “展示更多” 来给用户展示更多的选择。  
在这看似简单的需求当中，其实涉及到了一个难点，那就是怎样动态的监听到内容区域的高度变化？  
因为在这里面会含有图片资源，他们在渲染的时候会发起网络请求，等待图片加载完成后触发浏览器重排，该区域的高度被撑开。  
因此，内容区域的高度是动态变化，且变化的时间点是未知的，那么怎样知道我们的内容区高度发生了变化呢？  
做了以下几种尝试：  
- MutationObserver
- IntersectionObserver
- ResizeObserver
- 监听所有资源的 onload 事件
- iframe

## MutationObserver
> MutationObserver 接口提供了监视对 DOM 树所做更改的能力。它被设计为旧的 Mutation Events 功能的替代品，该功能是 DOM3 Events 规范的一部分

**observe(target, options)**  
这个方法会根据传入的 options 配置，观察 DOM 树中的单个 Node 或者所有的子孙节点的变化。  
怎么使用这个 API 来监听目标区域的高度变化呢？  
1. 首先我们要创建对该区域的 dom 根结点引用：
``` 
// useRef创建引用
const contentRef = useRef();

// 绑定ref
<div
  className="content"
  dangerouslySetInnerHTML={{ __html: details }}
  style={{ maxHeight }}
  ref={contentRef}
/>;
```
2. 然后我们需要创建一个 MutationObserver 实例：
``` 
const [height, setHeight] = useState(-1);
const [observer, setObserver] = useState<MutationObserver>(null!);
useEffect(() => {
  const observer = new MutationObserver((mutationList) => {
    if (height !== contentRef.current?.clientHeight) {
      console.log("高度变化了！");
      setHeight(contentRef.current.clientHeight);
    }
  });
  setObserver(observer);
}, []);
```
3. 当我们的 ref 或者 observer 发生变化的时候，对 ref 节点进行观察：
``` 
useEffect(() => {
  if (!observer || !contentRef.current) return;
  observer.observe(contentRef.current, {
    childList: true, // 子节点的变动（新增、删除或者更改）
    attributes: true, // 属性的变动
    characterData: true, // 节点内容或节点文本的变动
    subtree: true, // 是否将观察器应用于该节点的所有后代节点
  });
}, [contentRef.current, observer]);
```

完整代码：  
``` 
const Details = () => {
    // useRef创建引用
    const contentRef = useRef();
    const [height, setHeight] = useState(-1);
    const [observer, setObserver] = useState<MutationObserver>(null!);

    useEffect(() => {
          const observer = new MutationObserver((mutationList) => {
            if (height !== contentRef.current?.clientHeight) {
                console.log('高度变化了！');
                setHeight(contentRef.current.clientHeight);
            }
          });
          setObserver(observer);
    }, []);

    useEffect(() => {
          if (!observer || !contentRef.current) return
          observer.observe(contentRef, {
            childList: true, // 子节点的变动（新增、删除或者更改）
            attributes: true, // 属性的变动
            characterData: true, // 节点内容或节点文本的变动
            subtree: true// 是否将观察器应用于该节点的所有后代节点
          });
    }, [contentRef.current, observer]);

    // 绑定ref
    return<div className="content" dangerouslySetInnerHTML={{ __html: details }} style={{ maxHeight }} ref={contentRef} />
}
```
经过上面的一番操作之后，发现根本达不到效果，因为我们的 css 属性根本没有发生变化（我们是通过 maxHeight 来约束容器的高度的）, 但是资源加载完毕之后，浏览器重排根本没有产生 css 属性的变化，它的高度是自动计算的  
它确实可以监听到认为修改容器的高度产生的变化，比如：contentRef.current.style.height = '1000px'，这个 api 是可以监听到这一操作的，但是并不符合我们的场景  

## IntersectionObserver
IntersectionObserver 这个 API，它可以监听一个元素是否进入用户视野,使用起来和 MutationObserver 几乎一样，只是名字不一样而已  
它监听的值里面有一个比较重要的属性：intersectionRatio  
当用户滚动网页的时候（或者不滚动，此时目标区域已经出现在屏幕中），可以得到 intersectionRatio 的值，通过判断这个值是否等于 1 来决定要不要展示 “展示更多” 按钮  
经过编码实现后，发现滚动事件发生的时候，intersectionRatio 的变化是不可靠的，有时候完全可见了，但是它并不等于 1。经过多轮实验，结果依然如此。但是它确实可以用来判断一个元素是否进入用户视野
## ResizeObserver
这个 API 就是专门监听 DOM 尺寸变化的，只不过它还处于试验阶段，各浏览器的兼容性很差，所以基本不考虑  

## 监听所有资源的 onload 事件
监听所有带有 src 属性的 DOM 元素的 onload 事件，通过他的回调来判断当前容器的高度情况  
这种实现方式，在思路上是完全符合目的的，具体做法参考如下：  
``` 
const [height, setHeight] = useState(-1);
const [showMore, setShowMore] = useState(false);
// contentRef 的定义见 MutationObserver 一节
useEffect(() => {
  const sources = contentRef.current.querySelectorAll("[src]");
  sources.onload = () => {
    const height = contentRef?.current?.clientHeight ?? 0;
    const show = height >= parseInt(MAX_HEIGHT, 10);

    setHeight(height);
    setShowMore(show);
  };
}, []);
```
通过这种方式可以实现对富文本中的图片进行加载后，对容器高度进行相应的判断。  
但是这种方式，存在不确定性，即无法判断是否找齐了所有高度由内容撑开的资源。  
## Iframe
既然 window 可以监听到 resize 事件，可以利用 iframe 来达到同样的效果，具体做法就是在容器里面嵌套一个隐藏的高度为 100% 的 iframe，通过监听他的 resize 事件，来判断当前容器的高度。  
具体实现方式如下：  
``` 
const Detail: FC<{}> = () => {
  const ref = useRef<HTMLDivElement>(null);
  const ifr = useRef<HTMLIFrameElement>(null);
  const [height, setHeight] = useState(-1);
  const [showMore, setShowMore] = useState(false);
  const [maxHeight, setMaxHeight] = useState(MAX_HEIGHT);
  const introduceInfo = useAppSelect(
    (state) => state.courseInfo?.data?.introduce_info ?? {}
  );
  const details = introduceInfo.details ?? "";
  const isFolded = maxHeight === MAX_HEIGHT;
  const onresize = useCallback(() => {
    const height = ref?.current?.clientHeight ?? 0;
    const show = height >= parseInt(MAX_HEIGHT, 10);

    setHeight(height);
    setShowMore(show);
    if (ifr.current && show) {
      ifr.current.remove();
    }
  }, []);

  useEffect(() => {
    if (!ref.current || !ifr.current?.contentWindow) return;
    ifr.current.contentWindow.onresize = onresize;
    onresize();
  }, [details]);

  if (!details) returnnull;

  return (
    <section className="section detail-content">
      <div className="content-wrapper">
        <div
          className="content"
          dangerouslySetInnerHTML={{ __html: details }}
          style={{ maxHeight }}
          ref={ref}
        />
        {/* 这个iframe是用来动态监听content高度的变化的 */}
        <iframe title={IFRAME_ID} id={IFRAME_ID} ref={ifr} />
      </div>
      {isFolded && showMore && (
        <>
          <div
            className="show-more"
            onClick={() => {
              setMaxHeight(isFolded ? "none" : MAX_HEIGHT);
            }}
          >
            查看全部
            <IconArrowDown className="icon" />
          </div>
          <div className="mask" />
        </>
      )}
    </section>
  );
};
```

参考:
[动态监听DOM元素高度变化](https://mp.weixin.qq.com/s/adz6CG-r17qh6hC8AduUZA)
