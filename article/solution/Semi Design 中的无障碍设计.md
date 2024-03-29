# Semi Design 中的无障碍设计
作为设计师，有责任也有义务让每一个人都能够轻松的访问所创造的产品。因此，Semi 成立了 A11y 专项，致力消除障碍并创造适合所有人的包容性产品体验。  
**什么是 A11y**  
A11y 是 Accessibility 的缩写，即无障碍设计。虽然无障碍设计广泛意义上被认为是为残障人士服务，但其实是为了所有人提供同等的体验。因为我们每一个人都可能会存在残障时刻。比如说：做了近视眼手术，暂时性无法使用眼睛。再比如，当强光照射时，看不清楚事物，这种就是情景性残障。  
## 了解用户需求
要设计开发包容性产品，首先需要了解不同用户的不同需求并考虑他们使用的辅助工具和方法。目前主要的障碍类型是视觉障碍、听觉障碍、肢体障碍和认知障碍，下面我们就简单介绍下各个障碍类型的使用产品的特点。  
**视觉障碍**  
视觉障碍人群里又有三个分类：盲人、低视力人群以及色盲。其中色盲用户和低视力人群在访问网站时遇到的障碍类似。大家可以通过 Coblis —色盲模拟器 来查看有色觉缺陷的人群眼中的世界。  
盲人用户依赖屏幕阅读器访问网站，通常他们会通过浏览特定元素（如标题、链接或表单元素）来导航页面。同时需要注意的是，这类用户一般不使用鼠标，通常使用特制的键盘来访问网站。  
低视力用户根据其视力障碍的性质不同有不同的需求。有些用户在没有放大情况下无法区分文本或其他内容，尤其是小文本；有些用户难以区分低色彩对比度的文本和图像；有些用户可能分不清色彩（色盲用户遇到是同样的问题）。这些用户通常会使用屏幕放大器、对比度增强或颜色反转软件以及屏幕阅读器等辅助工具来访问网站。  
**听觉障碍**  
听障用户借助助听器或者人工耳蜗等科技产品就能如常访问网站。因此相对其他障碍类型的用户，无障碍设计对于听障用户关照不大，但需要注意的是当我们使用视频等媒体工具时，应当为视频添加字幕来改善听障用户的访问体验，并且字幕对一些非母语用户也会有一定的帮助。  
其次，我们也需要避免无必要的自动播放，同时让用户可以选择关闭声音来屏蔽干扰  
**肢体障碍**  
用户摔坏了鼠标或者摔坏了胳膊的时候，还有一些患有慢性疾病的人，例如重复性压力损伤 (RSI)，应该限制或避免使用鼠标。  
这些用户访问网站时，仅依赖键盘，用户通过 Tab 键 及其他按键 在各个可交互元素之间移动。  
**认知障碍**  
有认知障碍的用户，难以专注于长篇密集的文本，无法理解复杂的结构。  
## 实施
### 颜色  
对比度  
对比度是可访问性中一个非常重要的指标。高对比度使视觉元素从背景中脱颖而出，更加引人注目。我们这里说到的对比度主要是指文字元素的对比和组件层面的对比。  
**文字元素对比**  
根据 WCAG 建议阈值：文本元素的文本和背景颜色之间的对比度至少应达到 4.5:1（包含组件内的文本），对于 18px 或更大的文本对比度可以降至 3:1，但对于禁用文本可以不受对比度要求的限制  
**组件状态和对比**  
所有可以操作的组件都需要有一个焦点状态（focus）。组件的 active、hover、focus 状态都需要满足与相邻颜色 3:1 的对比度要求。但不同状态之间没有对比度要求。  
**组件状态和对比**  
所有可以操作的组件都需要有一个焦点状态（focus）。组件的 active、hover、focus 状态都需要满足与相邻颜色 3:1 的对比度要求。但不同状态之间没有对比度要求。  
对于有描边的组件，只需满足描边颜色与底色的 3:1 对比即可。填充色与描边色之间不要求对比度。  
**例外情况**  
对于一些主要以阅读为主的组件通过屏幕阅读器就能让残障用户得到很好的访问了，如：Message、Banner 等，因此可以不用严格按照 AA 标准。但为了确保可操作性，组件内的可操作项还是需要满足对比度要求  
**A11y 主题包 及 相关插件推荐**  
为了更好的支持无障碍，上线了A11y 主题包。这里需要注意的是橙色这一色相的对比度在 WACG 2.0 的标准中，一直是一个老大难的问题。因为任何含有超过 50％ 黄色的颜色都会自然反射更多的光。大家在考虑无障碍场景时需要谨慎使用黄色、橙色、绿色和蓝绿色这类高风险颜色。  
深色模式时，也需要注意避免使用纯黑色作为主要的背景颜色。这是因为当你将白色文字置于纯黑色背景上时可能会产生光晕效应，导致用户阅读压力加大。在大文本或用户患有散光时，这种压力会尤为突出。  
虽然纯黑色的对比高很高，但 Semi Dark 模式下的 bg-0 和 text-0，是深灰色背景下的浅灰文本，在保证对比度的前提下也消除了阅读压力。 

