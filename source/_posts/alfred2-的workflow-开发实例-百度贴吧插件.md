title: "Alfred2 的workflow 开发实例-百度贴吧插件"
id: 314
date: 2013-07-06 15:22:05
tags: 
- alfred
- os x
- php
- workflow
categories: 
- 原创作品
- 技术文章
---

貌似有点懒，今年第二篇…

Alfred2 os x 上的一个比较好的插件好的插件，对于命令控来说绝对是福音。

官网地址：http://www.alfredapp.com

它的插件机制加上新版本的workflow支持fallback后似的他的更受欢迎。

先上一张今天要介绍的插件，贴吧的workflow。自己平时比较喜欢在贴吧里面看小说。

之前找了一个贴吧的插件，不过用起来发现作者采用了缓存机制，其实加入缓存机制其实还是

比较好的，不过缓存了一天多都没变化这个就有问题了。本来想直接修改原来的插件去掉缓存机制或者

把缓存时间修改短点也行，但是发现是python运行编译的文件，去作者github上的看看能找到源码不结果

令人很失望，一个这么简单的东西放了那么一大堆不知道都起什么作用的包，还不开放源码，还敢用吗？

<!--more-->

所以果断删掉，自己写。先看看效果图：

[![QQ20131106-6](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-6.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-6.png)

1\. 首先从百度贴吧抓取贴吧对应的帖子，我们知道pc端访问http://tieba.baidu.com/f?kw=吧名 就可以访问贴吧的帖子

不过为了加快访问速度和效率，我们可以抓取手机端贴吧帖子。手机端贴吧帖子页面大小相对pc端要小很多。所以获取

贴吧帖子信息速度会快很多，贴吧手机版本地址为：wapp.baidu.com/f?kw=吧名

2\. 抓取到信息后用正则过滤出我们需要的内容然后生成相应的xml这样就可以实现我们自己的workflow

代码如下：

```php
<?php
/*========================================================================
#   FileName: mytieba.php
#     Author: Creturn
#      Email: master@creturn.com
#   HomePage: http://www.creturn.com
# LastChange: 2013-07-05 02:24:51
========================================================================*/
require_once('workflows.php');
class App
{
	private static $_instance;
	private $_workflows;
	function __construct(){
		$this->_workflows = new workflows();
	}
	
	public function getInstance()
	{
		if (!self::$_instance instanceof self) {
			self::$_instance = new App();
		}
		return self::$_instance;
	}
	public function request($url)
	{
		return $this->_workflows->request($url);
	}
	public function filterDataList($data)
	{
		$dataList = array();
		preg_match_all('/<div class="i">(.*?)<\/div>/', $data, $dataList);
		return $dataList[1];
	}
	public function getData($keyword)
	{
		//后面可以在这里加入缓存机制
		//缓存路径$this->_workflows->cache();
		$url = 'http://wapp.baidu.com/f?kw=' . $keyword;
		$data = $this->request($url);
		$num = 1;
		foreach( $this->filterDataList($data) as $item ) {
			preg_match('/<p>(.*?)<\/p>/', $item, $time);
			preg_match('/kz=(.*?)&amp;/', $item, $id);
			$time= " " . str_replace('&#160;', ' ', $time[1]);
			preg_match('/<a (.*?)>(.*?)<\/a>/',$item, $title);
			$item = str_replace('&#160;', '', $title[2]);
			$this->_workflows->result($num . '.' . time(), 'http://tieba.baidu.com/p/' . $id[1], $item, $time, 'icon.png');
			$num ++;
		}
		
		return $this->_workflows->toxml();
	}
	public function run($query)
	{
		return $this->getData($query);
	}
}
#echo App::getInstance()->run('大主宰');
?>

```

当然你也可以去github下载： [https://github.com/creturn/alfredwordflow-tieba](https://github.com/creturn/alfredwordflow-tieba)
