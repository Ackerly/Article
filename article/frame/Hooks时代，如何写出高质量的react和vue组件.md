# Hooks时代，如何写出高质量的react和vue组件
**概述**
一个组件内部的所有代码——无论vue还是react——都可以抽象成以下几个部分:  
1. 组件视图，组件中用来描述视觉效果的部分，如css和html、react的jsx或者vue的template代码
2. 组件相关逻辑，如组件生命周期，按钮交互，事件等
3. 业务相关逻辑，如登录注册，获取用户信息，获取商品列表等与组件无关的业务抽象

单独拆分这三块并不难，难的是一个组件可能写得特别复杂，里面可能包含了多个视图，每个视图相互之间又有交互；同时又可能包含多个业务逻辑，多个业务的函数和变量杂乱无章地随意放置，导致后续维护的时候要在代码之间反复横跳  

**组件什么时候拆？怎么拆**  
一个常见的误区是，只有需要复用的时候才去拆分组件，这种看法显然过于片面了。你可以思考一下，自己是如何抽象一个函数的，你只会在代码需要复用的时候才抽出一个函数吗？显然不是。因为函数不仅有代码复用的功能，还具有一定的描述性质以及代码封闭性。这种特性使得我们看到一个函数的时候，不必关注代码细节，就能大概知道这部分代码是干啥的。  
还可以再用函数将一部分函数组合起来，形成更高层级的抽象。按国内流行的说法，高层级的抽象被称为粗粒度，低层级的抽象被称为细粒度，不同粗细粒度的抽象可以称它们为不同的抽象层级。并且一个理想的函数内部，一般只会包含同一抽象层级的代码。  
组件的拆分也可以遵循同样的道理。可以按照当前的结构或者功能、业务，将组件拆分为功能清晰且单一、与外部耦合程度低的组件(即所谓高内聚，低耦合)。如果一个组件里面干了太多事，或者依赖的外部状态太多，那么就不是一个容易维护的组件了。  
为了保持组件功能单一，是不是要将组件拆分得特别细才可以呢？事实并非如此。因为上面说过，抽象是有粗细粒度之分的，也许一个组件从较细的粒度来讲功能并不单一，但是从较粗的粒度来说，可能他们的功能就是单一的了。例如登录和注册是两个不同的功能，但是你从更高层级的抽象来看，它们都属于用户模块的一部分。  
是否要拆分组件，最关键还是得看复杂度。如果一个页面特别简单，那么不进行拆分也是可以，有时候拆分得过于细可能反而不利于维护。  
拆分组件的时候可以参考下面几个原则：  
- 拆分的组件要保持功能单一。即组件内部代码的代码都只跟这个功能相关
- 组件要保持较低的耦合度，不要与组件外部产生过多的交互。如组件内部不要依赖过多的外部变量，父子组件的交互不要搞得太复杂等等
- 用组件名准确描述这个组件的功能。就像函数那样，可以让人不用关心组件细节，就大概知道这个组件是干嘛的。如果起名比较困难，考虑下是不是这个组件的功能并不单一

**如何组织拆分出的组件文件**  
拆分出来的组件应该放在哪里呢？一个常见的错误做法是一股脑放在一个名为components文件夹里，最后搞得这个文件夹特别臃肿。建议相关联的代码最好尽量聚合在一起。  
为了让相关联的代码聚合到一起，我们可以把页面搞成文件夹的形式，在文件夹内部存放与当前文件相关的组成部分，并将表示页面的组件命名为index放在文件夹下。再在该文件夹下创建components目录，将组成页面的其他组件放在里面  
如果一个页面的某个组成部分很复杂，内部还需要拆分成更细的多个组件，那么就把这个组成部分也做成文件夹，将拆分出的组件放在这个文件夹下  
最后就是组件复用的问题。如果一个组件被多个地方复用，就把它单独提取出来，放到需要复用它的组件们共同的抽象层级上。 如下：  
1. 如果只是被页面内的组件复用，就放到页面文件夹下
2. 如果只是在当前业务场景下的不同页面复用，就放到当前业务模块的文件夹下
3. 如果可以在不同业务场景间通用，就放到最顶层的公共文件夹，或者考虑做成组件库

