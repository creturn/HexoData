title: "android版微信打飞机无敌补丁分析及其制作方法"
id: 303
date: 2013-08-13 15:05:43
tags: 
- android
- java
- 打飞机
categories: 
- 原创作品
- 技术文章
---

最近微信5.0版本发布后增加了游戏中心，并且自带一款打飞机游戏,ios版本刚发布1天就被crack掉,并且除了一个无敌补丁，奈何手里没ios设备也没法测试.自己玩了下，感觉游戏真没多少技术含量，不过微信的优势很明显，集成了好友数据,所以结果就是最近天天听到那些家伙喊着”打飞机” ~。~

不过对于游戏一点都不感冒的我看到好友那么高的分数再对比下自己，额真有点那个啥。所以对于技术控来说去拼命玩那个玩意就有点浪费生命了，所以本着hack for fun 的思想就试试看看能crack掉不.

首先分析下游戏

游戏对象：

飞机，子弹，炸弹，敌机
实现方法就有多种成为无敌的方法:

1.让飞机无敌，也就是不死

2.让子弹威力变大，并且不消失,这样一个子弹就可以打多个敌机

3.给自己添加”大量”炸弹,这样也就无敌了

4.让敌机碰到自己无效,这样就相当于隐形模式了,同样无敌了

5.给自己多加几条名，这样既可以达到无敌的方式又可以死掉（方便提交分数）不然死都死不掉。

<!--more-->

代码分析:

刚开始看到这个游戏以为用的是html5+js写的,不过解包后看到里面的东西就傻眼了，不过依然有方法.

1.下载 5.0 android的版本的微信,修改后缀名为zip然后解压,如图:

