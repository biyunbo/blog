---
title: vue cli chromedriver报错
date: 2017-09-23 17:51:21
categories: vue
tags:
	  - vue
---

# vue cli chromedriver报错
今天休息在家，想要把自己没有写完的vue小项目写写，从自己的github上clone下来，然后npm install 。。。。。
没想到竟然报错。
![git4](1.png)
第一次遇到这样的报错。因为用的是vue cli 并不是自己配置的webpack，所以有点不之所措词。
然后查了是因为chromedriver没有安装成功的问题。
后来网上查了下，发现了解决办法。
```
npm install chromedriver --chromedriver_cdnurl=http://cdn.npm.taobao.org/dist/chromedriver
```
然后继续npm install，就没有出现报错了。