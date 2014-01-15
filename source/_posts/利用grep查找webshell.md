title: "Linux下利用grep查找webshell"
id: 21
date: 2012-07-01 02:22:58
tags: 
- webshell
- 后门
categories: 
- 技术文章
---

## WEB SHELL 排查
grep （global search regular expression(RE) and print out the line,全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。Unix的grep家族包括grep、egrep和fgrep。**

利用grep命令我们可以查找常见的漏洞、webshell和其他恶意文件。本文使用的grep版本为2.9，如果你使用一个低于2.5.4的grep，那本片文章中的一些命令可能无法正常工作。可以用grep -v或grep -version确定一下版本。你也可以使用grep –help查看更多信息。如下图：

[![](http://www.creturn.com/asset/uploads/2012/07/1.png "1")](http://www.creturn.com/asset/uploads/2012/07/1.png)
<!--more-->
##发现漏洞的常见方法：

为什么大多数web应用程序都会发现一些不安全的代码，就是因为调用了一些不安全的函数，而又未进行过滤。例如：命令注入或远程代码执行，可以执行客户端传递的参数。这里常常会用到shell_exec函数。我们可以用grep命令搜索文件中存在shell_exec函数的地方，如下：

```shell
grep -Rn “shell_exec *( ” /var/www
```

[![](http://www.creturn.com/asset/uploads/2012/07/2.png "2")](http://www.creturn.com/asset/uploads/2012/07/2.png)

上图中，我们可以看到可能存在漏洞的函数行和路径。

另一个例子：

include require include_once和require_once都是可能存在问题的地方，它可能会造成本地文件包含漏洞。

我们可以使用grep查找出现该函数的地方然后进行测试判断，如下:

```shell
grep -Rn “include *(” /var/www
grep -Rn “require *(” /var/www
grep -Rn “include_once *(” /var/www
grep -Rn “require_once *(” /var/www
```

[![](http://www.creturn.com/asset/uploads/2012/07/3.png "3")](http://www.creturn.com/asset/uploads/2012/07/3.png)

以上两个简单的例子可以做为白盒挖掘漏洞的参考，下面介绍一下查找webshell和其他恶意文件的方法：

常见的webshell都会包含一些功能，如执行命令、下载文件、编辑文件、反弹连接等行为。

除了常见的shell_exec，base64_decode和eval，还有其他的一些特征码，比如phpspy2006中会包含“Version:2006,proxycontents”，phpspy2008中会包含“phpspypass,goaction(‘backconnect”等等。

还有以下一些常见的特征：

```php
phpinfo
system
php_uname
chmod
fopen
flclose
readfile
edoced_46esab
passthru
```

我们可以用grep搜索包含这些函数的文件，如下：

```shell
grep -Rn “shell_exec *(” /var/www
grep -Rn “base64_decode *(” /var/www
grep -Rn “phpinfo *(” /var/www
grep -Rn “system *(” /var/www
grep -Rn “php_uname *(” /var/www
grep -Rn “chmod *(” /var/www
grep -Rn “fopen *(” /var/www
grep -Rn “fclose *(” /var/www
grep -Rn “readfile *(” /var/www
grep -Rn “edoced_46esab *(” /var/www
grep -Rn “eval *(” /var/www
grep -Rn “passthru *(” /var/www
```

[![](http://www.creturn.com/asset/uploads/2012/07/4.png "4")](http://www.creturn.com/asset/uploads/2012/07/4.png)

[![](http://www.creturn.com/asset/uploads/2012/07/5.png "5")](http://www.creturn.com/asset/uploads/2012/07/5.png)

当然这些可以合并成一条命令，如下：

```shell
grep -RPn “(passthru|shell_exec|system|phpinfo|base64_decode|chmod|mkdir|fopen|fclose|readfile|php_uname|eval|tcpflood|udpflood|edoced_46esab) *(” /var/www
```

本文转载于： http://www.freebuf.com/articles/4074.html
