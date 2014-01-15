title: "android最近问题总结 "
id: 296
date: 2013-07-25 14:27:53
tags: 
- android
categories: 
- 原创作品
- 技术文章
---

1\. 主线线程中更新UI
由于android的限制，不允许在主线程中对UI进行修改操作，必须要再新线程中。其实这点和lpython 的pyGtk很相似，估计都同为linux的产物。
自己更新UI的方式：

```java
//先声明一个Handler，用来接收消息
private Handler messageHandler;
//消息标识
private finalist UPDATE_UI = 1;

messageHandler = new Handler() {
@Override
public void handleMessage(Message msg) {
    if (msg.what == UPDATE_UI) {
        updateUI(); //更新UI的方法
    }
}

```
<!--more-->

其中updateUI()方法中写自己需要更新的方法就行
那么如何触发？ 当然是发送消息了~
在需要更新的操作地方直接发送消息给handler就可以更新UI操作了

```java
messageHandler.sendEmptyMessageDelayed(UPDATE_UI, 1000);
```

2.有时候在2.3的系统上测试正常的代码发放到4.0的系统上直接报网络错误，其实从3.0开始android是不允许在主线程中直接进行耗时操作如网络访问，所以还是需要在线程中进行网络操作。
自己用到的方式：

```java
//先声明一个Runnable，然后实现它的Run（）接口,在Runnable里面写自己的业务逻辑，如网络访问
Runnable loginRunnable = new Runnable() {
@Override
public void run() {
//这里书写业务逻辑
}};
//启动个新线程执行Runnable

new Thread(loginRunnable).start();
其实自己用的最多的还是定义相关操作类，继承Runnable并且实现它的run接口
还有一个Runable错误具体忘记了，不过在run开始前和结束前加上这两个方法就OK
//开始前
Looper.prepare();
//结束前
Looper.loop();
```

3.json操作，android自带json处理方法，自己用的最多的是解析字符串json为对象。具体处理方法如下：

```java
String jsonStr = &quot;{status:1, username:'creturn'}&quot;;
//定义一个JSONToken加载json字符串
JSONTokener jsonTokener = new JSONTokener(jsonStr);

//定义json对象，从TOkenr中取得json
JSONObject jsonObject = null;

//使用此方法可取得json对象
jsonObject = (JSONObject) jsonTokener.nextValue();
例如要获取username可以这样获取
jsonObject.getString(&quot;username&quot;)

```

4.android里面读取assets目录下的文件
在activity里面或者能够访问到context的地方都行，最终用到里面的一个方法是：
getResources()

它会返回一个资源访问类，在里面有诸多访问资源的方法，我们用到的是里面的
getAssets()

用词方法里面会返回一个assetsManger类，通过assetsManger类里面的open方法就可以访问并返回一个InputStream
这样就可以对输入流进行操作了。具体代码如下：

```java
InputStream is = _conContext.getResources().getAssets().open(LOGIN_VIEW_BACKGROUND);
```

其中—conContext是context类如果在activity里面就可以直接使用getResources方法
5.androi里面的layout纯代码操作，由于最近有一个项目比较特殊，需要打包成Jar给别人使用，但是又不能把过多的内容暴露在外面，
所以传统的在xml布局文件中得方法不得不放弃，采用纯代码写布局。
layout的操作，例如设置其参数默认layout里面没有的方法，那么就需要用到相应的layoutParms类来实现，例如给一个layout设置布局为宽度大小根据内容而定，高度一样根据内容来自适应那么代码就如下（以linnerlayout为例）：

```java
//定义linearlayoutParms参数
LinearLayout.LayoutParams layout_params = new LinearLayout.LayoutParams(
LinearLayout.LayoutParams.WRAP_CONTENT,

LinearLayout.LayoutParams.WRAP_CONTENT);
//如果需要LinearLayout居中显示可以设置其gravity
layout_params.gravity = Gravity.CENTER|Gravity.CENTER_HORIZONTAL|Gravity.CENTER_VERTICAL;

//设置参数
LinearLayout.setLayoutParams(layout_params);
```

6.纯代码绘制view的背景，并用指定图片进行平铺操作
首先用指定代码进行平铺那么就需要读取图片资源，就可以用到上面介绍的方法读取一个inputstream

```java
InputStream is = _conContext.getResources().getAssets().open(&quot;图片地址&quot;);
```

注意这里的图片是放在assets目录下面
由于所有的view最终不管你是用什么方式修改其背景，最终都会转换成drawble进行绘制，所以吧inputstream转换成一个drawable就行，代码如下：

```java
//从inputstream里面生成BitMapDrawable
BitmapDrawable bitmapDrawable = (BitmapDrawable) BitmapDrawable.createFromStream(is, null);
//这里设置其位纵横repeat也就是平铺
bitmapDrawable.setTileModeXY(TileMode.REPEAT, TileMode.REPEAT);
//设置背景

setBackgroundDrawable((Drawable) bitmapDrawable);
```

7\.  EditView编辑框中怎么定义其只允许输入指定的内容 ？ 其实如果是xml布局的话很简单，只要设置它的一个参数digits参数就行
但是悲剧的我只能用代码，那么怎么处理，其实不管是任何一个平台或者相应的和GUI有关的开发，第一个应该想到的就是它的事件，肯定有有关change的事件。在事件发生的适合判断输入的内容是否在允许范围内就行。代码如下：

