# Request模块学习笔记
8102年了还在用`require('http')`?赶紧换request吧！  
request是一个简化HTTP/HTTPS调用的库，我们使用request来编写一个简单的网络爬虫来进行学习。  
项目Github地址：https://github.com/request/request
## 关于request
Request的设计目的便是简化Node的HTTP调用难度，并且提供了对HTTPS的支持。  
Request的使用十分简单，举个例子，我们要获取某不存在的网站的主页：
```js
const request = require('request');
request('https://www.google.com/',function(error,response,body){
    if(!error&&response.statusCode==200){
        console.log(body);
    }else{
        console.log(error);
    }
})
```