[![201381901926167](http://www.creturn.com/asset/uploads/2013/11/201381901926167.png)](http://www.creturn.com/asset/uploads/2013/11/201381901926167.png)

2.在解压的目录中找到assset目录然后打开里面的preload目录，如图:

[![201381901927891](http://www.creturn.com/asset/uploads/2013/11/201381901927891.png)](http://www.creturn.com/asset/uploads/2013/11/201381901927891.png)

可以看到这个目录放了一些jar包，猜测微信采用插件机制来实现功能模块的扩展,其中以”com.tencent.mm.plugin.shoot”开头的jar包，从名字中可以猜测到它就是我们的目标

这里要说后面需要用到的两款软件:

1.apktool :用来把apk或者jar包反编译成android的java虚拟机的机器码,方便修改代码逻辑

下载地址: https://code.google.com/p/android-apktool/

2.dex2jar :用来把dex文件反编译jar

下载地址: http://pan.baidu.com/share/link?shareid=2401317361&amp;uk=2317334154

3.auto-sign: 用来对apk进行签名的

下载地址: http://pan.baidu.com/share/link?shareid=2387101337&amp;uk=2317334154

4.jd-gui: 用来查看编译过的class java代码

下载地址: http://java.decompiler.free.fr/?q=jdgui

工具齐全后就开始对代码动刀

用dex2jar把com.tencent.mm.plugin.shoot开头的那个文件处理下,会得到以dex2jar.jar结尾的文件用jd-gui打开这个文件，我们就能看到编译出来的文件，不过腾讯的同学对代码进行了混淆处理所以里面有很多a,b,c,d,e这样的文件,不过很不幸的是腾讯的家伙估计太粗心了，把游戏对象和功能代码没做混淆，混淆的反而是gdx游戏框架的代码,如图:

[![201381901929371](http://www.creturn.com/asset/uploads/2013/11/201381901929371.png)](http://www.creturn.com/asset/uploads/2013/11/201381901929371.png)

在actor这个包里面明显能看出来各个类的作用，不知道腾讯的同学是妹子想多了还是不给发奖金了，核心部分不做混淆处理…

上面分析的时候说过无敌模式的几种方式，我这里采用最后一种给自己多加几条命这样也能死掉~~  不然分数都提交不了就没有意义了，看看Player类中的构造函数（或者叫做初始化函数也行）：

[java]
&lt;pre&gt;public Player()
 {
 setLiftCount(1);
 setType(GameSprite.Type.HERO);
 setState(GameSprite.State.FLIGTHING);
 setSpeedVelocity(new ag(0.0F, 0.0F));
 getBounds().height = HEIGHT;
 getBounds().width = WIDTH;
 this.mShootDelay = (4.0F * h.Ml().MC().Mi().cEP);
 this.mShootDoubleTime = h.Ml().MC().Mi().cEQ;
 }&lt;/pre&gt;
[/java]

明显能够看出来setLiftCount() 这个方法是用来初始化生命值的，如果把这里改大点，不就实现了我们的多条命了。

用apktool工具对 com.tencent.mm.plugin.shoot开头的这个包进行反编译，命令如下：

[shell]
&lt;pre&gt;apktool d 文件名&lt;/pre&gt;
[/shell]

后面的d参数是反编译，如果换成b就是再次编译回去。

解包后文件如下图所示：

[![201381901932654](http://www.creturn.com/asset/uploads/2013/11/201381901932654.png)](http://www.creturn.com/asset/uploads/2013/11/201381901932654.png)

其中smali目录就是反编译后的代码，里面的文件以 smali结尾，文件位置和包位置保持一致如下图：

[![201381901933428](http://www.creturn.com/asset/uploads/2013/11/201381901933428.png)](http://www.creturn.com/asset/uploads/2013/11/201381901933428.png)

用任意一款文本编辑器打开Player.smali文件，我们可以看到里面的代码，如下图：

[![201381901935312](http://www.creturn.com/asset/uploads/2013/11/201381901935312.png)](http://www.creturn.com/asset/uploads/2013/11/201381901935312.png)

仔细看有点像汇编，不过它却不是网上可以找到smali语法帮助，我们在这个文件中找到初始化方法，constructor &lt;init&gt;()看着是不是很眼熟：

[java]
&lt;pre&gt;.method public constructor &lt;init&gt;()V
 .locals 2

.prologue
 const/4 v1, 0x0

.line 46
 invoke-direct {p0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;&lt;init&gt;()V

.line 32
 sget-object v0, Lcom/tencent/mm/plugin/shoot/actor/Player$BulletType;-&gt;NORMAL:Lcom/tencent/mm/plugin/shoot/actor/Player$BulletType;

iput-object v0, p0, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;mBulleType:Lcom/tencent/mm/plugin/shoot/actor/Player$BulletType;

.line 47
 const/4 v0, 0x1

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;setLiftCount(I)V

.line 48
 sget-object v0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;-&gt;HERO:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;setType(Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;)V

.line 49
 sget-object v0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;-&gt;FLIGTHING:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;setState(Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;)V

.line 50
 new-instance v0, Lcom/badlogic/gdx/math/ag;

invoke-direct {v0, v1, v1}, Lcom/badlogic/gdx/math/ag;-&gt;&lt;init&gt;(FF)V

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;setSpeedVelocity(Lcom/badlogic/gdx/math/ag;)V

.line 52
 invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;getBounds()Lcom/badlogic/gdx/math/af;

move-result-object v0

sget v1, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;HEIGHT:F

iput v1, v0, Lcom/badlogic/gdx/math/af;-&gt;height:F

.line 53
 invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;getBounds()Lcom/badlogic/gdx/math/af;

move-result-object v0

sget v1, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;WIDTH:F

iput v1, v0, Lcom/badlogic/gdx/math/af;-&gt;width:F

.line 55
 invoke-static {}, Lcom/tencent/mm/plugin/shoot/a/h;-&gt;Ml()Lcom/tencent/mm/plugin/shoot/a/h;

move-result-object v0

invoke-virtual {v0}, Lcom/tencent/mm/plugin/shoot/a/h;-&gt;MC()Lcom/tencent/mm/plugin/shoot/a/d;

move-result-object v0

invoke-virtual {v0}, Lcom/tencent/mm/plugin/shoot/a/d;-&gt;Mi()Lcom/tencent/mm/plugin/shoot/a/b;

move-result-object v0

iget v0, v0, Lcom/tencent/mm/plugin/shoot/a/b;-&gt;cEP:F

const/high16 v1, 0x4080

mul-float/2addr v0, v1

iput v0, p0, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;mShootDelay:F

.line 59
 invoke-static {}, Lcom/tencent/mm/plugin/shoot/a/h;-&gt;Ml()Lcom/tencent/mm/plugin/shoot/a/h;

move-result-object v0

invoke-virtual {v0}, Lcom/tencent/mm/plugin/shoot/a/h;-&gt;MC()Lcom/tencent/mm/plugin/shoot/a/d;

move-result-object v0

invoke-virtual {v0}, Lcom/tencent/mm/plugin/shoot/a/d;-&gt;Mi()Lcom/tencent/mm/plugin/shoot/a/b;

move-result-object v0

iget v0, v0, Lcom/tencent/mm/plugin/shoot/a/b;-&gt;cEQ:I

int-to-long v0, v0

iput-wide v0, p0, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;mShootDoubleTime:J

.line 60
 return-void
.end method&lt;/pre&gt;
[/java]

根据smali语法修改如下代码：

[java]
&lt;pre&gt;.line 47
const/4 v0, 0x1

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;setLiftCount(I)V
[/java]

改为:

[java]
&lt;pre&gt;.line 47
const/4 v0, 0x7  # 注意这里修改成7

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/Player;-&gt;setLiftCount(I)V&lt;/pre&gt;
[/java]

修改的地方把默认的1改成了7，也就是说有7条命了。不过别高兴太早，如果这样改了，是没有效果的。因为程序中判断飞机不在战斗状态时候
<div></div>
<div>是不显示的，所以还需要修改一处代码，先看看原来的java代码GameSprite.smali：</div>
<div> [java]&lt;/div&gt;
&lt;div&gt;
&lt;pre&gt;public void hit(boolean paramBoolean)
 {
this.liftCount = (-1 + this.liftCount);
if (this.liftCount &lt;= 0)
{
if (paramBoolean)
{
setState(GameSprite.State.DEAD);
setSpeedVelocity(this.speedZero);
return;
}
if ((getType() == GameSprite.Type.ENEMY_AIRCAFT) || (getType() == GameSprite.Type.ENEMY_LARGE_AIRCAFT) || (getType() == GameSprite.Type.ENEMY_MIDDLE) || (getType() == GameSprite.Type.PROPS_BOMB) || (getType() == GameSprite.Type.PROPS_DOUBLE))
h.Ml().MD().c(getType());
while (true)
{
setState(GameSprite.State.EXPLODING);
break;
if (getType() == GameSprite.Type.HERO)
h.Ml().MD().MT();
}
}
setState(GameSprite.State.HITING);
 }
[/java]

这段代码是所有游戏对象基类里面的碰撞处理代码，当生命值小于1 就移除该对象，如果生命值大于1则更改属性为HITING ,不过飞机如果遇到这
<div></div>
<div>个状态肯定不行的，撞机了还能继续飞？所以这里需要判断下如果是飞机，那么就要修改它的状态为：FLIGTHING</div>
<div></div>
<div>所以如果按照java代码修改的代码会和下面的代码一样：</div>
<div>[java]&lt;/div&gt;
&lt;div&gt;
&lt;div&gt;
&lt;pre&gt;public void hit(boolean paramBoolean)
{
this.liftCount = (-1 + this.liftCount);
if (this.liftCount &lt;= 0)
{
if (paramBoolean)
{
setState(GameSprite.State.DEAD);
setSpeedVelocity(this.speedZero);
return;
}
if ((getType() == GameSprite.Type.ENEMY_AIRCAFT) || (getType() == GameSprite.Type.ENEMY_LARGE_AIRCAFT) || (getType() == GameSprite.Type.ENEMY_MIDDLE) || (getType() == GameSprite.Type.PROPS_BOMB) || (getType() == GameSprite.Type.PROPS_DOUBLE))
h.Ml().MD().c(getType());
while (true)
{
setState(GameSprite.State.EXPLODING);
break;
if (getType() == GameSprite.Type.HERO)
h.Ml().MD().MT();
}
}

//在这里添加我们的代码

if(getType==GameSprite.Type.HERO)

{

setState(GameSprite.State.FLIGTHING);

} else {
setState(GameSprite.State.HITING);

}
}
[/java]

</div>
<div></div>
<div>java 代码我们知道怎么修改了，那么smali里面的代码同样根据这个逻辑处理下即可，当然这块稍微有点难度，得看下语法手册再来修改，</div>
<div></div>
<div>修改的代码如下所示：</div>
<div>[java]&lt;/div&gt;
&lt;div&gt;
&lt;pre&gt;.method public hit(Z)V
 .locals 2
 .parameter

.prologue
 .line 45
 iget v0, p0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;liftCount:I

add-int/lit8 v0, v0, -0x1

iput v0, p0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;liftCount:I

.line 46
 iget v0, p0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;liftCount:I

if-gtz v0, :cond_4

.line 47
 if-eqz p1, :cond_0

.line 48
 sget-object v0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;-&gt;DEAD:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;setState(Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;)V

.line 57
 :goto_0
 iget-object v0, p0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;speedZero:Lcom/badlogic/gdx/math/ag;

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;setSpeedVelocity(Lcom/badlogic/gdx/math/ag;)V

.line 61
 :goto_1
 return-void

.line 50
 :cond_0
 invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;getType()Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

move-result-object v0

sget-object v1, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;-&gt;ENEMY_AIRCAFT:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

if-eq v0, v1, :cond_1

invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;getType()Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

move-result-object v0

sget-object v1, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;-&gt;ENEMY_LARGE_AIRCAFT:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

if-eq v0, v1, :cond_1

invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;getType()Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

move-result-object v0

sget-object v1, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;-&gt;ENEMY_MIDDLE:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

if-eq v0, v1, :cond_1

invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;getType()Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

move-result-object v0

sget-object v1, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;-&gt;PROPS_BOMB:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

if-eq v0, v1, :cond_1

invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;getType()Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

move-result-object v0

sget-object v1, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;-&gt;PROPS_DOUBLE:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

if-ne v0, v1, :cond_3

.line 51
 :cond_1
 invoke-static {}, Lcom/tencent/mm/plugin/shoot/a/h;-&gt;Ml()Lcom/tencent/mm/plugin/shoot/a/h;

move-result-object v0

invoke-virtual {v0}, Lcom/tencent/mm/plugin/shoot/a/h;-&gt;MD()Lcom/tencent/mm/plugin/shoot/a/l;

move-result-object v0

invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;getType()Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

move-result-object v1

invoke-virtual {v0, v1}, Lcom/tencent/mm/plugin/shoot/a/l;-&gt;c(Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;)V

.line 55
 :cond_2
 :goto_2
 sget-object v0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;-&gt;EXPLODING:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;setState(Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;)V

goto :goto_0

.line 52
 :cond_3
 invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;getType()Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

move-result-object v0

sget-object v1, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;-&gt;HERO:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

if-ne v0, v1, :cond_2

.line 53
 invoke-static {}, Lcom/tencent/mm/plugin/shoot/a/h;-&gt;Ml()Lcom/tencent/mm/plugin/shoot/a/h;

move-result-object v0

invoke-virtual {v0}, Lcom/tencent/mm/plugin/shoot/a/h;-&gt;MD()Lcom/tencent/mm/plugin/shoot/a/l;

move-result-object v0

invoke-virtual {v0}, Lcom/tencent/mm/plugin/shoot/a/l;-&gt;MT()V

goto :goto_2

###注意这里是我家的代码

invoke-virtual {p0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;getType()Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

move-result-object v0

sget-object v1, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;-&gt;HERO:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$Type;

if-ne v0, v1, :cond_11

###本段结束

.line 59
 :cond_4
 sget-object v0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;-&gt;HITING:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;setState(Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;)V

###注意这里是我家的代码
 :cond_11
 sget-object v0, Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;-&gt;FLIGTHING:Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;

invoke-virtual {p0, v0}, Lcom/tencent/mm/plugin/shoot/actor/GameSprite;-&gt;setState(Lcom/tencent/mm/plugin/shoot/actor/GameSprite$State;)V

###本段结束

goto :goto_1
.end method
[/java]

同理，需要给自己飞机多增加一些炸弹或者双子弹等都可以在palyer初始化的时候调用相应的方法，根据添加的方法写出对应的smali代码就能实现上面分析的无敌模式的多种情况 接下来我们需要进行打包，修改完代码后，用如下命令打包

[shell]
&lt;pre&gt;apk b 解包的包文件夹名称
[/shell]

这里需要注意的是，打包后必须进行签名，不然只对微信的apk包做签名处理是不起作用的。 签名命令如下：

[shell]
&lt;pre&gt;# 注意 update.zip 是需要签名的文件 update_signed.zip 是签名后的文件
java -jar signapk.jar testkey.x509.pem testkey.pk8 update.zip update_signed.zip
[/shell]

签名后把微信的apk安装包重命名为zip结尾，然后用压缩工具打开，替换里面相应的com.tencent.mm.plugin.shoot.XXXXXX
替换后对微信安装包重新签名即可，注意用adb安装程序的话不管什么结尾的文件名都行，要是copy到内存卡中进行安装必须改成apk不然识别不了。
这里提供一个改好的apk安装包，下载地址：[http://pan.baidu.com/share/link?shareid=1113420314&amp;uk=2317334154](http://pan.baidu.com/share/link?shareid=1113420314&amp;uk=2317334154)
注意：修改别人的软件是不合法的，所以这里提供的修改版本安装包我可没说是自己改的或许是从网上找的，大家懂得就行。

</div>
</div>
