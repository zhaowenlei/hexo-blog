---
title: MacOX上安装MongoDb
date: 2016-05-17 12:54:56
category: [技术总结]
tags: [技术总结]
---
### Mac OX上安装MongoDb

MongoDB的安装有好多种安装方法，有普通青年的HomeBrew方式，也有文艺青年的源码编译方式。我只想快速的装起来用一下，所以我选最简单的HomeBrew。

请参考官方文档 ： http://docs.mongodb.org/manual/tutorial/install-mongodb-on-os-x/

更新Homebrew的package数据库，在Mac的终端中输入：

`$ brew update`

然后耐心等待，这个没有任何显示，估计要几分钟，取决于网络的速度。然后就列出了一大堆东西，就可以进行后续步骤了。

开始安装MongoDb

`$ brew install mongodb`

然后继续等待MongoDb下载完成。这个比较贴心了，有下载进度百分比。

```
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/mongodb-2.6
######################################################################## 100.0%
==> Pouring mongodb-2.6.5.mavericks.bottle.2.tar.gz
==> Caveats
To have launchd start mongodb at login:
ln -sfv /usr/local/opt/mongodb/*.plist ~/Library/LaunchAgents
Then to load mongodb now:
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mongodb.plist
Or, if you don’t want/need launchctl, you can just run:
mongod —config /usr/local/etc/mongod.conf
==> Summary
🍺 /usr/local/Cellar/mongodb/2.6.5: 17 files, 331M
dus-MacBook-Pro:CountMeInServer dudaniel$
```
启动MongoDb

上面最后提示的直接启动MongoDb的方法.

`mongod —config /usr/local/etc/mongod.conf`
连接到MongoDb,可以用命令行工具mongo连接：

```
$ mongo
MongoDB shell version: 2.6.5
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type “help”.
For more comprehensive documentation, see
http://docs.mongodb.org/
Questions? Try the support group
http://groups.google.com/group/mongodb-user
```
还可以找个可视化的工具。MongoDb的可视化管理工具有很多，这里有个列表http://docs.mongodb.org/ecosystem/tools/administration-interfaces/， 经人推荐试用了一下Robomongo，这个是跨平台的，Windows，Mac, Linux下都可以使用，不错。

![](/imgs/robomongo.jpg)
其实这在其次，MongoDb的用法大多数还都是编程使用，比如和nodeJs结合使用，正在探索。
