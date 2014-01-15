title: "EasyUi 分页实现方法"
id: 117
date: 2012-07-11 23:59:14
tags: 
- easyui
categories: 
- 原创作品
- 技术文章
---

最近项目中使用到EasyUI的前端框架,分页我想基本上没有那个项目能够离开的，

结果今天悲剧的把官方文档翻了一边竟然没找到实例和说明。最终一番Google

最终找到方法:

```php

//自定义Grid分页数据表
function CreateGridPage(url,feildData){
 var title = arguments[2] ? arguments[2] : '数据表格';
 var tab = arguments[3] ? arguments[3] : '#tab';
 var id = &quot;gidi_&quot;+Math.floor(Math.random()*1000+1);
 $(tab).tabs('add',{
 id:id,
 title:title,
 closable:true,
 });
 CreateGrid(id,url,feildData,title,true);
}
//创建window窗口数据表格
function CreateGridWind(url,feildData,width,heigth){
 var id = &quot;gidi_win_&quot;+Math.floor(Math.random()*1000+1);
 var title = arguments[4] ? arguments[4] : '数据表格';
 $('body').append('&lt;div id=&quot;'+id+'&quot;&gt;&lt;/div&gt;');
 $('#'+id).window({
 title:title,
 width:width,
 height:heigth,
 modal:true
 });
 CreateGrid(id,url,feildData,null);
}
//创建带分页的数据表格
function CreateGrid(id,url,feildData,title,pagination){
 $('#'+id).datagrid({
 title:title,
 iconCls:'icon-save',
 url:url,
 idField:feildData.idField,
 singleSelect:true,
 striped:true,
 rownumbers:true,
 pagination:pagination,
 fitColumns:true,
 loadMsg:'亲~ 正在努力拉取数据中...',
 columns:[
 feildData.field
 ]
 });
 var page = $('#'+id).datagrid('getPager');
 $(page).pagination({
 showRefresh:false,
 beforePageText:'第',
 afterPageText:'页 共{pages}页',
 displayMsg:'每页 {to} 条 总共 {total} 记录',
 onSelectPage:function(num,size){
 $('#'+id).datagrid({
 queryParams:{
 page:num,
 size:size
 }
 });
 }
 });
}
```

上面代码是自己写好的两个，一个是创建在tab组件中显示Grid表格，另外一个是创建弹出window窗口中显示Grid表格
<!--more-->
注意：

>queryParams  参数是发送给服务的的参数，把page和size定义在这个后台就利用这个进行分页，和普通分页类似


分页代码：

```javascript
$(page).pagination({
showRefresh:false,
beforePageText:'第',
afterPageText:'页 共{pages}页',
displayMsg:'每页 {to} 条 总共 {total} 记录',
onSelectPage:function(num,size){
$('#'+id).datagrid({
queryParams:{
page:num,
size:size
}
});
}
});
```
