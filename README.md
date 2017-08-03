node-crawler 爬取页面图片素材
===========================

先说下思路：

> 1. 使用[node-crawler](http://nodecrawler.org/)先批量怕渠道页面的整个html结构代码的字符串
> 
> 2. 使用正则表达式检索出所有的img标签
> 
> 3. 循环找出的全部img匹配标签，再次使用正则找出全部的图片地址
> 
> 4. 将找出的全部图片地址拼装为JSON格式，把那个调用node-fs模块保存在本地
> 
> 5. 使用批量下载工具将爬取的全部图片下载到本地
> 
> 6. 仅限于不需要登录就可以看到图片的网站，且网站的内容由后台渲染。（像花瓣网，是通过js读取数据来生成的页面，本文所提到的代码还不能对其进行抓取）

## 1. 安装必要的node模块

```

$ npm install crawler

$ npm install fs

```

## 2-4. 这里贴出主要代码

```js

var Crawler = require("crawler");
var fs = require("fs");

var count = 0;
var imgUrlName = ['duowan', 'nipic', 'sina'];
var allData = [];
var imgReg = /<img.*?(?:>|\/>)/gi;
var c = new Crawler({
    maxConnections : 10,
    callback : function (error, res, done) {
        if(error){
            console.log(error);
        }else{
            var imgArr = res.body.match(imgReg) || [''];
            var length = imgArr.length;
            var srcArr = [];
            var name;
            for(var i = 0; i < length; i += 1){
                var imgUrl = imgArr[i].match(/src=[\'\"]?([^\'\"]*)[\'\"]?/i)[1];
                srcArr.push(imgUrl);
            };
            var obj = {
                'webName': imgUrlName[count],
                'imgArr': srcArr
            };
            allData.push(obj);
            count += 1;
            if(count > 2){
                fs.writeFile('index.json', JSON.stringify(allData), function(err) {
                    if (err) {
                        console.log('出现错误!');
                    }
                    console.log('已经存储到index.json');
                })
            }
        }
        done();
    }
});

// Queue a list of URLs
c.queue(['http://tu.duowan.com/tu', 'http://www.nipic.com/index.html', 'http://photo.sina.com.cn/']);

```

> 匹配img标签的正则： `/<img.*?(?:>|\/>)/gi`
>
> 匹配img标签中图片地址的正则： `/src=[\'\"]?([^\'\"]*)[\'\"]?/i`
>
> c.queue队列可以添加单个或多个url，具体请参考[node-crawler 官方文档](http://nodecrawler.org/)
>
> `count`变量作为一个统计变量，统计回调函数调用的次数，当调用次数大于或等于`c.queue`队列中数组最大长度时输出全部数据
>
> 使用fs模块，将抓取到的数据保存在本地

## 批量下载并存储在服务器中，这个我不会哈哈。

都是php大大帮我存在后台的，具体怎么存等我研究明白了在回来补上。

### 参考链接： 

http://www.jb51.net/article/67464.htm

http://nodecrawler.org/