常用的一种页面级别的文件的组织方式：  
``` 
homePage // 存放当前页面的文件夹
    |-- components // 存放当前页面组件的文件夹
        |-- componentA // 存放当前页面的组成部分A的文件夹
            |-- index.(vue|tsx) // 组件A
            |-- AChild1.(vue|tsx) // 组件a的组成部分1
            |-- AChild2.(vue|tsx) // 组件a的组成部分2
            |-- ACommon.(vue|tsx) // 只在componentA内部复用的组件
        |-- ComponentB.(vue|tsx) // 当前页面的组成部分B
        |-- Common.(vue|tsx) // 组件A和组件B里复用的组件
    |-- index.(vue|tsx) // 当前页面
```
实际上这种组织方式，在抽象意义上并不完美，因为通用组件和页面组成部分的组件并没有区分开来。但是一般来说，一个页面也不会抽出太多组件，为了方便放到一起也不会有太大问题。但是如果你的页面实在复杂，那么再创建一个名为common的文件夹也未尝不可  
**如何用hooks抽离组件逻辑**  
在hooks出现之前，曾流行过一个设计模式，这个模式将组件分为无状态组件和有状态组件（也称为展示组件和容器组件），前者负责控制视觉，后者负责传递数据和处理逻辑。但有了hooks之后，我们完全可以将容器组件中的代码放进hooks里面。后者不仅更容易维护，而且也更方便把业务逻辑与一般组件区分开来  
在抽离hooks的时候，不仅应该沿用一般函数的抽象思维，如功能单一，耦合度低等等，还应该注意组件中的逻辑可分为两种：组件交互逻辑与业务逻辑。如何把视图、交互逻辑和业务逻辑区分开来，是衡量一个组件质量的重要标准。  
以一个用户模块为例。一个包含查询用户信息，修改用户信息，修改密码等功能的hooks可以这样写：  
``` 
// 用户模块hook
const useUser = () => {
    // react版本的用户状态
    const user = useState({});
    // vue版本的用户状态
    const userInfo = ref({});
    
    // 获取用户状态
    const getUserInfo = () => {}
    // 修改用户状态
    const changeUserInfo = () => {};
    // 检查两次输入的密码是否相同
    const checkRepeatPass = (oldPass，newPass) => {}
    // 修改密码
    const changePassword = () => {};
    
    return {
        userInfo,
        getUserInfo,
        changeUserInfo,
        checkRepeatPass,
        changePassword,
    }
}
```
交互逻辑的hook可以这么写  
``` 
// 用户模块交互逻辑hooks
const useUserControl = () => {
    // 组合用户hook
    const { userInfo, getUserInfo, changeUserInfo, checkRepeatPass, changePassword } = useUser();
    // 数据查询loading状态
    const loading = ref(false);
    // 错误提示弹窗的状态
    const errorModalState = reactive({
        visible: false, // 弹窗显示/隐藏
        errorText: '',  // 弹窗文案
    });
    
    // 初始化数据
    const initData = () => {
        getUserInfo();
    }
    // 修改密码表单提交
    const onChangePassword = ({ oldPass, newPass ) => {
        // 判断两次密码是否一致
        if (checkRepeatPass(oldPass, newPass)) {
            changePassword();
        } else {
            errorModalState.visible = true;
            errorModalState.text = '两次输入的密码不一致，请修改'
        }
    };
    return {
        // 用户数据
        userInfo,
        // 初始化数据
        initData: getUserInfo,
        // 修改密码
        onChangePassword,
        // 修改用户信息
        onChangeUserInfo: changeUserInfo,
    }
}
```
然后只要在组件里面引入交互逻辑的hook即可   
vue版本：  
``` 
<template>
    <!-- 视图部分省略，在对应btn处引用onChangePassword和onChangeUserInfo即可 -->
</template>
<script setup>
import useUserControl from './useUserControl';
import { onMounted } from 'vue';

const { userInfo, initData, onChangePassword, onChangeUserInfo } = useUserControl();
onMounted(initData);
<script>
```
react版本：  
``` 
import useUserControl from './useUserControl';
import { useEffect } from 'react';

const UserModule = () => {
    const { userInfo, initData, onChangePassword, onChangeUserInfo } = useUserControl();
    useEffect(initData, []);
    return (
        // 视图部分省略，在对应btn处引用onChangePassword和onChangeUserInfo即可
    )
}
```
拆分出的三个文件放在组件同级目录下即可；如果拆出的hooks较多，可以单独开辟一个hooks文件夹。如果有可以复用的hooks，参考组件拆分里面分享的方法，放到需要复用它的组件们共同的抽象层级上即可。  
可以看到抽离出hooks逻辑后，组件变得十分简单、容易理解，我们也实现了各个部分的分离。不过这里还有一个问题，那就是上面的业务场景实在太过简单，有必要拆分得这么细，搞出三个文件这么复杂吗？  
针对逻辑并不复杂的组件，我个人觉得和组件放到一起也未尝不可。为了简便，我们可以只把业务逻辑封装成hooks，而组件的交互逻辑就直接放在组件里面。如下：  
``` 
<template>
    <!-- 视图部分省略，在对应btn处引用changePassword和changeUserInfo即可 -->
</template>
<script setup>
import { onMounted } from 'vue';
// 用户模块hook
const useUser = () => { 
    // 代码省略
}

const { userInfo, getUserInfo, changeUserInfo, checkRepeatPass, changePassword } = useUser();
// 数据查询loading状态
const loading = ref(false);
// 错误提示弹窗的状态
const errorModalState = reactive({
    visible: false, // 弹窗显示/隐藏
    errorText: '', // 弹窗文案
});

// 初始化数据
const initData = () => { getUserInfo(); }
// 修改密码表单提交
const onChangePassword = ({ oldPass, newPass ) => {};
    
onMounted(initData);
<script>
```
但是如果逻辑比较复杂，或者一个组件里面包含多个复杂业务或者复杂交互，需要抽离出多个hooks的情况，还是单独抽出一个个文件比较好。总而言之，依据代码复杂度，选择相对更容易理解的写法。  
也许单独一个组件，你并不能体会出hooks写法的优越性。但当你封装出更多的hooks之后，你会逐渐发现这样写的好处。正因为不同的业务和功能被封装在一个个hooks里面，彼此互不干扰，业务才能更容易区分和理解。对于项目的可维护性和可读性提升是非常之大的。  
**全局状态的管理**  
现在的前端项目还有一个较为常见的误区，那就是全局状态管理库（即redux、vuex等）的滥用。依据抽象层级的思维，实际上很多项目并不需要放较多的状态到全局，这种情况利用react和vue自身的状态管理就足够了  
如果非要用状态管理库，也要警惕放较多状态和函数到全局。一个状态是否要放到全局，我一般有两个判断标准：
1. 状态是否在多个页面间共享
2. 跳转页面后又返回该页面，是否需要还原跳转之前的状态（仅对react而言，vue有keep-alive）

而全局状态管理库中的函数，则只放置与全局状态有关的逻辑。除此之外的状态，一律交由react和vue组件本身进行管理  

原文: 
[Hooks时代，如何写出高质量的react和vue组件](https://mp.weixin.qq.com/s/_A3CmH9awg_2z3mCu8odQQ)
