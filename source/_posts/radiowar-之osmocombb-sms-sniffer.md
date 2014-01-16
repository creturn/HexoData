title: "Radiowar 之OsmocomBB SMS Sniffer"
id: 320
date: 2013-11-06 20:34:49
tags: 
categories: 
- 原创作品
- 技术文章
---

好久没写东西了，前几天看到Radiowar这个词的时候感觉有点意思，

可能是因为之前写过War Driving 的文章对War很敏感吧.

Radiowar指的是无线安全攻击，不过很多人听到无线攻击就和无线网络破解划等号了。

其实Radiowar 包含所有无线电子设备如：2.4GHz网络、RFID、NFC等等.

很多人现在都在玩Proxmark3 奈何成本过大，做个试验最少也得千把来块，自己对这个也仅仅是感兴趣，

所以就从一些简单且Cheap的设备玩起，刚好最近看到有几个人发了GSM网络嗅探的文章，

自己就弄些设备来玩，成本较低，一套设备下来100左右。



设备清单：

1. 摩托罗拉 C118 (25块)
2. FT232RL USB TO TTL (30元)
3. 摩托罗拉 Motorola C118专用数据连接线 (10块)
4. MiniUSB 链接线(10元，这个大家手里应该都有)


如下图：

[![QQ20131106-7](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-7.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-7.png)
<!--more-->

淘宝购买地址就放在文章结尾吧.

外围设备是上面那些，现在还需要一台PC以及装有Linux操作系统的虚拟机，当然PC上面直接装的linux更好

还有一个主角：OsmocomBB

这里稍微介绍下OsmocomBB，OsmocomBB是GSM协议栈(Protocols stack)的开源实现，全称是Open source mobile communication Baseband.

目的是要实现手机端从物理层(layer1)到layer3的三层实现。

但是目前来看，真正的物理层(physical layer)并没有真正的开源实现，暂时也没看到实施计划。只有物理层控制。

因为真正的物理层是运行在baseband processor的DSP core上,涉及到许多信号处理算法的实现，而且还要牵扯很多硬件RF的东西。

额好吧，上面这段OsmocomBB介绍文字来自百度~~

我自己的实验环境为：

os x 10.9  + VirtualBox + Ubuntu 12.04 (32位)

操作系统各个平台都无所谓，我想大多数都用虚拟机来测的，所以建议使用Ubuntu 12.04 至少按照我这里介绍的做下来应该不会出错。

虚拟机里面如何安装Ubuntu 12.04我这里几不操作了，网上教程一大堆。

首先我们安装交叉编译环境：

1\. 用户目录下建立source/arm

2\. 进入arm目录再创建三个目录  build、 install 、src

3\. 进入src目录分别下载这三个文件包：

http://ftp.gnu.org/gnu/gcc/gcc-4.5.2/gcc-4.5.2.tar.bz2

http://ftp.gnu.org/gnu/binutils/binutils-2.21.1a.tar.bz2

ftp://sources.redhat.com/pub/newlib/newlib-1.19.0.tar.gz

4\. 下载 gnu-arm-build.2.sh 文件到arm目录 http://bb.osmocom.org/trac/raw-attachment/wiki/GnuArmToolchain/gnu-arm-build.2.sh

目录结构如图所示：

[![屏幕快照 2013-11-06 06.05.58 PM](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.05.58-PM.png)](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.05.58-PM.png)

编译前需要安装相应的库文件执行如下命令：

[shell]
sudo apt-get install build-essential libgmp3-dev libmpfr-dev libx11-6 libx11-dev texinfo flex bison libncurses5 libncurses5-dbg libncurses5-dev libncursesw5 libncursesw5-dbg libncursesw5-dev zlibc zlib1g-dev libmpfr4 libmpc-dev
[/shell]

安装完

在arm根目录执行如下命令:

[shell]
chmod +x  gnu-arm-build.2.sh
./ gnu-arm-build.2.sh
[/shell]

解释下这两条命令的意思，chmod +x 是给与  gnu-arm-build.2.sh 可执行权限 ， ./ gnu-arm-build.2.sh 是执行它

执行 gnu-arm-build.2.sh后会提示你是否继续，ctrl+c 则取消，直接回车。如下图所示：

[![屏幕快照 2013-11-06 06.16.42 PM](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.16.42-PM.png)](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.16.42-PM.png)

完成后会看到arm/install目录结构如图所示：

[![屏幕快照 2013-11-06 06.31.12 PM](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.31.12-PM.png)](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.31.12-PM.png)

我们需要将 arm/install/bin目录加入环境变量中，注意这里最好直接填写绝对路径

获取绝对路径命令，在arm/install/bin目录下执行pwd命令。如下图

[![屏幕快照 2013-11-06 06.32.52 PM](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.32.52-PM.png)](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.32.52-PM.png)

加入环境变量的方法用vi打开 ~/.bashrc 在最后一样加入  export PATH=$PATH:/home/creturn/source/arm/install/bin

注意$PATH:冒号后面接的是你自己的绝对路径，有的同学不会用vi的话我就把具体命令写出来

[shell]
cd 回车  #切换到主目录
vi ./.bashrc  #打开文件
shift+g  #按shift和g 跳转到最后一样
o       #输入小写字母o换行并进入插入模式，然后把上面需要加入的代码加入进去
esc    # 按esc键进入命令模式
:wq  #按冒号然后输入回车编辑完成
[/shell]

为照顾linux新手对写点东西，汗~~

重新打开终端或者直接执行source ~/.bashrc 让环境变量生效

在终端中输入arm然后按tab键，如果出现 arm-开头的如下图所示就说明编译环境搞定了：

[![屏幕快照 2013-11-06 06.43.44 PM](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.43.44-PM.png)](http://asset.creturn.com/asset/uploads/2013/11/屏幕快照-2013-11-06-06.43.44-PM.png)

好了累了半死才把编译环境搞定，接下来下载 OsmocomBB 源码

在source目录下执行下面两条命令：

[shell]
git clone git://git.osmocom.org/osmocom-bb.git
git clone git://git.osmocom.org/libosmocore.git
[/shell]

先编译osmocom核心库文件,进入libosmocore 执行如下命令：

[shell]
autoreconf -i
./configure
make
sudo make install
[/shell]

然后进入osmocom/src下目录

git checkout --track origin/luca/gsmmap 切换到这个分支

执行make命令

如果不出意外你所需软件环境和固件都编译好了。Ubuntu12.04 默认能够识别FT232R所以不用装驱动。

接下来链接硬件， FT232RL USB TO TTL 和 摩托的数据线链接注意上面对应的标识接入GND、RX、TX 所以不用担心接错，我也第一次摸板子。

链接完成后需要注意，如果链接正常下载板上面的蓝色的灯和红色的灯都要亮，不然肯定是接触不良，

我之前由于接触不良导致固件写入一半就会停止，或者包其他错误。所以这里需要特别注意。

插入硬件后在虚拟机菜单中把接入下载板子的usb接口分配给虚拟机如图：

[![QQ20131106-9](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-9.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-9.png)

在终端中输入lsusb 如果驱动正常就能看到usb-serial

看下分配前和分配后的区别如下图：

[![QQ20131106-10](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-10.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-10.png)

接下来我们写入固件到手机中，这里需要说明的一点，很多人说是刷入固件，导致很多人误认为是刷机，其实只是把

固件加载到手机raw中执行而已，所以也不要担心输入固件后手机就不能用了。

切换到 osmocom-bb/src/host/osmocon/目录 执行如下命令：

[shell]
/osmocon -m c123 -p /dev/ttyUSB0  ../../target/firmware/board/compal_e88/layer1.compalram.bin
[/shell]

这里同样需要说明下有的 -m 后面接的是 c123xor 区别是是否检测数据总和

注意上面命令需要在关机下执行，然后短按开机键，注意不要长按不然就开机了

如果不出意外就能看到如下图：

[![QQ20131106-11](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-11.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-11.png)

这里需要提醒下，由于买的手机都是二手的，所以接口处有可能会有松动，所以如果看到数据写入被取消就多试几次

调整下接口看看是否有松动，我机会每次都要试4-5次才能正常写入固件，看到staring up 基本就成功了，同事可以看到

手机上面也显示了 Layer 1 osmocom-bb 字样

[![QQ20131106-12](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-12.png)

](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-12.png)接下来在 /osmocom-bb/src/host/layer23/src/misc/ 目录执行  ./cell_log

输出日志信息，在里面可以看到基站信息

找到类似信息

cell_log.c:248 Cell: ARFCN=117 PWR=-62dB MCC=460 MNC=01

如下图：

[![QQ20131106-13](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-13.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-13.png)

记住ARFCN后面的编号

然后在同样目录下输入下面命令把数据流导入本地4729端口

[shell]

./ccch_scan -i 127.0.0.1 -a 117 # 注意这里的117就是上面的ARFCN后面的编号

[/shell]

如下图：

[![QQ20131106-14](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-14.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-14.png)

接下来用wireshark 抓包，输入如下命令：

[shell]

sudo wireshark -k -i lo -f 'port 4729'

[/shell]

然后在wireshark里面过滤gsm_sms协议数据，里面就包含了短信数据

[![QQ20131106-15](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-15.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-15.png)

&nbsp;

好了就到这里了，剩下的自己慢慢玩吧，做了一个视频演示：

&nbsp;
<embed src="http://player.youku.com/player.php/sid/XNjMwNjY5OTA0/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>

高清版下载地址：[http://pan.baidu.com/s/1ACSOE](http://pan.baidu.com/s/1ACSOE)

淘宝地址在这里就去掉了，给别人带来了“一些”麻烦， 需要买的直接在淘宝里面搜索相应的关键字就能找到。 修改日期：2013.12.1

手机就随便在淘宝上找一家就行
