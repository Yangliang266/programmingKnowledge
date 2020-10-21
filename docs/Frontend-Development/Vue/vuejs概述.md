### 高级功能

1. 逻辑视图
2. 可复用组件
3. 前端路由
4. 状态管理
5. 虚拟路由(DOM)

### mvvc模式 

**MVVM是Model-View-ViewModel的简写。即模型-视图-视图模型。**

1. 【模型】指的是后端传递的数据 - js的数据

2. 【视图】指的是所看到的页面 - 页面视图

3. 【视图模型】mvvm模式的核心，它是连接view和model的桥梁 - vue实例化的对象

4. 它有两个方向：
   1. 一是将【模型】转化成【视图】，即将后端传递的数据转化成所看到的页面。实现的方式是：数据绑定。
   2. 二是将【视图】转化成【模型】，即将所看到的页面转化成后端的数据。
   3. 实现的方式是：DOM 事件监听。这两个方向都实现的，我们称之为数据的双向绑定。

5. 总结：在MVVM的框架下视图和模型是不能直接通信的。它们通过ViewModel来通信，ViewModel通常要实现一个observer观察者，当数据发生变化，ViewModel能够监听到数据的这种变化，然后通知到对应的视图做自动更新，而当用户操作视图，ViewModel也能监听到视图的变化，然后通知数据做改动，这实际上就实现了数据的双向绑定。并且MVVM中的View 和 ViewModel可以互相通信。



​		  MVVM流程图如下：

![img](https://ss2.baidu.com/6ONYsjip0QIZ8tyhnq/it/u=32561255,2826043542&fm=173&app=25&f=JPEG?w=500&h=100&s=C8F78852C4B2FE207E66C9D20200D0AA)



### vue特性



1. 渐进式javascript框架

   1. 阶段性使用vue，不必一开始使用所有

   2. vue可以作为应用嵌入某个服务

   3. 将网页分割成可复用的组件，渲染到网页相应的地方，每个组件都有对应的html,css,js文件

   4. 单文件组件

      > 步步递进，，对vue的使用可大可小

2. 响应式系统

   1. 概述：数据变更，vue会更新网页中关联的地方

   2. 特点：保持状态和视图的同步

   3. 原理

   4. 主流框架实现双向绑定（响应式）的做法

      1. 脏值检查 - angular

      2. 观察者订阅者模式 - vue

         1. vue-Observer 数据监听器，把一个普通的 JavaScript 对象传给 Vue 实例的 data 选项。

         2. Vue 将遍历此对象所有的属性，并使用Object.defineProperty()方法把这些属性全部转成setter、getter方法

         3. 当data中的某个属性被访问时，则会调用getter方法，当data中的属性被改变时，则会调用setter方法。

         4. Compile指令解析器，它的作用对每个元素节点的指令进行解析，替换模板数据，并绑定对应的更新函数，初始化相应的订阅。

         5. Watcher 订阅者，作为连接 Observer 和 Compile 的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数。

         6. Dep 消息订阅器，内部维护了一个数组，用来收集订阅者（Watcher），数据变动触发notify 函数，再调用订阅者的 update 方法。

         7. 执行流程如下：

            ![img](https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=1581589677,2197583542&fm=173&app=25&f=JPEG?w=640&h=342&s=5926347301CA614B4E65C0CA0000E0B3)

            8.流程解释 - 当执行 new Vue() 时，Vue 就进入了初始化阶段。

            1. 一方面Vue 会遍历 data 选项中的属性，并用 Object.defineProperty 将它们转为 getter/setter，实现数据变化监听功能；
            2. 另一方面，Vue 的指令编译器Compile 对元素节点的指令进行解析，初始化视图，并订阅Watcher 来更新视图， 此时Watcher 会将自己添加到消息订阅器中(Dep),初始化完毕。
            3. 当数据发生变化时，Observer 中的 setter 方法被触发，setter 会立即调用Dep.notify()，Dep 开始遍历所有的订阅者，并调用订阅者的 update 方法，订阅者收到通知后对视图进行相应的更新。
            4. 因为VUE使用Object.defineProperty方法来做数据绑定，而这个方法又无法通过兼容性处理，所以Vue 不支持 IE8 以及更低版本浏览器。
            5. 另外，查看vue原代码，发现在vue初始化实例时， 有一个proxy代理方法，它的作用就是遍历data中的属性，把它代理到vm的实例上，这也就是我们可以这样调用属性：vm.aaa等于vm.data.aaa。