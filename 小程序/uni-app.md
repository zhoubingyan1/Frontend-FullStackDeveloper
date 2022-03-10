###环境安装
npm install -g @vue/cli
 - vue必须是在3.x以上的版本

卸载1.x或者2.x的vue-cli版本。
安装3.x版本失败，


我将vue-cli更新到最新版也出现了这种问题，后来发现我的热更新失效是webpack版本的问题，@VUE中的一些包限制了webpack的版本，如下："webpack": ">=4 < 4.29"，但是有其他依赖安装了最新版的webpack，导致热更新失效，最后统一webpack版本就可以热更新了。

1.更新webpack	
sudo npm install webpack

2.卸载原先的vue
sduo npm uninstall vue-cli -g

3.重新安装vue
安装3.x版本以上 sudo npm install -g @vue/cli

###创建uni-app
vue create -p dcloudio/uni-preset-vue my-project

###运行并发布uni-app
npm run dev:%PLATFORM%
npm run build:%PLATFORM%

| 值 | 平台 | 
| ------------ | ------------ |
| h5|	H5|
|mp-alipay|支付宝小程序|
|mp-baidu|百度小程序|
|mp-weixin|微信小程序|
|mp-toutiao|头条小程序|
|mp-qq|qq 小程序|

- dev 模式编译出的各平台代码存放于根目录下的 /dist/dev/目录，打开各平台开发工具选择对应平台目录即可进行预览（h5 平台不会在此目录，存在于缓存中）；
- build 模式编译出的各平台代码存放于根目录下的 /dist/build/ 目录，发布时选择此目录进行发布； 
- dev 和 build 模式的区别：
    dev 模式有 SourceMap 可以方便的进行断点调试；
    build 模式会将代码进行压缩，体积更小更适合发布为正式版应用；
    进行 环境判断 时，dev 模式 process.env.NODE_ENV 的值为 development，build 模式 process.env.NODE_ENV 的值为 production。


cnpm i cross-env --save-dev    