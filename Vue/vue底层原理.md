#### vue底层原理

> Vue是一个典型的MVVM框架，模型（Model）只是普通的JavaScript对象，修改它则视图（View）会自动更新。这种设计让状态管理变得非常简单而直观。那么Vue是如何把模型和视图建立起关联的呢？

- Vue实现这种数据双向绑定的效果，需要三大模块：

    Observer：能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者
    Compile：对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数
    Watcher：作为连接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图

##### Observer
- Observer的核心是通过Obeject.defineProperty()来监听数据的变动，这个函数内部可以定义setter和getter，每当数据发生变化，就会触发setter。这时候Observer就要通知订阅者，订阅者就是Watcher。

##### Watcher
- Watcher订阅者作为Observer和Compile之间通信的桥梁，主要做的事情是：
    在自身实例化时往属性订阅器(dep)里面添加自己
    自身必须有一个update()方法
    待属性变动dep.notice()通知时，能调用自身的update()方法，并触发Compile中绑定的回调


##### Compile
- Compile主要做的事情是解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图

具体实现:https://segmentfault.com/a/1190000006599500