> 光晕效应：光的传播超出其适当边界，在明亮图像的边缘周围形成雾所引起的效果

除了 A11y 主题包之外，也推荐视障用户使用 Chrome 插件 -- 高对比度模式 来访问网站。这个插件提供了增加对比度、灰度、反转颜色等多种模式，满足各类视障用户的需求。  
_不要使用颜色作为唯一的视觉提示_  
当你在传达重要信息、反馈提示时，不要将颜色作为传递信息的唯一途径。低视力用户或者色盲用户可能很难理解你想要表达的内容  
尝试使用添加图标、文本、下划线等，以确保所有人群都能收到相同的信息。典型例子就是 Form 的报错文本提示  
还有就是可以通过在图表中添加纹理及标签来表达差异，当然纹理对于常规的数据分析师来说增加了干扰项，不符合数分规范，但就无障碍而言却是合理的  
**键盘和焦点**  
键盘的可访问性是无障碍设计中最核心的一部分，有运动障碍的用户、盲人用户，以及一些有特殊操作习惯的用户都依赖键盘来导航和访问网站。  
_键盘快捷键_  
这类用户可以通过一些键盘快捷键来实现浏览网站：  
- Tab 键切换焦点：Tab 键顺序应遵循可预测的顺序层次结构，如：从上到下，从左到右。一些关键元素被获取焦点时，应当显示该元素的提示信息；失去焦点后，提示消失；
- 箭头：在相关的单选按钮、菜单项或小部件项之间导航；
- Enter：激活按钮，提交表单等；
- 空格：激活按钮，选中开关等；
- Esc ：退出各类弹层

_焦点_  
用键盘 Tab 键来浏览网页上的可交互元素最重要的一个点就是元素的焦点状态，因为焦点状态能让键盘用户知道焦点当前在哪里。因此 Semi Design 为每一个可交互组件设计了 foucs 状态。同时为了确保纯键盘用户可以同普通用户一样的浏览体验  
Semi 组件的焦点需要遵循一下原则：  
- 初始焦点：为了使用户能够有效地完成任务，请始终为任务设置初始焦点。焦点切换时，如果当前焦点控件被覆盖，焦点需要自动切换到新页面的第一个焦点区域。初始焦点设置时，需要注意：  
  - 初始焦点一般设置为任务中的第一个逻辑交互元素或第一个元素
  - 当这个任务包含了一个不可逆转过程的最后一个步骤，比如：删除数据等，那么这个初始焦点最好放在破坏性最小的可交互元素上，如：关闭按钮
  - 若任务为阅读任务或继续处理任务时，建议将初始焦点设置在最可能常用的交互元素上，如：确定按钮
- 导航可逆：用户通过 Tab 键切换到下一个焦点，就一定可以通过 Shift + Tab 切换会上一个焦点；
- 可返回：如果当前聚焦的元素消失，焦点状态应始终返回到之前的位置。例如，关闭模态可能意味着您的焦点在关闭按钮上；当模态关闭时，您应该将焦点返回到打开模态的按钮

