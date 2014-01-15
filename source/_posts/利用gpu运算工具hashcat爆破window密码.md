title: "利用GPU运算工具hashcat爆破window密码"
id: 258
date: 2012-11-25 12:55:47
tags: 
- window密码
- 爆破
categories: 
- 原创作品
- 技术文章
---

今天介绍一款利用显卡超强运算能力进行爆破密码的工具hashcat，用它来进行爆破window密码

当然他支持的不仅仅是window SAM密码的爆破。

我本机环境和显卡信息如图：

[![224255j1foiui0lrc10rsh](http://www.creturn.com/asset/uploads/2013/11/224255j1foiui0lrc10rsh.png)](http://www.creturn.com/asset/uploads/2013/11/224255j1foiui0lrc10rsh.png)

<!--more-->

hashcat的下载地址为：http://hashcat.net/files_legacy/hashcat-0.40.7z 这个为非gui，如果需要gui的可以去官网下载gui版本

解压后可以看到如图内容：

[![224256bl2n8535y5pvl87n](http://www.creturn.com/asset/uploads/2013/11/224256bl2n8535y5pvl87n.png)](http://www.creturn.com/asset/uploads/2013/11/224256bl2n8535y5pvl87n.png)

其中window和linux对应的版本都有。我是64位ubuntu 那么就用hashcat-cli64.bin

为了方便使用我做了一个软链接：

```shell
ln -s /home/return/soft/hack/hashcat/hashcat-cli64.bin /usr/bin/hashcat

```

这样打开终端后不管在什么位置直接输入hashcat就行了，看看它支持的密码类型：

[![224300uew6cie9zieq7cbq](http://www.creturn.com/asset/uploads/2013/11/224300uew6cie9zieq7cbq.png)](http://www.creturn.com/asset/uploads/2013/11/224300uew6cie9zieq7cbq.png)

其中红色部分就是sam加密类型

我们先看看hashcat的运算能力怎么。我们用php的md5加密6个z试试看看它进行爆破速度怎么样：

```php
<?php
	file_put_contents('hashcat.txt',md5('zzzzzz');
?>
```

上面代码保存成x.php ,终端下php x.php 会在当前目录下生成hashcat.txt 里面就保存了zzzzz的md5值：

453e41d218e071ccfb2d1c99ce23906a

然后终端输入：

```shell
time hashcat -a 3 -m 0 hashcat.txt ?l?l?l?l?l?l
```

解释下命令：time 用来统计执行这条命令耗费的时间，-a 是破解模式 -m 密码类型 后面的？l 是密码的配置符号。？l为小写字符a-z

结果如图：

[![224303cnhu6wowugpmlpm6](http://www.creturn.com/asset/uploads/2013/11/224303cnhu6wowugpmlpm6.png)](http://www.creturn.com/asset/uploads/2013/11/224303cnhu6wowugpmlpm6.png)

可以看到我标记出来的：

1.当前的匹配模式  2.破解结果 3.time 计算出耗费的时间

可以看到穷举6位字母用时14秒效果已经很不错了。当然这个也跟自己的显卡性能有关

接下来回到正题，看看如果破解window密码。破解之前我们还需要两个工具bkhive,和samdump2

ubuntu下执行如下命令即可安装

```shell
sudo apt-get install bkhive
```

好了。继续，看下我的分区情况。因为现在在ubuntu下我的window在c盘第一分区上装着，如图：

[![224307iz3gs432za1vawbx](http://www.creturn.com/asset/uploads/2013/11/224307iz3gs432za1vawbx.png)](http://www.creturn.com/asset/uploads/2013/11/224307iz3gs432za1vawbx.png)

挂在sda1分区到我的/win

```shell
mkdir /win &amp; mount /dev/sda1 /win
```

上面命令是在根目录下创建win文件夹并且挂载sda1

看看win下是否挂载成功，如果成功了，可以看到win目录下有window系统c盘（系统盘）的内容

[![224310ayqj3g611m6tgy11](http://www.creturn.com/asset/uploads/2013/11/224310ayqj3g611m6tgy11.png)](http://www.creturn.com/asset/uploads/2013/11/224310ayqj3g611m6tgy11.png)

拷贝：cp /win/Windows/System32/config/SYSTEM  和SAM 文件到当前文件夹

然后执行命令如下图：

[![224313ubvwmmtwwfyxlttg](http://www.creturn.com/asset/uploads/2013/11/224313ubvwmmtwwfyxlttg.png)](http://www.creturn.com/asset/uploads/2013/11/224313ubvwmmtwwfyxlttg.png)

命令解释：

>1和2是拷贝 SYSTEM和SAM到当前文件夹

>3.根据SYSTEM文件提取bootkey

>4.利用bootkey提取出hash值

提取出来的哈是值如图：

[![224314lf5dy5dq3b95zkkd](http://www.creturn.com/asset/uploads/2013/11/224314lf5dy5dq3b95zkkd.png)](http://www.creturn.com/asset/uploads/2013/11/224314lf5dy5dq3b95zkkd.png)

window的hash密文其实就是最后两个‘：’号之间的数据，到这里拿到值后其实就可以去网上看下有已经破解的密文了没

可以看如图，已经找到密码了：

[![224317qpwypa77l2gly2zv](http://www.creturn.com/asset/uploads/2013/11/224317qpwypa77l2gly2zv.png)](http://www.creturn.com/asset/uploads/2013/11/224317qpwypa77l2gly2zv.png)

那么接下来我们爆破。取出密文，也就是最后两个‘：’之间的数据保存为pwd.txt

执行如下图命令：

[![224318amdzfb6wrydsiebr](http://www.creturn.com/asset/uploads/2013/11/224318amdzfb6wrydsiebr.png)](http://www.creturn.com/asset/uploads/2013/11/224318amdzfb6wrydsiebr.png)

[![224320t0idau0t1r5edyuu](http://www.creturn.com/asset/uploads/2013/11/224320t0idau0t1r5edyuu.png)](http://www.creturn.com/asset/uploads/2013/11/224320t0idau0t1r5edyuu.png)

中机会有提示确认信息，输入大写YES就行

命令解释：

>hashcat后面的参数 -a 3 破解模式 -m 1000 哈是类型 后面的？l?l是匹配符由于我的密码有些复杂所以前面我补上了几位明文，这样能提高点速度（写文章么，总不能等半天）。

>1. 可以看到已经破解出来  
>2. 用命令cat来查看已经破解出来的密码

这种是基于穷举的，也可以用字典。命令：
```shell
hashcat -a 0 -m 0 hashfile.txt passwordlist.txt
```
>解释下这条命令。-a 0利用字典破解 -m是hash类型0为md5 hashfile.txt为密文，passwordlist.txt为字典，这里就不演示了
