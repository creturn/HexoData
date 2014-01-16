title: "Ubuntu下的PHP开发环境架设"
id: 198
date: 2012-08-13 22:56:07
tags: 
- apache
- php
- ubuntu
categories: 
- 原创作品
- 技术文章
---

今天重新装了ubuntu那么就吧过程记录下。

打开终端，也就是命令提示符。

我们先来最小化组建安装，按照自己的需求一步一步装其他扩展。命令提示符输入如下命令：

```shell
sudo apt-get install apache2 php5-mysql libapache2-mod-php5 mysql-server
```

上面的命令是最小化组建安装amp也就是apache2 ,php5 和 mysql 在加上一个php的mysql扩展

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-002350的屏幕截图.png "2012-08-14 00:23:50的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-002350的屏幕截图.png)
<!--more-->
上面命令输入完成后提示输入密码，成功后询问你是否安装y继续。然后就等待着完成安装...

安装的时候第一次出现一个这样的界面，意思是让你设置root管理员密码，重复一次后继续..

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-003043的屏幕截图.png "2012-08-14 00:30:43的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-003043的屏幕截图.png)

安装完成后地址栏输入localhost回车后如果正常安装成功可以看到一段文字如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-003425的屏幕截图.png "2012-08-14 00:34:25的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-003425的屏幕截图.png)

我们写个PHP的探针脚本试试看看PHP有没有被支持操作如下：

```php

# sudo touch /var/www/test.php            //默认apache网站root目录是/var/www

# sudo vim /var/www/test.php             //用自己习惯的编辑器编辑如果不会用vim 可以用gedit提供vim命令

<?php
phpinfo()               //php探针脚本，就一句话
?>

```

如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-004005的屏幕截图.png "2012-08-14 00:40:05的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-004005的屏幕截图.png)

然后我们访问localhost/test.php看看能否运行，如果正常的花就可以看到如下图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-004207的屏幕截图.png "2012-08-14 00:42:07的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-004207的屏幕截图.png)

看到这个至少你的php环境已经搭建成功了，然后自己选择自己需要的组建。打开命令提示符输入下面命令：

```shell
sudo apt-get install php5    //然后按tab键  可以看到如下php扩展
```

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-004439的屏幕截图.png "2012-08-14 00:44:39的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-004439的屏幕截图.png)

像我自己就会安装如下几个组建：

```shell
sudo apt-get install php5-gd php5-curl php5-xdebug

gd                     //图库，如生成验证码，处理图片都离不开它

curl                //支持ftp，http等等协议。用起来很方便

xdebug        //装这个配合eclipse进行断点调试相当爽~~
```

其它的根据项目需要自行添加。

自己还有个习惯就是基本上从来不是用默认/var/www路径，自己一般定义在用户目录下

如我的站点目录会配置在/home/return/workspace/web  目录下，这样归档起来比较方便

修改站点目录方法,打开

```shell
vim /etc/apache2/sites-enabled/0XXXX   //在sites-enabled/0xx开头的文件里面有默认站点配置信息
```

用编辑器打开:vim(或者gedit) /etc/apache2/sites-enabled/0xxx //0xxx指的是以0开头的那个文件
如图：
[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-005858的屏幕截图.png "2012-08-14 00:58:58的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-005858的屏幕截图.png)

修改完成保存后，重新启动一次apache 让其加载刚才修改的配置文件

如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-011716的屏幕截图.png "2012-08-14 01:17:16的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-011716的屏幕截图.png)

然后在你的用户目录下的workspac/web下面写个php文件测试下看看是否正常。

当然还有最后一个配置就是虚拟目录，如果经常输入localhost或者一些项目中需要配置一些域名，而测试的话又经常需要改来改去的

因此我是习惯性的在hosts做本地域名解析，然后绑定虚拟目录。例如test.com 是我们项目用到的域名，那么首先修改hosts文件做本地解析

命令和内容如下：

```shell
sudo vim(或者gedit) /ect/hosts                   //本地域名解析就是靠它的
```

在文件中加入 127.0.0.1  test.com 如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-013902的屏幕截图.png "2012-08-14 01:39:02的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-013902的屏幕截图.png)

然后在/etc/apache2/sites-enabled/下面建立一个名为www.test.com的文件，最好直接复制一份0xxx开头的配置文件做修改就行

取名为www.test.com方便识别和辨认。apache默认会加载/etc/apache2/sites-enabled目录下的所有配置文件

文件内容如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-014154的屏幕截图.png "2012-08-14 01:41:54的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-014154的屏幕截图.png)

注意：
>SeverName就是你要绑定的域名DocumentRoot是要绑定的目录，我直接绑定了/home/return/workspace/web/test文件夹

如果不写入serverName的话test.com是无法解析到/home/return/workspace/web/test目录的

在里面加入一个php探针文件如图：

[![](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-014604的屏幕截图.png "2012-08-14 01:46:04的屏幕截图")](http://asset.creturn.com/asset/uploads/2012/08/2012-08-14-014604的屏幕截图.png)

可以看到已经解析到对应的目录了。好了基本配置就这写，每个人的使用习惯和风格不一样，自己用多了就有了

自己的使用习惯了，包括自己会了简化一些工作写一些自己的实用脚本等等。
