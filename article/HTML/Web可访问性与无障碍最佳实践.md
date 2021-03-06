# Web可访问性与无障碍最佳实践
通过调整 HTML 的结构和标签，增加 HTML 属性，配合 CSS 和 JavaScript 等手段来提高页面的可访问性和无障碍性。例如使用了 a 标签制作了按钮，如果不进行额外的优化，读屏软件在朗读时会读作"文字内容 链接"，但实际上该 a 标签是用作按钮使用，因此可以在标签上添加 role="button" 属性。此时，读屏软件会读作"文字内容 按钮"。  
读屏软件在朗读时会在结尾朗读出元素的属性，这也是无障碍优化中重要的一环。无障碍优化就是要解决如何使得元素的属性被正确识别，如何使得元素的内容被清晰准确地朗读，如何排除干扰元素等问题。  
Web 无障碍开发的基础知识参考资料:  
- 页面无障碍基础知识：Accessible Rich Internet Applications (WAI-ARIA) 1.1
- 无障碍教程：可访问性 - 学习 Web 开发 | MDN
- 读屏软件的使用：VoiceOver - iPhone 使用手册

## WeRead H5 的开发细则
**DOM 的顺序很重要**  
读屏软件在读屏时默认按照 DOM 的顺序朗读，因此如果 DOM 的顺序与内容的语义顺序不一致，例如使用了 flex-direction: row-reverse; 使得内容的顺序倒序显示，会使得内容难以理解。因此尽量避免使用会影响到 DOM 视觉顺序的样式，如果无法避免，需要手动设置 tabIndex 属性，告知读屏软件正确的内容顺序。  
**为非文本元素提供文本说明**  
<img> 标签需要加上 alt 属性，读屏软件会自动读出 alt 的内容，例如 alt 内容为"一只目光汹汹凝视远方的猫"，那么会被读作"一只目光汹汹凝视远方的猫 图像"。如果没有添加 alt 属性，那么仅会读作"图像"，视障用户会完全无法理解其实际含义。  
<img> 标签出现在 <a> 标签内部，作为一个图像链接时，应在 <a> 上使用 title 属性，<img>标签可不加 alt 属性。  
<video> 标签需要加上 title 属性，例如 title 内容为"一只正在奔跑的猫"，那么会被读作一只正在奔跑的猫 视频"。  
**使用语义化的元素**  
语义化的 HTML 标签，例如 <header> <footer> <nav> <section> <main> <aside> <button>，使用语义化的标签，主要影响两个方面：  
- 选中元素时是否会整块选中
- 朗读时结尾会加上怎样的修饰词

其中默认设置下，目前仅 <button> 标签可以使得选中元素时会整块选中，而不单独选中子元素。至于修饰词这里列举具体的情况：
- <header> 读作"xxx 横幅 标志性内容"。
- <footer> 读作"xxx 页脚 标志性内容"。
- <nav> 读作"xxx 导航 标志性内容"。
- <section> 仅读作"xxx"，没有结尾修饰词。
- <main> 读作"xxx 主要 标志性内容"。
- <aside> 读作"xxx 补充 标志性内容"。
- <a> 读作"xxx 链接"。

在浏览器内部，使用语义化标签会隐式加上特定的 role 属性，最后朗读时的结尾修饰词也正是这些 role 属性的值以及分类，其他 role 的值朗读时也可以以此类推，而以上标签与 role 属性具体对应的关系如下：  


|  HTML 标签   | role 属性值  |  
|  ----  | ----  |
| header  | banner |
| footer  | contentinfo |
| nav  | navigation |
| section  | region |
| main  | main |
| aside  | complementary |
| button  | button |
| a  | link |

