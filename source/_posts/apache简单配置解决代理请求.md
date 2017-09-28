---
title: apache简单配置解决代理请求
date: 2017-09-28 10:23:56
categories: apache
tags: 
      - apache
---

# apache简单配置解决代理请求

在写[vue-train](http://train.byb224.top)自己的小项目中，遇到一个问题。

由于api请求是自己在网上爬的，所以本地请求被限制了，于是想到了代理。

我就用了webpack-dev-server做了一个简单的代理，如下图：

![代理](1.png)

本地是没有问题的，可是，打包放到服务器，出现问题了。

由于是静态资源，所以服务器也需要配置下代理。

我用的是apache。安装在这里就不说了。

找到目录 Apache/conf/httpd.conf

在最下面添加下面代码。

```
<VirtualHost *:80>     #监听80端口  
    DocumentRoot "C:\bobo\train"       #根目录
    ServerName www.train.byb224.top     #访问的域名
	ServerAlias train.byb224.top		#别名
	ProxyPass /train/ http://m.tuniu.com/api/train/    #需要代理的目标网站
	#ProxyPassReverse /train/ http://m.tuniu.com/api/train/
	<Directory />
		Options FollowSymLinks
		AllowOverride None
		Order allow,deny
		Allow from all
	</Directory>
</VirtualHost>
```

还可以在```httpd.conf```文件里找到```Virtual hosts```将他下面的代码解除注释。

然后在找到下面路径的文件夹，```httpd-vhosts.conf```将代码贴到那里面去。这相当于引入。

重启服务，请求路径就通了。