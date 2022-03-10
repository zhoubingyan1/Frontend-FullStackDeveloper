第一次接触VUE，今天为了跑通公司项目，着实费了不少劲。

主要起因是命令：

npm run dev 
在编译过程中报错：Syntax Error: Unexpected token (4:19)



 

 原来是import这儿报错了，这就需要babel的插件了，vue-router官网上有一段提示：
如果您使用的是 Babel，你将需要添加 syntax-dynamic-import 插件，才能使 Babel 可以正确地解析语法。

运行命令：

npm install babel-plugin-syntax-dynamic-import --save-dev
然后修改webpack的js的loader部分：

 

复制代码
{

test: /\.js$/,

loader:'babel-loader',

options:{

plugins:['syntax-dynamic-import']

},

},
复制代码
增加了option选项，至此，能识别我们：const App = () => import('../component/Login.vue');的语法。

 

我的代码是从gitlab clone下来的，执行

npm install

npm run dev
这时报错 Error: No PostCSS Config found in... 
解决方案
在项目根目录新建postcss.config.js文件，并对postcss进行配置：

module.exports = { 
  plugins: { 
    'autoprefixer': {browsers: 'last 5 version'} 
  } 
}
 

 

好了 试试

npm run dev
至此，问题全部解决了。

项目为什么上传到 GitHub 之后，再 clone 下来，再运行就得单独写一个 postcss.config.js 的文件并配置一下呢？

在npm上查到的postcss配置在webpack.config.js，
postcss.config.js是针对webpack3.0做的特殊处理