_示例_   
Semi 每一个组件都提供了详细的键盘和焦点的说明，大家可以去 Semi 站点详细体验（目前部分组件尚在施工中），下面是 Popconfirm 键盘交互的示例

**语义化**  
虽然原生元素具有浏览器内置的语义信息，比如：button 自带 role 为 button。但对于自定义组件，Semi 为它们使用了 ARIA（Accessible Rich Internet Applications） 来添加语义信息，帮助屏幕阅读器用户更好与组件进行交互  
ARIA 能够实现以下辅助功能：  
- 让用户知道当前元素是啥，当前元素是干什么的；
- 当前元素是啥状态，是否可以选择，是否禁用，是否有弹出层等；
- 当前我处于列表的第几项，总共有几项等

Semi 组件实现了全面语义化，再结合键盘交互，让障碍人群能够更加顺畅丝滑的访问网站  
**媒体**  
_图片的替代文本_  
低视力人群依靠屏幕阅读来访问网站。若不对图片进行处理，屏幕阅读器阅读到图片时，视障用户只能听到这里有一个图片，但是不能知道这个图片具体表达的信息。甚至可能听到是这个图片的文件名称。  
为了避免上述情况的发生，我们为图片元素提供了替代性的文本。如：Semi 头像等图片类型的组件使用了 alt 属性来解释图片的内容。  
_视频和音频_  
为音频或视频内容提供字幕，屏幕阅读器就可以朗读出这些文本信息，这样听障用户就可以很好的访问到这些信息了。  
**动画**  
在设计动画或视频时，应当避免闪烁的内容。页面上的任何内容都不应当每秒在屏幕上闪烁超过 3 次，因为它可能会引发一些用户的癫痫发作。除非闪烁的内容足够小且闪烁内容的对比度低不包含过多的红色。  
1997年日本动漫《神奇宝贝》第38集《电脑战士多边兽》里皮卡丘用电击摧毁导弹，导弹爆炸后出现异常鲜艳并且频繁闪烁的红蓝交替画面导致将近700个儿童癫痫发作。  
作为内容设计者，我们有责任和义务确保我们生产的内容的安全性。  
**布局**  
合理的简洁的布局方式让用户可以轻松地探索和发现内容。设计师们在设计界面时，应当确保  
- 信息是清晰、简洁且易于浏览的；
- 考虑视觉层次结构，将内容分成简短的相关部分，并避免长段落；
- 整体布局造成的浏览顺序是线性且一致的；
- 在 200% 的放大器下仍然具有可阅读性

这对依赖屏幕阅读器的用户、屏幕放大器用户以及有认知障碍的用户非常有帮助，且不会丢失信息或功能。  
**使用**  
_校验颜色非唯一信息_  
想要校验自己的设计是否满足“颜色不是唯一的视觉提示”这个标准时，可以使用 chrome 插件 -- 高对比度模式 ，将模式选择为【反转灰色】，将界面转为去色界面，测试在没有颜色的情况下是否还能准确的获取全部信息  
_页面焦点顺序_  
如果想要整体的键盘交互使用顺畅，页面级别的焦点顺序就很重要。可以 figma 插件 A11y - Focus Orderer 来标记焦点顺序。标记后，可以让研发同学通过 tabIndex 来实现你的诉求。  
_受控组件的焦点处理_  
使用了 Semi 组件的一些受控组件时，比如 Popover ，由于受控组件 Semi 无法确认触发器的触发方式，用户通过 esc 关闭 Popover后，焦点将会丢失。这里需要开发者手动处理一下，将焦点切换到下一个可交互元素上  
_图片和图标语义注释_  
当你的设计稿中包含图片或 icon 时，使用 alt 标注替代文本。当然如果你的设计的按钮中包含了图标和标签，那么则不需要添加 alt 属性，不然屏幕阅读器就会播报两遍一样的信息，反而影响用户体验。

原文: 
[Semi Design 中的无障碍设计](https://mp.weixin.qq.com/s/kBuPJ8_WFyYnQTtKHz6Ehg)
