title: Android 上传PHP xUtils Bug修复分析过程
date: 2014-01-15 15:12:04
tags:
- Android
- xUtils
---

##起因

作为全职PHPer偶尔需要客串下Androider,最近公司的一个项目需要Android的客户端(主要图片特效处理及其上传)，自己就客串下Androider.

之前有过Android开发经验所以做这个挺顺手的，几乎所有东西直接github中拿过来改改就用，不过在处理图片上传的时候选择了xUtils这个

开源工具类,用起来确实比较好用，挺方便的，例如如下代码就可以实现上传:

```java
RequestParams params = new RequestParams();
params.addBodyParameter("file", file);
HttpUtils httpUtils = new HttpUtils();
httpUtils.send(HttpRequest.HttpMethod.POST, UPLOAD_URL, params, new RequestCallBack<String>() {
    @Override
    //上传失败处理方法
    public void onFailure(HttpException arg0, String msg) {
        alert(msg);
    }
    @Override
    //上传进度处理
    public void onLoading(long total, long current,
            boolean isUploading) {
        if (isUploading) {
            Log.i(LOG_NAME, "upload:" + current + "/" + total);
        }
    }
    @Override
    //上传成功处理
    public void onSuccess(ResponseInfo<String> responseInfo) {
        alert(responseInfo.result);
        Log.i(LOG_NAME, responseInfo.result);
    }
});
```
可以看到用起来比较方便，如果自己写还是比较麻烦的。不过最让人头疼的不是使用方法，而是作为接收端为PHP的话是接收不到上传的文件，最后经证实
不仅仅是PHP C# 也有问题, 网上搜素了下不少人都遇到问题不过没有解决方案，看来只能自己动手解决了
<!--more-->
##问题分析

既然要解决问题，那么就需要分析bug可能出现的地方，既然是HTTP上传那么我们得知道在PHP 在HTTP协议中文件是怎么处理上传的，直接在官方文档
就可以找到

>PHP 能够接受任何来自符合 RFC-1867 标准的浏览器（包括 Netscape Navigator 3 及更高版本，打了补丁的 Microsoft Internet Explorer 3 或者更高版本）上传的文件。

这个是PHP官方文档中给出的解释, PHP在处理上传的时候遵循的是RFC-1867标准，那么我们接下来看看什么是RFC-1867。

