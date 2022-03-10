###vue组件间的传值
1.provide / inject
这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。
2.Vue.observable
让一个对象可响应。Vue 内部会用它来处理 data 函数返回的对象。
返回的对象可以直接用于渲染函数和计算属性内，并且会在发生改变时触发相应的更新。也可以作为最小化的跨组件状态存储器，用于简单的场景：
const state = Vue.observable({ count: 0 })

const Demo = {
  render(h) {
    return h('button', {
      on: { click: () => { state.count++ }}
    }, `count is: ${state.count}`)
  }
}
3.$attrs
包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。
4.$listeners
包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。
5.props
6.$emit
7.eventbus
8.vuex
9.$parent / $children / ref



###npm脚本，捆绑时将package.json复制到dist
在package.json里加
"build:h5": "cross-env NODE_ENV=production UNI_PLATFORM=h5 vue-cli-servigitce uni-build && cp baidu_verify_cVeaY8ydSK.html dist/build/h5",
```shell
{
  "name": "zhenyouhaofang-uniapp",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "serve": "npm run dev:h5",
    "build": "npm run build:h5",
    "info": "node node_modules/@dcloudio/vue-cli-plugin-uni/commands/info.js",
    "dev:h5": "cross-env NODE_ENV=development UNI_PLATFORM=h5 vue-cli-service uni-serve",
    "beta:h5": "cross-env NODE_ENV=development UNI_PLATFORM=h5 vue-cli-service build --no-clean --mode build",
    "dev:mp-qq": "cross-env NODE_ENV=development UNI_PLATFORM=mp-qq vue-cli-service uni-build --watch",
    "dev:mp-weixin": "cross-env NODE_ENV=development UNI_PLATFORM=mp-weixin vue-cli-service uni-build --watch",
    "dev:mp-baidu": "cross-env NODE_ENV=development UNI_PLATFORM=mp-baidu vue-cli-service uni-build --watch",
    "dev:mp-alipay": "cross-env NODE_ENV=development UNI_PLATFORM=mp-alipay vue-cli-service uni-build --watch",
    "dev:mp-toutiao": "cross-env NODE_ENV=development UNI_PLATFORM=mp-toutiao vue-cli-service uni-build --watch",
    "pre:h5": "cross-env NODE_ENV=release UNI_PLATFORM=h5 vue-cli-service uni-serve",
    "alpha:h5": "cross-env NODE_ENV=release UNI_PLATFORM=h5 vue-cli-service build --no-clean --mode build",
    "pre:mp-qq": "cross-env NODE_ENV=release UNI_PLATFORM=mp-qq vue-cli-service uni-build --watch",
    "pre:mp-weixin": "cross-env NODE_ENV=release UNI_PLATFORM=mp-weixin vue-cli-service uni-build --watch",
    "pre:mp-baidu": "cross-env NODE_ENV=release UNI_PLATFORM=mp-baidu vue-cli-service uni-build --watch",
    "pre:mp-alipay": "cross-env NODE_ENV=release UNI_PLATFORM=mp-alipay vue-cli-service uni-build --watch",
    "pre:mp-toutiao": "cross-env NODE_ENV=release UNI_PLATFORM=mp-toutiao vue-cli-service uni-build --watch",
    "build:app-plus": "cross-env NODE_ENV=production UNI_PLATFORM=app-plus vue-cli-service uni-build",
    "build:mp-qq": "cross-env NODE_ENV=production UNI_PLATFORM=mp-qq vue-cli-service uni-build",
    "build:mp-weixin": "cross-env NODE_ENV=production UNI_PLATFORM=mp-weixin vue-cli-service uni-build",
    "build:mp-baidu": "cross-env NODE_ENV=production UNI_PLATFORM=mp-baidu vue-cli-service uni-build",
    "build:mp-alipay": "cross-env NODE_ENV=production UNI_PLATFORM=mp-alipay vue-cli-service uni-build",
    "build:mp-toutiao": "cross-env NODE_ENV=production UNI_PLATFORM=mp-toutiao vue-cli-service uni-build",
    "dev:custom": "cross-env NODE_ENV=development uniapp-cli custom",
    "build:custom": "cross-env NODE_ENV=production uniapp-cli custom",
    "dev:app-plus": "cross-env NODE_ENV=development UNI_PLATFORM=app-plus vue-cli-service uni-build --watch",
    "build:h5": "cross-env NODE_ENV=production UNI_PLATFORM=h5 vue-cli-servigitce uni-build && cp baidu_verify_cVeaY8ydSK.html dist/build/h5",
    "test:h5": "vue-cli-service build --no-clean --mode build",
    "test1:h5": "cross-env NODE_ENV=preonline UNI_PLATFORM=h5 vue-cli-service uni-build --watch"
  },
  "dependencies": {
    "@dcloudio/uni-app-plus": "^2.0.0-24220191115006",
    "@dcloudio/uni-h5": "^2.0.0-24220191115006",
    "@dcloudio/uni-helper-json": "*",
    "@dcloudio/uni-mp-alipay": "^2.0.0-24220191115006",
    "@dcloudio/uni-mp-baidu": "^2.0.0-24220191115006",
    "@dcloudio/uni-mp-qq": "^2.0.0-24220191115006",
    "@dcloudio/uni-mp-toutiao": "^2.0.0-24220191115006",
    "@dcloudio/uni-mp-weixin": "^2.0.0-24220191115006",
    "@dcloudio/uni-stat": "^2.0.0-24220191115006",
    "flyio": "^0.6.2",
    "regenerator-runtime": "^0.12.1",
    "vue": "^2.6.10",
    "vuex": "^3.1.2"
  },
  "devDependencies": {
    "@dcloudio/uni-cli-shared": "^2.0.0-24220191115006",
    "@dcloudio/uni-template-compiler": "^2.0.0-24220191115006",
    "@dcloudio/vue-cli-plugin-hbuilderx": "^2.0.0-24220191115006",
    "@dcloudio/vue-cli-plugin-uni": "^2.0.0-24220191115006",
    "@dcloudio/vue-cli-plugin-uni-optimize": "^2.0.0-24220191115006",
    "@dcloudio/webpack-uni-mp-loader": "^2.0.0-24220191115006",
    "@dcloudio/webpack-uni-pages-loader": "^2.0.0-24220191115006",
    "@types/html5plus": "*",
    "@types/uni-app": "*",
    "@vue/cli-plugin-babel": "3.5.1",
    "@vue/cli-service": "^3.0.0",
    "babel-plugin-import": "^1.11.0",
    "mini-types": "*",
    "miniprogram-api-typings": "^2.8.0-2",
    "node-sass": "^4.13.0",
    "postcss-comment": "^2.0.0",
    "sass-loader": "^8.0.0",
    "vue-template-compiler": "^2.6.10"
  },
  "browserslist": [
    "Android >= 4",
    "ios >= 8"
  ],
  "uni-app": {
    "scripts": {
      "h5-weixin": {
        "title": "微信服务号",
        "BROWSER": "Chrome",
        "env": {
          "UNI_PLATFORM": "h5"
        },
        "define": {
          "H5-WEIXIN": true
        }
      }
    }
  }
}
```

