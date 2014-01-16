title: "户外黑阔之War Driving 应用原理分析及其工具使用"
id: 280
date: 2012-12-20 13:34:34
tags: 
categories: 
- 原创作品
- 技术文章
---

很久没写东西了，最近太忙了～～～

今天给介绍一个新名词 “War Driving” ，看过幽灵的朋友们有没有留意到最后抓捕大哥组织里面那群人时候他们开着货车移动式攻击有没有印象？如果不感兴趣，那么下面的文章可以不用看了。感兴趣的继续～～

War Driving：是驾驶攻击也称为接入点映射，这是一种在驾车围绕企业或住所邻里时扫描无线网络名称的活动。

要想进行驾驶攻击你就要具备一辆车、一台电脑（膝上型电脑）、一个工作在混杂模式下的无线以太网网卡，还有一个装在车顶部或车内的天线。

因为一个无线局域网可能仅局限于一栋办公楼的范围内，外部使用者就有可能会入侵网络，获得免费的企业内部网络连接，还可能获得公司的一些记录

和其他一些资源。 用全方位天线和全球定位系统（GPS），驾驶攻击者就能够系统地将802.11b/g/n无线接入点映射地址映射。

好吧，上面的废话都是来自读娘。驾乘式攻击，其实在现实当中用来收集信息才是比较使用的。那么收集到的信息有哪些？

<!--more-->

看两张图吧：

[![20121222153346861](http://asset.creturn.com/asset/uploads/2013/11/20121222153346861.png)](http://asset.creturn.com/asset/uploads/2013/11/20121222153346861.png)

上图是sqllite数据库进行的一个联合查询的结果，也是收集到信息达一部分。

bssid：是无线路由或者说是无线设备的硬件地址

lat , lon :是此无线设备达经纬度GPS信息

ssid：是无线设备达名称

capabilities:可以明显的看到这个是无线设备达加密方式

说道这里，很多人可能会说，你丫显得没事要这个有什么用？好吧，确实有点～～

那么接下来看看这张图片：

[![20121222153349302](http://asset.creturn.com/asset/uploads/2013/11/20121222153349302.png)](http://asset.creturn.com/asset/uploads/2013/11/20121222153349302.png)

上图是根据第一张图片上面信息以地图达方式展现出来～～～

上图中，所有绿色为没有加密的无线网络，黄色达为wep加密方式。红色的是wpa/wpa2加密方式。。。。

至于这个有什么用？试想下，开部车。外卖兜一圈，周围达无线网络信息全部都会收集到了。至于用途，你们懂的～

下来说分析重点：

这些信息如何得到的并且在地图上显示？其实需要两样东西，一个GPS用于记录当前所在位置。另外一个就是wifi设备。用于收集

wifi信息。记录下当前GPS位置周围达wifi信息不就可以了～

设备需求：

一般情况下如果要做专业点并且比较精确信息量大，那么就需要一个外接USB GPS，笔记本一台，wifi增益天线。车子一辆

大家熟知的bt5就自带一个kismon  有需要的可以google youtube上有视频演示。

不过这些条件有点苛刻。不过我的方法是只要你有一台android手机那么，恭喜你，可以玩玩war driving 了。为什么这么说？

现在的android手机GPS和wifi都是必备了。那么只要我们写个APP根据上面原理进行采集信息不久行了？很不幸达是APP都有人写好了

我们直接拿过来用了～～wigle 恩，就是它！

接下来介绍如何使用它：

先看看装完以后桌面图标：

[![2012122215342748](http://asset.creturn.com/asset/uploads/2013/11/2012122215342748.png)](http://asset.creturn.com/asset/uploads/2013/11/2012122215342748.png)

上面圈出来的就是它安装后达图标

[![2012122215345131](http://asset.creturn.com/asset/uploads/2013/11/2012122215345131.png)](http://asset.creturn.com/asset/uploads/2013/11/2012122215345131.png)

1.数据库中新增的数据量，也就是本次采集达数据（手机放兜里出去转移圈也就不少了）

2。数据库中达总数据量

3。当前GPS信息（好吧，忘记删掉自己达位置信息了）

4。是当前区域可以接收到的wifi信息

怎么把这些信息放到地图上看？这个作者还是想的比较周到。支持从数据库中导出kml格式（google地图标记用的就是这个）

切换到data栏目-》KML Export DB 然后收集插到电脑上存储设备（内存卡）根目录下的wigle目录里面就存在刚才导出达KML

和sqllite数据库信息如图：

[![2012122215347148](http://asset.creturn.com/asset/uploads/2013/11/2012122215347148.png)](http://asset.creturn.com/asset/uploads/2013/11/2012122215347148.png)

其实kml就是xml的数据格式改了个名字而已可以打开看看：

[![2012122215349575](http://asset.creturn.com/asset/uploads/2013/11/2012122215349575.png)](http://asset.creturn.com/asset/uploads/2013/11/2012122215349575.png)

然后打开google earth然后选择打开文件找到你刚才导出达kml就可以看到效果了～～

做了一个视频演示用GoogleEarth打开kml，系统为Ubuntu 12。04 为了推广linux，做的一个视频演示吧
<div class="CTPnoimage CTPplaceholder" style="width: 480px !important; height: 400px !important; position: relative !important; top: auto !important; right: auto !important; bottom: auto !important; left: auto !important; z-index: auto !important; clear: none !important; float: none !important; vertical-align: -webkit-baseline-middle !important; margin: 0px !important; -webkit-margin-before-collapse: collapse !important; -webkit-margin-after-collapse: collapse !important;" title="http://player.youku.com/player.php/sid/XNDkxMDE2MDEy/v.swf">
<div class="CTPplaceholderContainer" style="opacity: 0.5 !important;">
<div class="CTPlogoContainer">
<div class="CTPlogo">Flash</div>
<div class="CTPlogo CTPinset">Flash</div>
</div>
</div>
</div>
百度网盘现在地址：[http://pan.baidu.com/share/link?shareid=136595&amp;uk=2317334154](http://pan.baidu.com/share/link?shareid=136595&amp;uk=2317334154)
