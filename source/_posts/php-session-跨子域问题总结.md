title: "PHP session 跨子域问题总结"
id: 187
date: 2012-08-03 01:26:03
tags: 
categories: 
- 原创作品
- 技术文章
---

今天遇到这个问题是，自己要在别人现有的东西上面做修改。由于仅仅是子域

当时就行肯定有简单的解决方法，Google了10多分钟搞定：

Session主要分两部分：

一个是Session数据，该数据默认情况下是存放在服务器的tmp文件下的，是以文件形式存在
另一个是标志着Session数据的Session Id，Session ID，就是那个 Session 文件的文件名，Session ID 是随机生成的，因此能保证唯一性和随机性，确保 Session 的安全。一般如果没有设置 Session 的生存周期，则 Session ID 存储在内存中，关闭浏览器后该 ID 自动注销，重新请求该页面后，重新注册一个 session ID。如果客户端没有禁用 Cookie，则 Cookie 在启动 Session 会话的时候扮演的是存储 Session ID 和 Session 生存期的角色。
两个不同的域名网站，想用同一个Session，就是牵扯到Session跨域问题！
默认情况下，各个服务器会各自分别对同一个客户端产生 SESSIONID，如对于同一个用户浏览器，A 服务器产生的 SESSION ID 是 11111111111，而B 服务器生成的则是222222。另外，PHP 的 SESSION数据都是分别保存在本服务器的文件系统中。想要共享 SESSION 数据，那就必须实现两个目标：
一个是各个服务器对同一个客户端产生的SESSION ID 必须相同，并且可通过同一个 COOKIE 进行传递，也就是说各个服务器必须可以读取同一个名为 PHPSESSID 的COOKIE；另一个是 SESSION 数据的存储方式/位置必须保证各个服务器都能够访问到。这两个目标简单地说就是多服务器(A、B服务器)共享客户端的 SESSION ID，同时还必须共享服务器端的 SESSION 数据。

有三种解决方法：

1.只要在php页面的最开始（要在任何输出之前，并且在session_start()之前）的地方进行以下设置

```php
ini_set('session.cookie_path', '/');
ini_set('session.cookie_domain', '.mydomain.com');
ini_set('session.cookie_lifetime', '1800');
```

2.在php.ini里设置

```php
session.cookie_path = /
session.cookie_domain = .mydomain.com
session.cookie_lifetime = 1800
```

3.在php页面最开始的地方（条件同1）调用函数

```php
session_set_cookie_params(1800 , '/', '.mydomain.com');
```
<!--more-->

我的解决方法是在入口出添加如下代码：

```php
ini_set('session.cookie_path', '/');
ini_set('session.cookie_domain', '.domain.com'); //注意domain.com换成你自己的域名
ini_set('session.cookie_lifetime', '1800');
```

如图：

站点一

[![](http://www.creturn.com/asset/uploads/2012/08/session_1.png "session_1")](http://www.creturn.com/asset/uploads/2012/08/session_1.png)

站点二

[![](http://www.creturn.com/asset/uploads/2012/08/session_2.png "session_2")](http://www.creturn.com/asset/uploads/2012/08/session_2.png)

&nbsp;

可以看到两个站点的PHPSESSID是一样的，当然也解决了跨子域名的问题了
