title: "Copy命令制作图马和隐藏文件"
id: 119
date: 2012-07-18 13:17:05
tags: 
- 图马
- 隐藏私密文件
categories: 
- 原创作品
- 技术文章
---

好几天没写BLog了，就写一个利用copy命令制作图马和隐藏文件到图片里面的教程吧

首先看下copy 命令的参数（百科直接copy来的）：
```bat
COPY [/D] [/V] [/N] [/Y | /-Y] [/Z] [/A | /B ] source [/A | /B]
[+ source [/A | /B] [+ ...]] [destination [/A | /B]]
```

>source 指定要复制的文件。

>/A 表示一个 ASCII 文本文件。

>/B 表示一个二进位文件。

>/D 允许解密要创建的目标文件destination 为新文件指定目录和/或文件名。

>/V 验证新文件写入是否正确。

>/N 复制带有非 8dot3 名称的文件时，尽可能使用短文件名。

>/Y 不使用确认是否要覆盖现有目标文件的提示。

>/-Y 使用确认是否要覆盖现有目标文件的提示。

>/Z 用可重新启动模式复制已联网的文件
<!--more-->
首先准备一个图片，和一个一句话木马，如图：

[![](http://asset.creturn.com/asset/uploads/2012/07/tuma1.png "tuma1")](http://asset.creturn.com/asset/uploads/2012/07/tuma1.png)

然后打开cmd 切换到测试目录：

[![](http://asset.creturn.com/asset/uploads/2012/07/tuma2.png "tuma2")](http://asset.creturn.com/asset/uploads/2012/07/tuma2.png)

然后进行我们的图马合成，输入命令：copy mm.jpg/b + ma.txt/a tuma.jpg

注意：copy 图马名称/b +一句话/a 生成文件名

[![](http://asset.creturn.com/asset/uploads/2012/07/tuma3.png "tuma3")](http://asset.creturn.com/asset/uploads/2012/07/tuma3.png)

看看生成图马的预览图完全能够显示成图片：

[![](http://asset.creturn.com/asset/uploads/2012/07/tuma4.png "tuma4")](http://asset.creturn.com/asset/uploads/2012/07/tuma4.png)

然后用记事本或者任何文本编辑器打开ma.jpg

[![](http://asset.creturn.com/asset/uploads/2012/07/tuma5.png "tuma5")](http://asset.creturn.com/asset/uploads/2012/07/tuma5.png)

可以看到一句话已经插入到文件中，用菜刀测试下看看行不行，记住把生成的

图马改成php(当然我写如的是php一句话，根据你自己情况自行修改)

看看截图：

[![](http://asset.creturn.com/asset/uploads/2012/07/tuma6.png "tuma6")](http://asset.creturn.com/asset/uploads/2012/07/tuma6.png)

可以看到正常解析。

接下来演示如何隐藏文件到图片中，如图所示，我要把压缩文件隐藏到图片当中如图：

把一句话打包成rar文件，然后进行合并命令：copy mm.jpg/b + ma.rar/b newmm.jpg

打包文件如图：

[![](http://asset.creturn.com/asset/uploads/2012/07/tuma7.png "tuma7")](http://asset.creturn.com/asset/uploads/2012/07/tuma7.png)

合并命令：copy mm.jpg/b + ma.rar/b newmm.jpg

看看合并结果：newmm.jpg

[![](http://asset.creturn.com/asset/uploads/2012/07/tuma8.png "tuma8")](http://asset.creturn.com/asset/uploads/2012/07/tuma8.png)

可以看到newmm.jpg文件是我们合成后的文件，并且里面已经隐藏了一个rar文件

接下来我们看看我们隐藏的文件是否损坏，把newmm.jpg改成newmm.rar看看：

[![](http://asset.creturn.com/asset/uploads/2012/07/tuma9.png "tuma9")](http://asset.creturn.com/asset/uploads/2012/07/tuma9.png)

可以看到里面打包的文件完全正常~
