#目标
>建立一个 lesson1 项目，在其中编写代码。当在浏览器中访问 http://localhost:3000/ 时，输出 Hello World。


#挑战
>访问 http://localhost:3000/?q=alsotang 时，输出 alsotang 的 sha1 值，即 e3c766d71667567e18f77869c65cd62f6a1b9ab9。

#知识点
-（+*）包管理器 npm 。使用 npm 安装包，并自动安装所需依赖
-（+*）框架 express 。学习新建 express 实例，并定义 routes ，产生输出

#框架 Express
>express 是 Node.js 应用最广泛的 web 框架，现在是 4.x 版本，它非常薄。跟 Rails 比起来，完全两个极端


>新建一个文件夹叫 lesson1 的，进去里面安装 express
```shell
    $ mkdir lesson1 && cd lesson1
    # 这里没有从官方 npm 安装，而是使用了大淘宝的 npm 镜像
    $ npm install express --registry=https://registry.npm.taobao.org
```

>新建app.js  touch.js

```shell
    // 这句的意思就是引入 `express` 模块，并将它赋予 `express` 这个变量等待使用。
    var express = require('express');
    // 调用 express 实例，它是一个函数，不带参数调用时，会返回一个 express 实例，将这个变量赋予 app 变量。
    var app = express();

    // app 本身有很多方法，其中包括最常用的 get、post、put/patch、delete，在这里我们调用其中的 get 方法，为我们的 `/` 路径指定一个 handler 函数。
    // 这个 handler 函数会接收 req 和 res 两个对象，他们分别是请求的 request 和 response。
    // request 中包含了浏览器传来的各种信息，比如 query 啊，body 啊，headers 啊之类的，都可以通过 req 对象访问到。
    // res 对象，我们一般不从里面取信息，而是通过它来定制我们向浏览器输出的信息，比如 header 信息，比如想要向浏览器输出的内容。这里我们调用了它的 #send 方法，向浏览器输出一个字符串。
    app.get('/', function (req, res) {
    res.send('Hello World');
    });

    // 定义好我们 app 的行为之后，让它监听本地的 3000 端口。这里的第二个函数是个回调函数，会在 listen 动作成功后执行，我们这里执行了一个命令行输出操作，告诉我们监听动作已完成。
    app.listen(3000, function () {
    console.log('app is listening at port 3000');
    });
```

> node app.js
>这时候我们的 app 就跑起来了，终端中会输出 app is listening at port 3000。这时我们打开浏览器，访问 http://localhost:3000/，会出现 Hello World。如果没有出现的话，肯定是上述哪一步弄错了，自己调试一下。
