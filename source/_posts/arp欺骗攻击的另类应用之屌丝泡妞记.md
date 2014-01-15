title: "Arp欺骗攻击的另类应用之屌丝泡妞记"
id: 244
date: 2012-10-27 00:05:23
tags: 
- arp欺骗
- linux arp欺骗
categories: 
- 原创作品
---

EveryNote 真是个好动西，跨平台的笔记本。好到，平时记录一些自己的东西，都懒得在写博客了。呵呵，很久都

没有写文章了。最近一切烦心事一大堆，就弄个好玩的东西来玩玩。Arp欺骗的另类应用，为啥说是arp欺骗的另类

应用呢？假设合租屋里隔壁有个漂亮MM，想知道他平时都上网干些什么，喜欢什么，怎么弄到她的联系方式？

嘿嘿，接着往下看看。

arp欺骗，我想大家都应该知道怎么回事了。不知道的去问度娘。。。

就不废话了，还是直接上图上教程比较实在。环境，ubuntu 12.10 当然你也可以是ubuntu的其他版本。或者其他linux

系统。环境如图：

[![](http://www.creturn.com/asset/uploads/2012/10/001.png "001")](http://www.creturn.com/asset/uploads/2012/10/001.png)

Arp欺骗工具我们就用ettercap吧，经典工具我就不多说了。

```shell
sudo apt-get install  ettercap-text-only  //安装纯字符界面的ettercap
```

<!--more-->

上面命令就可以直接安装它了，因为ubuntu软件仓储里面已经有了，直接可以进行安装。我已经装好了，版本信息如下：

[![](http://www.creturn.com/asset/uploads/2012/10/002.png "002")](http://www.creturn.com/asset/uploads/2012/10/002.png)

还需要安装一个东西，也是今天的主角：driftnet

估计linux 的同学应该都很少有人知道这个是神码玩意吧，我就先说说它的作用吧，driftnet是一款

用于抓取指定接口数据流上面图片的软件并且吧嗅探到的图片友好的显示在一个窗口当中。

ubuntu安装命令：

```shell
sudo apt-get install driftnet
```

看看我的版本信息吧：

[![](http://www.creturn.com/asset/uploads/2012/10/0031.png "003")](http://www.creturn.com/asset/uploads/2012/10/0031.png)

装完这些估计有人就会问了，这个和MM有神码联系？

额，好吧，那就说说怎么个联系法。

先看张流程图片：

[![](http://www.creturn.com/asset/uploads/2012/10/005.png "005")](http://www.creturn.com/asset/uploads/2012/10/005.png)

这张图片已经说明了我们要做什么了，和怎么去做了，那么下来我们就进行实践操作。

首先，我们需要进行Arp欺骗，把MM的PC欺骗过来，欺骗过来后，MM那边的所有数据都是通过通过我们的机器

再出去（外网）的。

首先以知MM ip地址：192.168.1.100    网关地址：192.168.1.253

使用下面命令进行欺骗

```shell
sudo ettercap -i wlan0  -T -M arp:remote /192.168.1.253/ /192.168.1.100/   //记住需要root权限
```

注意：我这里用-i wlan0是因为我自己用的是无线，根据实际情况自行操作

输入命令后的效果如图：

[![](http://www.creturn.com/asset/uploads/2012/10/006.png "006")](http://www.creturn.com/asset/uploads/2012/10/006.png)

然后我们需要用到我们的主角了，driftnet

```shell
driftnet -i wlan0
```

运行后随便浏览下带有图片的网站效果如图：

[![](http://www.creturn.com/asset/uploads/2012/10/007.png "007")](http://www.creturn.com/asset/uploads/2012/10/007.png)

调整好浏览图片的窗口，然后我们就等着MM上网呗。这里当然只能自己做实验了，因为宽带独享。。。。。

手里只有个ipad那就欺骗它来看看欺骗效果。手机也可以欺骗哦。

电脑截图：

[![](http://www.creturn.com/asset/uploads/2012/10/009.png "009")](http://www.creturn.com/asset/uploads/2012/10/009.png)

怎么样？效果似乎不错是吧？嘿嘿，最后再来一张手机拍的PC和ipad的合照，这样也可以有个对照参考，如图：

[![](http://www.creturn.com/asset/uploads/2012/10/008.jpg "008")](http://www.creturn.com/asset/uploads/2012/10/008.jpg)

>再给点提示，以便屌丝们可以获取更多信息，tcpdump是linux比较有名的抓包工具，利用tcpdump可以抓取更详细的信息，这样再结合社会工程学，我想MM的信息肯定是跑步掉了！

以后有时间再写一个dns欺骗之最终泡妞必杀技！嘿嘿。。。