```java
txt_username.addTextChangedListener(new TextWatcher() {
String tmp = &quot;&quot;; //只允许输入的字符
String digits = &quot;@._0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLIMNOPQRSTUVWXYZ&quot;;
@Override
public void onTextChanged(CharSequence s, int start, int before, int count) {
}
@Override
public void beforeTextChanged(CharSequence s, int start, int count,int after) {
}
@Override
public void afterTextChanged(Editable s) {
String str = s.toString();
if(str.equals(tmp)){
return;
}
StringBuffer sb = new StringBuffer();
for(int i = 0; i &amp;lt; str.length(); i++){
if(digits.indexOf(str.charAt(i)) &amp;gt;= 0){
sb.append(str.charAt(i));
}
}
tmp = sb.toString();
txt_username.setText(tmp);
txt_username.requestFocus();
txt_username.setSelection(tmp.length());
}

});

```

8\. WebView 在制作sdk过程中其中一个过程需要用到webview，这个webview就是用来访问http。但是使用过程中也碰到一些问题，比如在实例化一个WebView的时候，通常根据他它的参数提示，创建的时候就直接传递Context就行，但是，如果用这样创建的webview来访问http的页面，当再次点击里面的链接的时候就直接报错，所以解决方法就行hi生成webview的时候把当前activiy传递进去，大妈如下：

```java
WebView webView = new WebView(this);
```

但是新的问题由来了，虽然不报错了可是所有打开的链接都是通过默认浏览器来执行了，这个显然不是我要的结果，解决方法就是设置一个WebViewClient进行相应的方法重写来实现，代码如下：

```java
//获取webview的setting
WebSettings webset = webView.getSettings();
//启用javascript的支持，不然js就不起作用了
webset.setJavaScriptEnabled(true);// 添加web view client ,监听里面的事件webView.setWebViewClient(new WebViewClient() {
@Override
publicboolean shouldOverrideUrlLoading(WebView view, String url) {// 如果是自定义事件
if (url.startsWith(&quot;http://msg_&quot;)) {
// 自定义事件处理}return super.shouldOverrideUrlLoading(view, url);
}

}
```

其实只要重写shouldOverrideUrlLoading方法不做任何处理就能够在你的web view里面访问新的链接，这里面的url是每次访问的任何一个自由的url地址。由于自身项目原因我这里需要自定义事件，其实就是根据url地址栏的变化自定义自己的消息。这里需要注意的是之前用过python写webkit自定义时间开头用的都是"event://"来自定义时间，但是如果是服务器返回event://来触发时间那么你的愿望将要落空，因为event://服务器是不认的web服务器是泡在http协议上的所以event://是不会返回给客户端的。了解了这个原理，现在的那些神马基于webkit的GUI用html写跨平台APP还有那么神秘吗？

9\. android模拟器中如何访问本机的web服务器？

由于需要和服务器进行通信并进行数据交互，但是在模拟器中肯定不能通过127.0.0.1来进行访问了，其实有两种方法
第一种，直接访问本机内网IP地址就行
第二种， 10.0.2.2 是保留地址，供android模拟器访问本地网络
10\. 按home见后退出activity并重新加载了？ 有时候我们切换应用的适合难免按到home不过如果不做任何设置，从新进来的activity是重新启动吃的初始的activity。
解决方法：

```java
androidmanifest中的actvity中加入一个参数android:launchMode=&quot;singleTask&quot;
```

11\. 在一个非aictivy中如何启动一另外一个activity，并且实现回调？
当时这个问题真把自己难倒了，启动一个activity其实只要有context基本就可以，但是还要实现回调，因为是通过一个非activity来启动另外一个activity所以就不能用到常规的方法了。其实解决方法还是比价简单，只是当时自己没有想到。

解决方法就是定义一个单例，把回调方法塞到她里面去，单例在整个运行过程中始终保持单例，所以可以通过单例进行跳转一次来实现回调。代码如下：

```java
public interface CallbackListenner {

public void onSuccess(Bundle bundle);
public void onError(Bundle bundle);

}

class signalClass {
//回调接口
private CallbackListener;
//获取单例
public getInstance(){
// …..
}

publicvoid startLoginDialog(CallbackListenner callback) {
    CallbackListener = callback;
	Intent intent = new Intent(_context,JoyCenterActivity.class); intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK); _context.startActivity(intent);
	}

}
```

在A的activity里面用startLoginDialog启动另外一个activity就可以了。只要在B的activity里面执行

```java
signalClass.getInstance().CallbackListener.onsuccess(bundle);
```

就可以实现回调了
12\. android 中jqueryMobile page切换和dialog弹出时候抖屏或者白屏问题
加入下面代码即可：

```java
$(document).bind(&quot;mobileinit&quot;, function(){
	if (navigator.userAgent.indexOf(&quot;Android&quot;) != -1){
		$.mobile.defaultPageTransition = 'none';
		$.mobile.defaultDialogTransition = 'none';
	}
});

```

但这里需要注意的一点是，加入这段代码要在引入jqueryMobile之前，作用很明显了，就是绑定属性去掉过渡效果
