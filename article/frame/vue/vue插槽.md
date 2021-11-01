# Vue插槽
``` 
<template id="panelTpl">
    <div class="demo">
        <div class="panel">
            <div class="title">Vue</div>
            <div class="content">
                <slot></slot>
            </div>
            <div class="footer">See more tips at https://vuejs.org/guide/deployment.html</div>
        </div>
    </div>
</template>
<script src="https://cdn.bootcss.com/vue/2.6.10/vue.js"></script>
<script type="text/javascript">
    Vue.component('panel',{
        template:'#panelTpl',
    });
    new Vue({
        el:'.demo'
    });


</script>
```
如上可以通过标签将组件中间的内容插入到自定义组件模板里面。
但是这里只有一个默认的slot，即模板里面的插入的内容就是自定义的组件标签里面的内容。那如果我还想动态的设置其他部分的内容怎么办呢，
其实vue中，在插入slot的时候，可以指定slot的名字的，表示这个插槽的名字。当组件在使用的时候可以指定那个插槽。
``` 
<!DOCTYPE html>
<html>
<head>
	<title>demo</title>
	<style type="text/css">
		.panel{
			max-width:300px;
			border: 1px solid red;
			padding:10px;
			word-break: break-all;
			margin-bottom: 30px;
		}
		.panel .title{
			font-size: 20px;
			border-bottom: 1px solid red;
		}
		.panel .footer{
			font-size: 14px;
			border-top: 1px solid red;
		}
		.panel>*{
			padding: 30px;
		}
	</style>
</head>
<body>
	<div class="demo">
		<panel>
			<div slot="title">
				title111111111111111111
			</div>
			<div slot="content">
				content111111111111111111
			</div>
			<div slot="footer">
				footer111111111111111111
			</div>
		</panel>
		<panel>
			<div slot="title">
				title2222
			</div>
			<div slot="content">
				content22222222
			</div>
		<!-- 	<div slot="footer">
				footer222
			</div> -->

		</panel>
		
	</div>


	<template id="panelTpl">
		<div class="demo">
			<div class="panel">
				<div class="title">
					<slot name="title">不会显示，因为上面有指定内容</slot>
				</div>
				<div class="content">
					<slot name="content"></slot>
					<!-- 这里的插槽，如果没有name，直接是对应使用组件时组件中的内容。但如果有了name，表示这里插入的插槽要对应这个name -->
				</div>
				<div class="footer">
					<slot name="footer">没有指定可以显示默认值，写在slot中</slot>
				</div>
			</div>
		</div>
	</template>
	<script src="https://cdn.bootcss.com/vue/2.6.10/vue.js"></script>
	<script type="text/javascript">
		Vue.component('panel',{
			template:'#panelTpl',
		});
		new Vue({
			el:'.demo'
		});


	</script>
</body>
</html>
```

参考：  
[了解Vue的插槽](https://juejin.cn/post/7025514359551950862)
