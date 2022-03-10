#使用 superagent 与 cheerio 完成简单爬虫

>npm install express --save 安装express框架
>npm install superagent --save 使用superagent抓取网页
>npm install cheerio --save   使用cheerio分析网页

```shell
    const express= require('express')
    const superagent = require('superagent')
    const cheerio = require('cheerio')
    const app = express()

    app.get('/',(req,res,next)=>{
        //用superagent 去抓取 https://cnodejs.org/ 的内容
        superagent.get('https://cnodejs.org/').end((err,sres)=>{
            //常规的错误处理
            if(err){
                return next(err);
            }
            //sres.text 里面存储着网页的html的内容，将它传给cheerio.load之后
            //就可以得到一个实现jquery接口的变量，我们习惯性地将它命名为 `$`
            //剩下就是jquery的内容了
            var $ = cheerio.load(sres.text);
            var items = [];
            $('#topic_list .topic_title').each((idx,element)=>{
                var $element = $(element);
                items.push({
                    title:$element.attr('title'),
                    href:$element.attr('href')
                })
            })
            res.send(items)
        })
    })

    app.listen(3000 ,() => console.log('Example app listening on port 3000!'))
```

 