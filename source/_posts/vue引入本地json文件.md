---
title: vue引入本地json文件
date: 2017-09-25 21:48:12
categories: vue
tags:
	  - vue
---

# vue引入本地json文件

今天在写vue火车票查票小项目是遇到了的，在选择城市的时候，因为没有数据，所以犯难了，于是想到引入一个本地的json文件。

我用的是vue-cli脚手架,打开dev-server.js
![vue](1.png)
```
var app = express() //这个后面

//外部引入json文件
var appData = require('../data.json');
 
app.get('/api/city',function (req,res) {
  res.json({
    data:appData
  });
});
```
这里用到了node的express框架，搭建了一个简易的api请state求。
再将json文件放到对应位置。
我用的是axios请求
```
axios.get('/api/city')
	.then(function(response){
		console.log(response.data)
		})
	.catch(function(err){
		console.log(err)
		})
	)
```

不过这个方法只能在本地用，如果你要打包的话，这个方法就不行了。打包的话，我是写在了state的默认数据里。暂时没有想到什么更好的办法。
