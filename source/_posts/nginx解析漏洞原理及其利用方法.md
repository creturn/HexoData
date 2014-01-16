title: "Nginx解析漏洞原理及其利用方法"
id: 233
date: 2012-08-29 03:08:09
tags: 
- Nginx解析漏洞
categories: 
- 原创作品
- 技术文章
---

Nginx解析漏洞已经是比较老的漏洞了，但是互联网上还有不少使用存在解析漏洞的nginx版本。

很久没写文章了，睡觉去去法客转了圈看到一片nginx漏洞的渗透文章，才发现自己似乎也没写过。

nginx解析漏洞是由于nginx部分版本程序本身的漏洞导致解析非可以执行脚本程序如PHP.

如下面两个假设在存在漏洞的站点上有一张图片url地址为：

```shell
www.creturn.com/logo.jpg   //假设存在这个图片
```

而当我们正常访问，nginx会把这个当作非脚本语言直接读取传送会客户端（也就是浏览器），但是

存在解析漏洞的nginx会把如下连接解析并且执行~：

```shell
www.creturn.com/logo.jpg/a.php  （老的解析方式）这样写的话nginx会把logo.jpg当作脚本解析执行后再输出
www.creturn.com/logo.jpg%00.php //这个是7月中旬爆出的解析漏洞
```

这样的解析漏洞有什么危害？其实很多站的安全或者程序样做的比较严谨的话，就没办法直接拿下，但是很多社交类

或者交互类的站点上往往允许用户上传图片，如社交网站一般都会允许上传头像~这样如果有心人传送一个图马上去就可以直接解析了。

<!--more-->

好了，就拿一个实例来说明：

某交友网站：

[![](http://asset.creturn.com/asset/uploads/2012/08/nginx.png "nginx")](http://asset.creturn.com/asset/uploads/2012/08/nginx.png)

看到这么多图片，嘿嘿~~~肯定有上传的地方。先注册个帐号。刚注册完就提示你上传照片。

上传一张事先准备好的图马。博客内有制作方法，可查询。

看到图片，还需要审核~~

[![](http://asset.creturn.com/asset/uploads/2012/08/n.png "n")](http://asset.creturn.com/asset/uploads/2012/08/n.png)

右击图片审查元素（我用的是chrome浏览器）：

[![](http://asset.creturn.com/asset/uploads/2012/08/nginx_t.png "nginx_t")](http://asset.creturn.com/asset/uploads/2012/08/nginx_t.png)

可以看到图片的连接，不过不要高兴~~ 看到名称middle.jpg了没，如果你是程序员的话，你应该知道

为了减少服务器压力，图片资源也会被进行处理某些区域显示的只是缩略图，缩略图是经过处理的所以

图马是不起作用的。所以必须找原图，其实原图的真实地址就是把.middle去掉，为啥？因为原图肯定会

保存的，由于我的图片在审核（当然我的头像肯定通过不了审核了），如果审核通过可以点击查看大图

一般这个大图就是你的原图（理论上）。

好了上菜刀看看连接地址为：

```shell
http://www.creturn.com/upload/picture/photo/006/65/26/503d136ee3aaf.jpg/.php
```

如下图，图片已经成功解析为可执行脚本：

[![](http://asset.creturn.com/asset/uploads/2012/08/nginx_t2.png "nginx_t2")](http://asset.creturn.com/asset/uploads/2012/08/nginx_t2.png)

稍微浏览了下，权限很松~~

好了，还是希望各大站长能够对安全重视~
