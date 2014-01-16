title: "python+pygtk+api写的一个简单的词典"
id: 273
date: 2012-11-11 13:14:54
tags: 
categories: 
- 原创作品
- 技术文章
---

最见看看gtk的资料顺便练练手，就顺手写个词典玩玩，十来分钟的事情～～

python越用越觉得顺手，简洁。不到50行代码就完成了，代码没有精简，要不然更短小～

查询是调用爱词霸的接口：http://dict-co.iciba.com/api/dictionary.php?w= 接口地址后面跟

查询的词汇

返回的是xml数值。里面附带所有信息，包括发音 的声音文件地址，如果想带发音的功能

可以直接使用它的文件，返回数值接口信息截图：

[![QQ20131106-1](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-1.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-1.png)

<!--more-->

截图：
[![QQ20131106-2](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-2.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-2.png)

查询结果：

[![QQ20131106-3](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-3.png)](http://asset.creturn.com/asset/uploads/2013/11/QQ20131106-3.png)

代码

```python
#!/usr/bin/env python
#coding: utf-8 
#Autor: return Blog:www.creturn.com Date:2012.11.11
import gtk,urllib
from xml.dom import minidom
def init():
        global bufferText 
        global inputText
        win = gtk.Window()
        win.set_title('我的词典')
        win.connect('destroy',lambda w:gtk.main_quit())
        inputText = gtk.Entry()
        findBtn = gtk.Button('查询')
        viewText = gtk.TextView()
        inputText.connect('key-press-event',bindingKeys)
        hbox = gtk.HBox()
        vbox = gtk.VBox()
        scroll = gtk.ScrolledWindow()
        scroll.add(viewText)
        bufferText = gtk.TextBuffer()
        findBtn.connect('clicked',findWordEvent)
        viewText.set_buffer(bufferText)
        viewText.set_size_request(200,200)
        viewText.set_wrap_mode(gtk.WRAP_WORD_CHAR)
        hbox.set_size_request(200,30)
        hbox.pack_start(inputText,False)
        hbox.pack_start(findBtn,False)
        vbox.pack_start(hbox)
        vbox.pack_start(scroll)
        win.add(vbox)
        win.show_all()
        gtk.main()
def bindingKeys(widget,keyval):
                # print keyval
                if keyval.keyval == 65293:
                        findWordEvent(None, None)

         
def findWordEvent(widget,data=None):
        bufferText.set_text('正在查询...')
        word = inputText.get_text()
        xml = get_word_translate(word)
        bufferText.set_text('')
        doc = minidom.parseString(xml)
        dicList = doc.getElementsByTagName('acceptation')
        if len(dicList):
                for dic in dicList:
                        bufferText.insert_at_cursor(dic.childNodes[0].nodeValue)
        else:
                bufferText.insert_at_cursor('查无此词')
def get_word_translate(word):
        urlApi = 'http://dict-co.iciba.com/api/dictionary.php?w=%s'%word
        result = urllib.urlopen(urlApi).read()
        return result
def main():
        init()
if __name__ == '__main__':
        main()
```

github: [https://github.com/creturn/workSnippet-python/blob/master/dict/main.py](https://github.com/creturn/workSnippet-python/blob/master/dict/main.py "https://github.com/creturn/workSnippet-python/blob/master/dict/main.py")