**role 属性**  
如果出于其他考虑，使用了非对应语义的标签，例如开头提到的使用 a 标签实现按钮，就需要添加 role="button" 属性来声明这是一个按钮。同理，其他类似情况也可以这样处理，主要的就是影响朗读时的修饰词。  
**禁用状态使用 disabled 属性**  
使用特定的 class 来增加禁用态样式是常见的手法，但由于 class 语义并不能被读屏软件识别，因此读屏时无法知道当前处于禁用态。可以改为使用 disabled 属性实现禁用态，例如：  
``` 
<input type="search" name="q" placeholder="请输入用户名" aria-label="搜索用户" disabled/>
/* 禁用态样式 */
input[disabled] {
    opacity: .5;
}
```
会读作 搜索用户 请输入用户名 变暗 搜索栏，读屏软件会用"变暗"这个词表示搜索栏处于不可用的状态。而对于没有 disabled 属性的标签，例如 a 标签，可以使用 aria-disabled 属性达到同样的效果。  
**可使用 aria 标签向不存在原生语义的元素添加语义**  
aria-label="screen reader only label"，用于添加朗读时的描述，读屏时会读出其中的内容，而忽略标签的原有的文字，例如为 a 标签同时添加 role="button" 和 aria-label="额外的按钮描述"，最终会朗读成"额外的按钮描述 按钮"。  
aria-controls="main"，用于给操作按钮关联控制区域，VoiceOver 上这个属性没有任何作用，但 PC 读屏软件中，添加了该属性后，可以把焦点从按钮快速移动到被控制区域。  
aria-live="true"，添加了该属性的元素，在其内容发送变化时，读屏软件会自动读出变化后的新内容。可以用于会动态刷新的元素，例如发现卡片上的“XXX人参与活动”，书城的换一批功能，用于监听实时变化的数据。实际效果可以参考这个 demo。  
**动画**  
可在 iOS 下通过 CSS 选择器 @media(prefers-reduced-motion) 来针对开启了“避免动画”的用户取消动画。  
**隐藏屏幕外的元素**  
确保屏幕外的内容已通过 display: none 或 visibility: hidden 隐藏（如浮动出现的 alert 和 banner 等），如没有隐藏，读屏软件仍会读出元素内容，但屏幕外的元素通常不希望被读出，如果不方便使用样式进行隐藏，可以为元素添加 aria-hidden="true" 属性，元素则会被读屏软件忽略。  
## 常用场景
**图像的编写**  
图像需要补充文字描述，补充时需要使用具体的内容标题，例如书籍封面，可以使用书籍名称，而不要直接统一描述为"书籍封面"，同理用户头像也应该使用用户名作为描述文字。  
**按钮的编写**  
在 H5 中，为了避免一些浏览器默认样式的干扰，以及制作点击效果（具体原因），目前采用 a 标签实现。但从无障碍的角度考虑，a 标签默认会被当做链接处理，读屏时会读作"链接内的文字 链接"。  
**基础无障碍适配**  
需要加上 aria="button" 属性，例如：  
``` 
<a class="test_btn" role="button" href="javascript:;">文字</a>
```
读屏时会读出"文字 按钮"。  
**增加描述文字**  
如果 a 标签内本身没有文字，例如以图片、背景色和边框制作的按钮，还需要加上 aria-label="描述文字"，读屏时会读作"描述文字 按钮"的形式。当 a 标签内的文字对于视障人士不足以描述清楚按钮作用时（例如需要结合上下的元素，或者结合按钮本身的背景图才能理解按钮的含义时），也可以加上 aria-label 属性，aria-label 的内容会被优先读出，例如：  
``` 
<a class="test_btn" role="button" href="javascript:;" aria-label="更完整的描述">文字</a>
```
**多重标签嵌套**  
a 标签内容如果有嵌套的标签，并不会影响文字被读出，例如：  
``` 
<a class="test_btn" role="button" href="javascript:;">
    <span class="test_btn_inner">
        <span class="test_btn_inner_text">文字</span>
    </span>
</a>
```
读屏时仍会读出"文字 按钮"。

## 整块可点击元素的编写
在遇到 banner 等本身由多个子元素组成，但点击时为整块点击的元素，需要分为两种情况考虑：  
**使用 a 标签实现**  
使用了默认的点击效果，即使用了 a 标签实现外层框，读屏时子元素会被分别选中，但实际上单独读出每个子元素不能表达按钮整体的完整含义。因为，我们建议整块当作按钮处理，但一般无需添加 aria-label，让读屏软件直接按 DOM 顺序读出子元素的文字内容即可，例如：  
``` 
<a class="welcomeBonus_packet" href="javascript:;" role="button" @click="packetRedeem">
    <div class="welcomeBonus_packet_info">
        <div class="welcomeBonus_packet_info_title">主标题内容文字</div>
        <div class="welcomeBonus_packet_info_desc">描述文字</div>
    </div>
    <div class="welcomeBonus_packet_btn">
        <span>提示文字</span>
    </div>
</a>
```
会被读作"主标题内容文字 描述文字 提示文字 按钮"，视障人士会清楚这是整体点击的按钮，并且了解到其作用。如果部分内容不希望被读出来，精简朗读文案的时长，例如作用不大的辅助语句，可以单独添加 aria-hidden="true"。  
可点击元素点击后跳转页面通常采用 role="link" 声明，而点击后进行一些操作则通常采用 role="button" 声明，读屏的时候结尾分别为"链接"和"按钮"，但本场景下建议统一使用 role="button"，因为 role="link" 并不会让元素整块被识别，实际体验上，整体识别能带来更好的体验，而视障人士对于"链接"和"按钮"的理解包容度也比较高。  
**使用语义化标签实现**  
无需使用默认的点击效果，建议使用语义化的标签实现外层框，例如 section、aside，这样用户在使用“container 模式”进行读屏时，元素会直接被整体识别，而不会单独读出子元素。  
以 VoiceOver 为例，双指旋转可以调节焦点选择的模式，”container 模式“下焦点仅会被 section 这类外层容器捕捉。  
**小程序注意事项**  
小程序中目前仅支持 aria-role（相当于原生 Web 的 role）和 aria-label 两个属性，如果读屏时需要忽略某些元素，无法使用 aria-hidden 来声明，因此需要注意尽量让无需被读屏的元素不输出 DOM。  
小程序中没有语义标签，因此整块点击的元素只能加上 aria-role="button"。


参考:
[Web 可访问性与无障碍最佳实践](https://mp.weixin.qq.com/s/JByRPTi0jp08Cj6Kp5ekIg)
