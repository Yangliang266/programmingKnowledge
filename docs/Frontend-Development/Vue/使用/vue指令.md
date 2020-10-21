### 模板语法



#### 实例

1. 数据与方法
   1. data 中存在的属性是响应式的
   2. Object.freeze() 阻止修改现有属性，响应式无法在追踪变化
2. 实例生命周期钩子
3. 实例化多个对象
   1. new 多个vue对象
   2. 改变其他实例的属性
      1. 内部 - 通过实例vue id，在methods，computed中更改
      2. 外部 - 通过实例vue id，更改对应的属性



#### 模板语法

1. 插值

   1. >  文本{{}}

   2. >  v-html 输出真正的html

   3. javascript表达式

2. 指令

   1. v-if 根据真假插入或移除元素
   2. 参数
      1. v-bind 响应式的更新html，以冒号表示：href，url，src
   3. 缩写
   4. 动态参数



### 计算属性侦听器过滤器



#### 计算属性

1. 当其依赖的属性发生变化时，这个属性的值也会发生改变
2. computed

#### 计算属性与方法

1. 区别 
   1. 计算属性 ：基于相关的依赖进行缓存，相关依赖发生改变才会重新求值，如果未改变则，不会执行
   2. 方法：不管属性是否发生变化都会执行



#### 侦听属性

1. 回调值为函数方法

2. 回调值为对象

   ```vue
   <html>
       <head>
           <meta charset="utf-8">
           <title></title>
           <script src="vue.js" type="text/javascript"></script>
       </head>
       <body>
   		<div id="app">
   			输入: <input type="text" v-model="message" />
   			user: <input type="text" v-model="User.name" />
   		</div>
       </body>
   
       <script type="text/javascript">
           var app = new Vue({
               el: '#app',
               data: {
   				message: '',
   				User: {
   					name: "yangl"
   				}
   			},
   			watch:{
   				message:'way',
   				// 'User.name': 'text',
   				// 'User.name': {
   				// 	handler: function(newvalue, oldvalue) {
   				// 		console.log("new:" + newvalue + "_____old:" + oldvalue);
   				// 	}
   				// },
   				change: {
   					handler: function(newvalue, oldvalue) {
   						console.log("new:" + newvalue + "_____old:" + oldvalue);
   					}
   				}
   			},
   			
   			// methods:{
   			// 	way:function(newvalue,oldvalue) {
   			// 		console.log("new:" + newvalue + "_____old:" + oldvalue);
   			// 	},
   			// 	userway:function(newvalue,oldvalue) {
   			// 		console.log("new:" + newvalue + "_____old:" + oldvalue);
   			// 	},
   			// 	text: function(newvalue,oldvalue) {
   			// 		console.log("new:" + newvalue + "_____old:" + oldvalue);
   			// 	}
   			// },
   			computed:{
   				change: function() {
   					return this.User.name;
   				}
   			}
           })
   		
   		
       </script>
   </html>
   ```

   

### 过滤器



### 内置指令



#### 基本指令

1. v-cloak
2. v-once
3. v-text与v-html
   1. 替换整个内容
4. v-bind
5. v-on



#### 条件渲染

1. v-if
   1. template使用
2. v-else
3. v-else-if
4. v-show
   1. 不支持template，v-else
5. key 管理复用元素
6. v-show与v-if的区别
   1. show 分为block和none隐藏，保留在dom中
   2. if 分为创建和摧毁



#### 列表渲染

1. v-for

2. 数组更新检测

   1. 变异方法
      1. push/pop/shift /unshift/splice/sort/reverse
      2. 显示排序或过滤的副本
   2. 非变异方法
      1. concat/slice/map/filter 
      2. 不会改变原始数组

3. 对象变更

   1. vue不能检测对象属性的添加或者删除
   2. 添加响应式属性

   ```VUE
   <!DOCTYPE html>
   <html>
   	<head>
   		<meta charset="utf-8">
   		<title></title>
   		<script src="vue.js" type="text/javascript"></script>
   	</head>
   	<body>
   		<div id="app">
   			<template v-if="loginType === 'username'">
   				<label>姓名：</label>
   				<input placeholder="输入用户名" key="val1"/>
   			</template>
   			<template v-else>
   				<label>邮箱：</label>
   				<input placeholder="输入邮箱" key="val2"/>
   			</template>
   			<button @click="change">change</button>
   			
   			<ul>
   				<li>-----------</li>
   				<!-- 遍历数组 -->
   				<li v-for="item of items">
   					{{item}}
   				</li>
   				<li>-----------</li>
   				<li v-for="(item,index) of items">
   					{{index}} - {{item}}
   				</li>
   				<li>-----------</li>
   				<!-- 遍历对象 -->
   				<li v-for="user of users">
   					{{user}}
   				</li>
   				<li>-----------</li>
   				<li v-for="(value,name,index) of users">
   					{{index}} - {{name}} : {{value}}
   				</li>
   			</ul>
   		</div>
   		
   	</body>
   	
   	<script type="text/javascript">
   		var app = new Vue({
   			el: '#app',
   			data: {
   				loginType: 'username',
   				items:['ma','hua','teng'],
   				users: {
   					name: 'yang',
   					age: '18',
   					adress: 'nantong'
   				}
   			},
   			methods:{
   				change: function() {
   					return this.loginType == 'username' ? this.loginType = 'email' : this.loginType = 'username'
   				}
   			}
   		})
   		
   		app.items.push('jj');
   		app.$set(app.users,'sex','nan');
   		
   		//添加对象新属性用两个对象的属性创建一个新的对象
   		app.userProfile = Object.assign(app.users, {
   			email: '26789',
   			phone: '123'
   		})
   		
   		
   	</script>
   </html>
   
   ```

   