这里我给出一个RFC-1867的说明文档地址 [RFC-1867 说明](http://www.ietf.org/rfc/rfc1867.txt)，太长了就不放在这里了只拿核心重点内容过来看看：

```http
# The client might send back the following data use POST method:

Content-type: multipart/form-data, boundary=AaB03x

--AaB03x
content-disposition: form-data; name="field1"

Joe Blow
--AaB03x
content-disposition: form-data; name="pics"; filename="file1.txt"
Content-Type: text/plain

 ... contents of file1.txt ...
--AaB03x--
```
这里通俗点讲，RFC-1867通过HTTP协议传输特定的格式来实现上传文件,比如我们上传一个文件名字叫做hello.txt的文本到服务端，那么发起请求的HTTP格式应该就如下：

```http
POST /up.php HTTP/1.1
Host: creturn.com
Content-Length: 294
Content-type: multipart/form-data, boundary=AaB03x

--AaB03x
content-disposition: form-data; name="file"; filename="hello.txt"
Content-Type: text/plain

hello
--AaB03x--
```

这里HTTP头部协议没有写完整只是为了说明问题写的格式。在这些描述信息中
1. 第一个Content-type（http头部描述信息）值multipart/form-data 是告诉服务器此次发送的数据是上传文件数据, boundary是告诉服务器文件数据之间分割标示符是 AaB03x
2. 在HTTP body数据中以--AaB03x 分割多个文件，每个分隔符下面的描述信息是作为上传文件的描述信息
3. 发送结束后需要以分隔符加“--”符号进行标示

如果这三点没有问题那么就能正确上传，当然次实例中肯定不成功因为HTTP头部协议描述信息简短切不争取（比如长度）

##解决过程

既然上面已经对问题进行分析了，同样也知道了只要发送过程是按照RFC-1867的标准进行发送那么至少PHP是能够接收到上传的文件，那么接下来我们要解决的就是如果判断

或者查看xUtils发送文件过程中是否遵循了RFC-1867标准

那么如何查看xUtils是否发送了正确的数据格式？ 有两种方案，一个是利用代理工具抓包，另外一个方案就是直接抓包

这里就说说直接抓包，代理抓包可以google一大堆。

pc 上面建立无线热点分享给手机，这样所有的数据都通过电脑走，不会用pc分享热点的google一大堆，或者为了偷懒买个传说中的mini WIFI都行

抓包工具window推荐smartSniff, linux或者os x直接就tcpdump也行

先写个简单的app装到手机上这里给出上传代码：

```java
String UPLOAD_URL = "http://www.creturn.com/up.php";
File file = new File(Environment.getExternalStorageDirectory() ,"hello.txt");
RequestParams params = new RequestParams();
params.addBodyParameter("file", file);
HttpUtils httpUtils = new HttpUtils();
httpUtils.send(HttpRequest.HttpMethod.POST, UPLOAD_URL, params, new RequestCallBack<String>() {
    @Override
    //上传失败处理方法
    public void onFailure(HttpException arg0, String msg) {
        alert(msg);
    }
    @Override
    //上传进度处理
    public void onLoading(long total, long current,
            boolean isUploading) {
        if (isUploading) {
            Log.i(LOG_NAME, "upload:" + current + "/" + total);
        }
    }
    @Override
    //上传成功处理
    public void onSuccess(ResponseInfo<String> responseInfo) {
        alert(responseInfo.result);
        Log.i(LOG_NAME, responseInfo.result);
    }
});
```
上面代码作用是把sdcard根目录的hello.txt文件上传到UPLOAD_URL, 所以在sdcard根目录放一个hello.txt文件里面内容随便写点

php服务端这边就直接打印上传的文件信息就行，代码很简单:

```php
<?php
print_r($_FILES);
?>
```
如果上传成功就会反馈上传文件的信息，写好app装到手机连接好wifi然后在pc上面抓包，我这里用的是smart sniffer

- 抓包

 1. 打开smart sniffer 
 2. 选择菜单 Options -> Capture Options 
 3. 选择你分享wifi的网卡
 4. 确定点击开始抓包
 
 在手机app上操作上传文件时候就可以看到抓包工具中已经有相应的http数据，抓包工具会对所有流量抓取所以如果有其他包干扰还可以
 
 设置相应的过滤规则，这里就不阐述google就能找到
 
 看看我们抓到的包内容:
 
![smart sniffer抓包内容](http://asset.creturn.com/asset/img/android_sniffer.png)

图种可以看到我们上传的HTTP包信息，不过很明显反馈的信息提示是没有上传成功的。

那么接下来我们怎么去分析这个？怎么去排查问题？很多人应该能够想到，要是有个正确参照物不就很容易分析出问题出处？

那么我们在建立一个html文件用浏览器同样上传sdcard里面的hello.txt文件,html内容如下:

```html
<html>
<body>

<form action="http://www.creturn.com/up.php" method="post" enctype="multipart/form-data">
<label for="file">Filename:</label>
<input type="file" name="file" id="file" /> 
<br />
<input type="submit" name="submit" value="Submit" />
</form>

</body>
</html>
```
 用同样的方法抓包看看正确的包内容是什么样的:
 
 ```http
POST /up.php HTTP/1.1
Host: www.creturn.com
Connection: keep-alive
Content-Length: 294
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Origin: http://222.73.234.196
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.63 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryIexdOW8e2EZyciDK
Referer: http://222.73.234.196/up.html
Accept-Encoding: gzip,deflate,sdch
Accept-Language: zh-CN,zh;q=0.8

------WebKitFormBoundaryIexdOW8e2EZyciDK
Content-Disposition: form-data; name="file"; filename="hello.txt"
Content-Type: text/plain

hello upload
------WebKitFormBoundaryIexdOW8e2EZyciDK
Content-Disposition: form-data; name="submit"

Submit
------WebKitFormBoundaryIexdOW8e2EZyciDK--
 ```
 可以看到处理HTTP头部描述信息和包体里面多了一个Submit，几乎一样
 
之前说过上传过程中的几个重点，然后对比下我们发现Content-Type描述信息多了一个charset字符编码描述信息

那么要做测试的话肯定就要把不同的地方去掉，然后对包进行回放看看是否成功

>包回放指的是包数据包重新发送一次

回放数据包有两种方法，一种直接修改xUtils源码重新上传。这里说一个简单的方法window自带的telnet ,用telnet 链接服务器80端口

手动发送数据
>注意： windows7默认没有安装需要在控制面板->程序和功能->打开或者关闭windows功能中开启

如何进行手动发送？按照我们之前的想法去掉charset描述信息然后手动发送，那么先去掉charset信息后的包内容放入记事本:

```http
POST /up.php HTTP/1.1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.73.11 (KHTML, like Gecko) Version/7.0.1 Safari/537.73.11
Content-Length: 233
Content-Type: multipart/form-data; boundary=6wYgRevA02R_Uy4EJP31EcIJtsBlZtRv
Host: wwwcreturn.com
Connection: Keep-Alive
Accept-Encoding: gzip

--6wYgRevA02R_Uy4EJP31EcIJtsBlZtRv
Content-Disposition: form-data; name="file"; filename="hello.txt"
Content-Type: application/octet-stream
Content-Transfer-Encoding: binary

hello upload

--6wYgRevA02R_Uy4EJP31EcIJtsBlZtRv--
```
打开cmd 然后输入telnet www.creturn.com 80 然后黏贴进去看看效果

![telnet post](http://asset.creturn.com/asset/img/andriod-telnet.png)

为了印证我们的才行可以把charset加上去和去掉的进行对比看看是不是加了之后就收不到上传文件的信息。

其实根据HTTP协议来讲理论上加不加charset应该不会影响上传，但结果这个问题确实是由于charset引起的。

接下来就简单了找到根源解决就行，在源码里面进行搜索 boundary ，找到地方根据作者写的方法注释掉其中添加charset的代码:

```java
    protected String generateContentType(
            final String boundary,
            final Charset charset) {
        StringBuilder buffer = new StringBuilder();
        buffer.append("multipart/" + multipartSubtype + "; boundary=");
        buffer.append(boundary);
        //这里就是需要注释掉的代码
        /*if (charset != null) {
            buffer.append("; charset=");
            buffer.append(charset.name());
        }*/
        return buffer.toString();
    }


```

好了搞定收工~~
