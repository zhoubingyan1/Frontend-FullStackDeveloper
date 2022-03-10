#node常见命令
> cd 对应的文件夹, npm init 生成package.json文件
> 引入模块,依赖 express 和 utility 两个模块,npm install express utility --save
> 起服务 node 文件.js

***
#目前demo用的模块
-（+*） express 是基于node.js平台框架   npm install express --save

-（+*） utility 这个库来生成 md5 值  npm install express utility --save   
        utility 的 github 地址：https://github.com/node-modules/utility
        var utility = require('utility')
        utility.md5(value)

-（+*） superagent 是个 http 方面的库，可以发起 get 或 post 请求
        superagent(http://visionmedia.github.io/superagent/ ) 


-（+*） cheerio是可以理解成Node.js版的jquery,用来从网页中以css selector取数据,使用方式跟jquery一样的
        cheerio(https://github.com/cheeriojs/cheerio )