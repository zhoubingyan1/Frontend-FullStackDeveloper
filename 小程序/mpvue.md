# mpvue框架开发小程序
created()不能使用,尽量使用onShow()

#ios音频不能播放
定位一个buton按钮在audio的标签上给个开关操作播放暂停

# 数据取回来先操作push，然后再赋值
```javascript
for(let i in res.data.data) {
    res.data.data[i].changeImage = false;
    }
that.arc_list = that.arc_list.concat(res.data.data);
```

#mpvue定义全局
```javascript
//在main.js使用
import Vue from 'vue'
import App from './App'
import request from './http/api'


Vue.config.productionTip = false
Vue.prototype.PATH = request.urls   //路由
App.mpType = 'app'

const app = new Vue(App)
app.$mount()


Vue.prototype.globalData = getApp().globalData

//在其他页面
比方说我需要存储用户一些基本信息可以这样

this.globalData.userInfo = {name:aaa}

不同页面获取方式一样

console.log(this.globalData.userInfo.name)

修改

this.globalData.userInfo.name = 'bbb'

https://www.cnblogs.com/Webzhoushifa/p/9563877.html 
```