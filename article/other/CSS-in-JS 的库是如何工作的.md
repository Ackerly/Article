# CSS-in-JS 的库是如何工作的
**调研**
社区中流行的 CSS-in-JS 库主要有两款：  
- emotion
- styled-components

## styled-components 核心能力  
**使用方式**
``` 
const Button = styled.div`
  background: palevioletred;
  border-radius: 3px;
  border: none;
  color: white;
`;

const TomatoButton = styled(Button)`
  background: tomato;
`;
```
通过在 React 组件中使用模板字符串编写 CSS 的形式实现一个自带样式的 React 组件  

**抽丝剥茧**  
clone styled-components 的 GitHub Repo到本地，安装好依赖后用 VS Code 打开，会发现 styled-components 是一个 monorepo，核心包与 Repo 名称相同  
直接从 src/index.ts 开始查看源码  
默认导出的 styled API 是从 src/constructors/styled.tsx 导出的  
src/constructors/styled.tsx 中的代码非常简单，去掉类型并精简后的代码如下：  
``` 
import createStyledComponent from '../models/StyledComponent';
// HTML 标签列表
import domElements from '../utils/domElements';
import constructWithOptions from './constructWithOptions';

// 构造基础的 styled 方法
const baseStyled = (tag) =>
  constructWithOptions(createStyledComponent, tag);

const styled = baseStyled;

// 实现 HTML 标签的快捷调用方式
domElements.forEach(domElement => {
  styled[domElement] = baseStyled(domElement);
});

export default styled;
```

``` 
const Button = styled.div`
  background: palevioletred;
  border-radius: 3px;
  border: none;
  color: white;
`;
```
实际上与以下代码完全一致：  
``` 
const Button = styled('div')`
  background: palevioletred;
  border-radius: 3px;
  border: none;
  color: white;
`;
```
styled API 为了方便使用封装了一个快捷调用方式，能够通过 styled[HTMLElement] 的方式快速创建基于 HTML 标签的组件  
继续向上溯源，找到与 styled API 有关的 baseStyled 的创建方式：  
``` 
const baseStyled = (tag) =>
  constructWithOptions(createStyledComponent, tag);
```  
找到 constructWithOptions 方法所在的 src/constructors/constructWithOptions.ts，去掉类型并精简后的代码如下：  
``` 
import css from './css';

export default function constructWithOptions(componentConstructor, tag, options) {
  const templateFunction = (initialStyles, ...interpolations) =>
    componentConstructor(tag, options, css(initialStyles, ...interpolations));

  return templateFunction;
}
```
baseStyled 是由 constructWithOptions 函数工厂创建并返回的。constructWithOptions 函数工厂的核心其实是 templateFunction 方法，其调用了组件的构造方法 componentConstructor，返回了一个携带样式的组件  

## 核心源码
constructWithOptions 函数工厂调用的组件构造方法 componentConstructor 是从外部传入的，而这个组件构造方法就是整个 styled-components 的核心所在  
baseStyled 是将 createStyledComponent 这个组件构造方法传入 componentConstructor 后返回的 templateFunction，templateFunction 的参数就是通过模板字符串编写的 CSS 样式，最终会传入组件构造方法 createStyledComponent 中。  
当用户使用 styled API 创建带有样式的组件时，本质上是在调用 createStyledComponent 这个组件构造函数  
从源码可以得知，createStyledComponent 的返回值是一个带有样式的组件 WrappedStyledComponent，在返回这个组件之前对其做了一些处理，大部分都是设置组件上的一些属性，可以在查看源码的时候暂时跳过。  
从返回值向上溯源，发现 WrappedStyledComponent 是使用 React.forwardRef 创建的一个组件，这个组件调用了 useStyledComponentImpl 这个 Hook 并返回了 Hook 的返回值。  
styled-components 最核心的部分：  
``` 
const generatedClassName = useInjectedStyle(
  componentStyle,
  isStatic,
  context,
  process.env.NODE_ENV !== 'production' ? forwardedComponent.warnTooManyClasses : undefined
);
```
CSS-in-JS 的库是通过在运行时解析模板字符串并且动态创建 <style></style> 标签将样式插入页面中实现的，从 useInjectedStyle Hook 的名称来看，其行为就是动态创建 <style></style> 标签并插入页面中。  
深入 useInjectedStyle，去掉类型并精简后的代码如下：  
``` 
function useInjectedStyle(componentStyle, isStatic, resolvedAttrs, warnTooManyClasses) {
  const styleSheet = useStyleSheet();
  const stylis = useStylis();

  const className = isStatic
    ? componentStyle.generateAndInjectStyles(EMPTY_OBJECT, styleSheet, stylis)
    : componentStyle.generateAndInjectStyles(resolvedAttrs, styleSheet, stylis);

  return className;
}
```
useInjectedStyle 调用了从参数传入的 componentStyle 上的 generateAndInjectStyles 方法，将样式传入其中，返回了样式对应的 className。  
进一步查看 componentStyle，是在 createStyledComponent 中被实例化并传入 useInjectedStyle 的：  
``` 
const componentStyle = new ComponentStyle(
  rules,
  styledComponentId,
  isTargetStyledComp ? styledComponentTarget.componentStyle : undefined
);
```
因此 componentStyle 上的 generateAndInjectStyles 实际上是 ComponentStyle 这个类的实例方法,核心是对模板字符串进行解析，并将类名 hash 后返回 className  

## solid-sc 实现
本质上就是将组件上携带的样式在运行时进行解析，给一个独一无二的 className，然后将其塞到 <style> 标签中。  
MVP 版本中只有两个函数：  
``` 
const createClassName = (rules: TemplateStringsArray) => {
  return () => {
    className++;
    const style = document.createElement('style');
    style.dataset.sc = '';
    style.textContent = `.sc-${className}{${rules[0]}}`.trim();
    document.head.appendChild(style);
    return `sc-${className}`;
  };
};

const createStyledComponent: StyledComponentFactories = (
  tag: keyof JSX.IntrinsicElements | Component
) => {
  return (rules: TemplateStringsArray) => {
    const StyledComponent: ParentComponent = (props) => {
      const className = createClassName(rules);
      const [local, others] = splitProps(props, ['children']);

      return (
        <Dynamic component={tag} class={className()} {...others}>
          {local.children}
        </Dynamic>
      );
    };

    return StyledComponent;
  };
};
```
两个函数的职责也非常简单，一个用于创建 <style> 标签并给样式生成一个唯一的 className，另一个用于创建经过样式包裹的动态组件  
还有非常多工作要做，比如：  
- 运行时解析不同方式传入的 CSS 规则
- 缓存相同组件的 className，相同组件复用样式
- 避免多次插入 <style> 标签，多个组件样式复用同一个
- &etc.

原文:  
[CSS-in-JS 的库是如何工作的](https://mp.weixin.qq.com/s/uK0WFipw2QgA_LNxenPI9A)
