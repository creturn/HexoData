title: "ubuntu下eclipse+erlang+mongodb开发环境搭建"
id: 222
date: 2012-08-15 23:56:28
tags: 
categories: 
- 原创作品
---

最近公司webGame项目中用到了Erlang+MongoDB，没办法项目需要那就学呗。

学这个东西最起码得有环境吧，今天搭建开发环境就顺便记录一下，依然在ubuntu下进行开发。

顺便说下，如果做开发，最好选择linux，因为很多环境在linux下搭建很方便。win下一般也会

有相应的发法搭建，但是经常会遇到一些莫名其妙的问题。因此建议做开发的同学还是Linux下吧。

好了不废话了，继续我们的环境搭建（Ubuntu下进行）：

Mongodb 安装

打开命令输入如下命令：

```shell
sudo apt-get install mongodb
```
<!--more-->
输入管理员密码后确认安装，然后等待安装结束。如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-224239的屏幕截图.png "2012-08-15 22:42:39的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-224239的屏幕截图.png)

网速快的花2-3分钟就装好了。然后我们检查下是否成功：

```shell
pgrep mongo -l     //查看是否存在mongo的进行。为啥不看端口...初次安装我不知道端口号
```

可以看到已经存在Id为3913的mongod进程，说明已经启动运行了。如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-224758的屏幕截图.png "2012-08-15 22:47:58的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-224758的屏幕截图.png)

接着我们直接输入mongo进去瞅瞅这货长啥样，输入help我们可以看看帮信息，如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-225103的屏幕截图.png "2012-08-15 22:51:03的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-225103的屏幕截图.png)

可以看到我的版本现在是2.0.4

好了，接下来就继续安装Erlang：

ubuntu的好处就是基本上所有需要的东西只需要一句话就搞定，输入如下代码：

```shell
sudo apt-get install erlang                   //是不是依旧那么简单，输入管理员密码然后确认安装，大概45M左右的程序包需要下载
```

等待了3分钟后安装完成。说实在的，第一次用还没看过文档，继续erlang看看有啥东西

结果，我的猜想是错误的，其实应该是erl ....  输入erl -h  可以看到一部分帮助信息。如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-231227的屏幕截图.png "2012-08-15 23:12:27的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-231227的屏幕截图.png)

接下来就是给神器eclipse装erlang的插件了。其实我更愿意称呼vim为神器。但在项目管理等方面还是觉得eclipse是不错的选择

>打开eclipse -> help -> install new soft ，点击work with 旁边的Add添加erlang的一个Ide插件，地址为：

```shell
http://erlide.org/update         //erlide 一个erlang的ide插件
```

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-231934的屏幕截图.png "2012-08-15 23:19:34的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-231934的屏幕截图.png)

点击确定等待几分钟后自动会更新插件信息

如图所示，我是两个一起选择，下面的是ide的插件上面的看名字应该是一些插件和code那么一并装了。

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-232358的屏幕截图.png "2012-08-15 23:23:58的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-232358的屏幕截图.png)

一路下一步就行，中间需要同意他们的协议，要不不能继续。

装玩后，file-&gt;new Project 可以看到里面已经有了erlang project如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-233218的屏幕截图.png "2012-08-15 23:32:18的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-15-233218的屏幕截图.png)

好了，环境也就搭建起来了~~
