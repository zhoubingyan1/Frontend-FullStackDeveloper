vue 性能优化总结
[toc]
### v-if与v-show 
- 运行时很少改变条件时使用v-if
- 频繁切换的时候用v-show

### computed与watch
- 依赖数据，并可基于依赖计算返回值时使用computed （computed具备缓存）
- 观察数据变化进行回调操作使用watch 

### v-for
- v-for 为item添加key
- v-for 避免同时使用v-if（v-for比v-if优先级高，每一次都要遍历整个数组，会影响速度，尤其是当之需要渲染很小一部分的时候，必要情况换成computed属性）

### 长列表优化
对于无限长列表来说，性能优化主要方法是保持仅渲染可视化部分。
- 使用 object.freeze 来冻结不需要监听的列表数据
或者使用[vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller) 组件

### 事件销毁
- 如果手动添加了addEventListener,记得在组件销毁时手动移除事件的监听

### 图片懒加载

### 路由懒加载
通过函数 + import实现 const Page404 = () => import(/* webpackChunkName: "error" */'@views/errorPage/404');

### 插件按需引入

### SSR
服务端渲染SSR，vue使用nuxt.js，

### 函数型组件

```shell
<template>
    <div>
        <header>
          <h1>I'm a template!</h1>
        </header>
        <p v-if="props.materialtype">{{ message }}</p>
        <p v-else>No message.</p>
    </div>
</template>
<script>
export default {
  name: 'App',
  data:{
      message:''
  },
  props: {
    materialtype:{
      type: String,
      value:''
    },
}
</script>
```