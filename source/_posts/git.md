---
title: git本地项目上传
date: 2017-09-21 15:37:48
categories: git
tags: 
      - git
---

* 第一步

  [github官网](https://github.com)申请帐号

* 第二步

  1. 创建ssh：打开终端检测是否存在ssh：命令cd ~/.ssh

     如果不存在，通过默认的参数直接生成ssh

     $ssh-keygen -t rsa -C xxxxx@gmail.com（注册github时的email）

  2. 在github中添加ssh

     ![git](git1.png)

     ​

     点击创建SSH Keys

     ![git2](git2.png)

     Title: 可以自己起，只是为了容易分辨。

     Key：打开你生成的id_rsa.pub文件，将其中内容拷贝至此。

  3. 上传项目

     ![git4](git4.png)

     ​

     Repository name:通常就写自己自己要建的项目名。

     Description：就是你对项目的描述了。

     点击“Create repository”。

     跳出下面页面

     ![git5](git5.png)

     开始上传项目

     1. git init //初始化本地仓库
     2. git add README.md //添加
     3. git add . 添加项目中所以的文件
     4. git commit -m "first commit"//提交到要地仓库，并写一些注释
     5. git remote add origin git@github.com:youname/Test.git //连接远程仓库并建了一个名叫：origin的别名
     6. git push -u origin master //将本地仓库的东西提交到地址是origin的地址，master分支下