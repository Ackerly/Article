# Selectors特性
## 伪类
**:has()**  
``` 
<div>
  <h1>H1</h1>
  <h2>H2</h2>
  <p>h1{margin: 0 0 0.25rem 0}</p>
</div>

// css
h1, h2 {
  margin: 0 0 1.0rem 0;
 }

 h1:has(+ h2) { // h1后面相邻兄弟元素是h2时
  margin: 0 0 0.25rem 0;
 }
```
**:is()**  
``` 
<div>
  <h1>H1</h1>
  <h2>H2</h2>
  <h3>H3</h3>
  <h4>H4</h4>
  <p>h1、h2和h3 {margin: 0 0 0.25rem 0}</p>
</div>

// css
 h1, h2, h3 {
  margin: 0 0 1.0rem 0;
 }

 :is(h1, h2, h3):has(+ :is(h2, h3, h4)) { // h1 or h2 or h3 后面相邻兄弟是 h2 or h3 or h4 时
  margin: 0 0 0.25rem 0;
 }
```
**:host**  
``` 
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"> 
    <title>:host</title>
  </head>
  <body>
    <my-component></my-component>
    <script>
      let shadow = document.querySelector("my-component").attachShadow({ mode: "open"})
      let styleEle = document.createElement("style")
      styleEle.textContent = `
        :host{
          display: block;
          margin: 20px;
          width: 500px;
          height: 300px;
          border: 3px solid green;
        }
        :host div {
          font-size: 30px;
          border: 2px solid blue;
        }
      `
      let headerEle = document.createElement("div");
      headerEle.innerText = "选取内部使用该部分 CSS 的 Shadow host 元素，其实也就是自定义标签元素。(注意：:host 选择器只在 Shadow DOM 中使用才有效果。)";
      shadow.appendChild(headerEle);
      shadow.appendChild(styleEle);

    </script>
  </body>
</html>
```
**:host()**  
``` 
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"> 
    <title>:host()伪类函数</title>
  </head>
  <body>
    <my-component class="my-component"></my-component>
    <my-component></my-component>
    <script>
      class MyComponent extends HTMLElement {
        constructor () {
          super();
          this.shadow = this.attachShadow({mode: "open"});
          let styleEle = document.createElement("style");
          styleEle.textContent = `
            :host(.my-component){
              display: block;
              margin: 20px;
              width: 300px;
              height: 200px;
              border: 3px solid green;
            }
            :host .component-header{
              border: 2px solid yellow;
              padding:10px;
              background-color: grey;
              font-size: 16px;
              font-weight: bold;
            }
          `;
          this.shadow.appendChild(styleEle);


          let headerEle = document.createElement("div");
          headerEle.className = "component-header";
          headerEle.innerText = ":host() 的作用是获取给定选择器的 Shadow Host。(注：:host() 的参数是必传的，否则选择器函数失效)";
          this.shadow.appendChild(headerEle);
        }
      }
  
      window.customElements.define("my-component", MyComponent);
  
    </script>
  </body>
</html>
```
**:host-textContent()**  
``` 
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"> 
    <title>:host-textContent()伪类函数</title>
  </head>
  <body>
    <div id="container">
      <my-component></my-component>
    </div>
    <my-component></my-component>
    <script>
      class MyComponent extends HTMLElement {
        constructor () {
          super();
          this.shadow = this.attachShadow({mode: "open"});
          let styleEle = document.createElement("style");
          styleEle.textContent = `
            :host-context(#container){
              display: block;
              margin: 20px;
              width: 300px;
              height: 200px;
              border: 3px solid green;
            }
            :host .component-header{
              border: 2px solid yellow;
              padding:10px;
              background-color: #4caf50;
              font-size: 16px;
              font-weight: bold;
            }
          `;
          this.shadow.appendChild(styleEle);

          let headerEle = document.createElement("div");
          headerEle.className = "component-header";
          headerEle.innerText = ":host-context() 用来选择特定祖先内部的自定义元素，祖先元素选择器通过参数传入。(注：:host-context() 的参数是必传的，否则选择器函数失效)";
          this.shadow.appendChild(headerEle);
        }
      }
    
      window.customElements.define("my-component", MyComponent);
    
    </script>
  </body>
</html>
```
**:in-range & :out-of-range**  
``` 
<style>
  input {
    width: 180px;
    height: 32px;
    color: white;
    font-weight: bold;
    font-size: 20px;
  }
  input:in-range {
    background: yellowgreen;
  }
  input:out-of-range {
    background-color: brown;
  }
</style>
<p>:in-range 输入的值在指定区间内时，设置指定样式</p>
<p>:out-of-range 输入的值在指定区间外时，设置指定样式</p>
<input type="number" min="5" max="10" value="7" />
<input type="number" min="5" max="10" value="3" />
```
**:root**  
``` 
<style>
  :root {
    --green: #9acd32;
    --font-size: 14px;
    --font-family-sans-serif: -apple-system, BlinkMacSystemFont;
  }
  body {
    color: var(--green);
    font-size: var(--font-size);
    font-family: var(--font-family-sans-serif);
  }
</style>

<div>
  :root中声明相当于全局属性，页面可以使用var()来引用
</div>
```
## 伪元素
**::first-letter**  
``` 
<style>
  p::first-letter {
    font-size: 1.5rem;
    font-weight: bold;
    color: brown;
  }
</style>

<p>Scientists exploring the depths of Monterey Bay unexpectedly encountered a rare and unique species of dragonfish. This species is the rarest of its species.</p>

<p>中文字体时.</p>
```

::placeholder 设置placeholder的样式  
::selection 鼠标选中文本的样式

原文:  
[CSS中你可能不知道的Selectors特性](https://mp.weixin.qq.com/s/Kb1K_IJ4GSUSwINSs9NIAg)
