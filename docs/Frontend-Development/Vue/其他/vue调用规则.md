### vue-cli 文件执行顺序

1. index.html

   ```vue
   <!DOCTYPE html>
   <html>
     <head>
       <meta charset="utf-8">
       <meta name="viewport" content="width=device-width,initial-scale=1.0">
       <title>alanmall-front</title>
     </head>
     <body>
       <div id="app"></div>
       <!-- built files will be auto injected -->
     </body>
   </html>
   ```

   

2. main.js

   ```js
   import Vue from 'vue'
   import App from './App'
   import store from './store/'
   import router from './router'
   
   Vue.config.productionTip = false
   
   new Vue({
     // 将Vue这个实例挂载在id="app"的DOM元素上
     el: '#app',
     store,
     router,
     // h指vm.$createElement,在vm._render阶段
     // render: h => h(App)
     components: { App },
     // 假如render和template属性都不存在，id="app"的div中的内容会被提取出来当模板，我们上面写的div中没有内容，
     // 而且，因为有template属性存在，所以用app.vue的内容 替换 挂载的元素，也就是替换id="app"的div。
     // 挂载元素没有内容，也没有插槽slot，所以直接就替换了。
       
     // 如果存在render选项，则 Vue 构造函数不会从 template 选项或通过 el 选项指定的挂载元素中提取出的 HTML 模板编译渲		染函数。
     // 如果 render 函数和 template property 都不存在，挂载 DOM 元素的 HTML 会被提取出来用作模板，此时，必须使用 	  Runtime + Compiler 构建的 Vue 库。
     // 只存在template会被编译成render
     template: '<App>'
   })
   
   ```

   

   

3. app.vue

   ```vue
   <template>
     // 替换了index.html的div id = “app”
     <div id="app">
       // <router-view>中的内容是路由加载的，加载内容在router/index.js中配置，我们默认让它加载index.vue
       <router-view class="main"></router-view>
     </div>
   </template>
   
   <style>
   #app {
     height: 100%;
   }
   
   .main {
       background: #ededed;;
     }
   </style>
   
   ```

   

4. index.js

   ```js
   import Vue from 'vue'
   import Router from 'vue-router'
   // import HelloWorld from '@/components/HelloWorld'
   import Index from '@/page/index'
   import Login from '@/page/login/login'
   
   Vue.use(Router)
   
   export default new Router({
     routes: [
       {
         path: '/',
         name: 'index',
         component: Index
       },
       {
         path: '/login',
         name: 'login',
         component: Login
       }
     ]
   })
   
   ```

   

