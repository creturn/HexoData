title: "Python拼接图片"
id: 290
date: 2013-03-12 13:42:13
tags: 
- python
categories: 
- 原创作品
- 技术文章
---

貌似这是13年第一次写文章了。好吧，去年主要写一些安全方面的文章，今年就把技术文章也加进来，已做笔记（其实天天用evenote）。

人常说，不偷懒的程序员不是好程序员。今天有个任务比较麻烦，当然这个还不是我自己的任务。其实是美工的，不过一个团队，能帮

点尽量帮点忙。神马任务呢？就是游戏的图场景已经被地图编辑器分割成300×300像素的小图。然后部门经理要美工吧图都拼成大图以便

修改方便…100多场景，每个场景里面至少150张小图，任你美工技术多牛叉，我想没一个星期是弄不来的。

So 就帮妹子一把~。~

<!--more-->

废话不多说的直接开始，先看看目录结构和图片的结构：

[![QQ20131106-4](http://www.creturn.com/asset/uploads/2013/11/QQ20131106-4.png)](http://www.creturn.com/asset/uploads/2013/11/QQ20131106-4.png)

其实分析小图，和对比里面的small我发现图片命名规则是pic后面一个数字是分割的列数，下划线后面的则是坐在行数

并且每张小图都是300×300

这样的话我自己的处理思路是这样的：

首先，分析文件提取出行数和列数，然后计算原图的大小。

然后不管用哪种语言，只要支持图片处理的就基本可以做。新建一个原图大小的画布，或者空的图片。

最后根据行列位置填充相应的小图就拼接出来原图了。自己虽然做php的，但更偏爱python来处理小事情。so…上码

```python
&lt;pre title=&quot;&quot;&gt;#!/usr/bin/env python
#coding: utf-8

import PIL.Image as Image
import os
class ImageUtil:
 def __init__(self,path):
 # 小图宽度
 self.sizeX = 300
 # 小图高度
 self.sizeY = 300
 # 高度有多少行
 self.sizeH = 0
 # 高度有多少列
 self.sizeW = 0
 #拼接图片存放位置
 self.imageDirPath = path
 # 拼接后图片的总宽度
 self.toImageWidth = 0
 # 拼接后图片的总高度
 self.toImageHeight = 0
 # 拼接图片后存放的位置
 self.toImageSavePath = ''
 # 获取需要转换成大图的图片大小
 def getToImageSize(self):
 tmpX = 0
 tmpY = 0
 imgList = os.listdir(self.imageDirPath)
 tmpList = []
 # 过滤出目标文件
 for img in imgList:
 if img.find('pic') == 0:
 # 分割文件名
 tmpList.append(img.replace('pic','').replace('.jpg','').split('_'))
 # 计算行和列
 for size in tmpList:
 if int(size[1]) &gt; tmpY:
 tmpY = int(size[1])
 if int(size[0]) &gt; tmpX:
 tmpX = int(size[0])
 self.sizeH = tmpY
 self.sizeW = tmpX
 # 计算目标大图长宽
 self.toImageWidth = self.sizeX * (tmpX - 1)
 self.toImageHeight = self.sizeY * (tmpY -1)
 return self
 # 开始转换
 def convert(self):
 toImage = Image.new('RGBA',(self.toImageWidth,self.toImageHeight))

for y in xrange(0,self.sizeH - 1):
 for x in xrange(0,self.sizeW - 1):
 fname = 'pic%s_%s.jpg'%(x,y)
 fname = self.imageDirPath + '/' + fname
 fromImage = Image.open(fname)
 toImage.paste(fromImage,(x*self.sizeX,y*self.sizeY))
 # print fromImage.size

toImage.save('./save_1001.jpg')

pass

def main():
 path = './front_1001'
 imgUtl = ImageUtil(path)
 imgUtl.getToImageSize().convert()

pass

if __name__ == '__main__':
 main()&lt;/pre&gt;
```

运行后会根据指定目录分析文件后，最后拼接城一个大图。

合成后的图片大小：

[![QQ20131106-5](http://www.creturn.com/asset/uploads/2013/11/QQ20131106-5.png)](http://www.creturn.com/asset/uploads/2013/11/QQ20131106-5.png)

嘿嘿，还是python好用
