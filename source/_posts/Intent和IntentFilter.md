---
title: Intent和IntentFilter
date: 2016-09-02 17:35:27
tags: Android
comments: true
categories: Android总结
---
作为四大组件的枢纽，Intent和IntentFilter起着功不可没的作用，所以有必要好好做个总结。

<!--more-->

### 一、Intent

Intent分为两种类型：显式Intent、隐式Intent

- 显示Intent:指定要启动组件的名称(类的全路径名)

```
// Executed in an Activity, so 'this' is the Context
// The fileUrl is a string URL, such as "http://www.example.com/image.png"
Intent downloadIntent = new Intent(this, DownloadService.class);
downloadIntent.setData(Uri.parse(fileUrl));
startService(downloadIntent);
```
- 隐式Intent:不指定特定的组件，而是通过设置常规操作，如：`Action`、`Data`、`Category`

```
// Create the text message with a string
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType("text/plain");

// Verify that the intent will resolve to an activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(sendIntent);
}
```

在构造Intent的时候可以给这个Intent指定这些信息：
`Component name`、`Action`、`Data`、`Category`、`Extras`、`Flags`

①`Component name`:主要是用来指定组件名称，可以通过` setComponent()`、`setClass()`、`setClassName()` 或者直接通过Intent的构造函数来指定组件名称。
示例：
```
 Intent intent = new Intent();
 //Intent intent = new Intent(this, BJActivity.class);//通过构造函数指定组件名
 ComponentName component = new ComponentName(this, BJActivity.class);
 intent.setComponent(component);//通过setComponent()方法指定组件名
 //intent.setClass(this, BJActivity.class);//通过setClass()指定组件名
 //intent.setClassName(this, "com.tiandawu.nougat.BJActivity");//通过setClassName()指定组件名
 startActivity(intent);
```

② `Action`:主要用来指定Intent的动作，可以通过`setAction()` 来指定Intent的动作。
示例：
```
Intent intent = new Intent();
intent.setAction(Intent.ACTION_CALL);//拨打电话的动作
intent.setData(Uri.parse("tel:10086"));//携带的数据
this.startActivity(intent);
```
Action的种类很多，具体种类可以去官网的Intent类中查看。

③ `Data`:主要用来指定Intent携带的数据，可以通过`setData()` 来指定Intent携带的数据。例如上面的例子中就就通过`setData` 来指定携带了电话号码。

> setData()和setType()的注意事项：
> 当要指定data和type的时候请使setDataAndType()，因为用setData和setType()来分别制定data和type时，setData()和setType()都会清除对方的值。

④ `Category`:主要用于隐式Intent，用来指定类别，通过`addCategory()` 来指定类别。主要用途是用来匹配Intent-Filter中的&lt;category&gt; 标签中的元素。后面会在IntentFilter中作详细讲解。

⑤ `Extras`: 主要用于携带完成请求操作所需的附加信息的键值对，通过`putExtra()` 方法来存入键值对。
示例：
```
intent.putExtra("strKey", "strValue");//存入一个字符串键值对
intent.putExtra("intKey", 20);//存入一个int型的键值对
```
`putExtra()` 能存入很多基本数据类型，还可以put对象，通过这个方法就可以完成Activity之间的数据交互。

`Flags` 见名思意，可以用来携带标志，通过`setFlags()` 就可以指定所需要的标志。
示例：
```
Intent intent = new Intent();
intent.setClass(this, MyActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);//新开一个任务栈，来启动Activity
startActivity(intent);
```
能指定的Flags有很多，具体的Flags有哪些可以去官网的Intent类中查看。

***

### 二、IntentFilter

IntentFilter是用来过滤Intent的，当用隐式Intent来启动一个Activity的时候，就会匹配`Intent`携带的信息和`intent-filter` 中的过滤信息，当这些信息匹配成功则启动Activity。`intent-filter` 的过滤信息分为三类，分别是：`action类`、`category类`、`data类`。
一个&lt;Activity&gt; 节下面可以包含多个&lt;intent-filter&gt; 节，当一个Intent能匹配任何一个&lt;intent-filter&gt; 中的`action类`、`category类` 和 `data类` 时则匹配成功，此时才能启动目标Activity。
![这里写图片描述](http://img.blog.csdn.net/20160831202518465)

下面是一个Activity中的过滤规则示例：
```
<activity android:name=".TestActivity" >
    <!--第一组intent-filter-->
    <intent-filter>
        <action android:name="com.tdw.action_1" />
        <action android:name="com.tdw.action_2" />
        <category android:name="com.tdw.category_1" />
        <category android:name="com.tdw.category_2" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
        <data android:mimeType="video/*" />
    </intent-filter>
    <!--第二组intent-filter-->
    <intent-filter>
        <action android:name="com.tdw.action_group_2" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
```
在这个&lt;Activity&gt;中有两组&lt;intent-filter&gt;，只要一个Intent能匹配其中任意一组&lt;intent-filter&gt;即可。这里的匹配是指需要同时匹配一组中的action类别、categroy类别和data类别。
下面分别详细介绍这三个类别的匹配规则。

**1、`action` 的匹配规则**
从上面的例子中可以看出，一个`intent-filter`中可以有多个`action`，如果Intent中的action和过滤规则中action的字符串相同(区分大小写)，则匹配。如果`intent-filter`中有多个action，那么只要匹配上过滤规则中的一个action也算匹配成功。如果Intent中没有指定action或则与`intent-filter`中的action一个也没匹配上，则匹配失败。比如：Intent中action的值为`com.tdw.action_1` 或 `com.tdw.action_2`就算匹配成功，如果一个都不相同则匹配失败。

**2、`category` 的匹配规则**
`category` 的匹配规则和`action`的匹配规则有所不同。如果Intent中携带了`category`，不管有几个`category`，对于携带的每个`category` 来说，它必须是过滤规则中已经定义好了的`category`。但是Intent中也可以没有携带`category`，如果没有的话也可以匹配成功，因为系统在调用`startActivity` 或者 `startActivityForResult` 的时候会默认为Intent加上“android.intent.category.DEFAULT”这个`category`。与`action` 不同的是，`action` 是要求Intent中必须有一个`action` 且必须能够和过滤规则中的某个`action` 相同。

**3、`data`的匹配规则**
要了解`data` 的匹配规则首先要了解`data` 的语法，所以还是先来学习`data` 的语法。

`data` 的语法示例：
```
<data
   android:scheme="string"
   android:host="string" 
   android:por="string"
   android:path="string"
   android:pathPattern="string"
   android:pathPrefix="string"
   android:mimeType="string"/>
```
`data` 由两部份组成，mimeType和URI。mimeType指媒体类型，比如：`image/jpeg` 和 `video/*` 可以分别表示图片和视频。mimeType的媒体类型有很多，具体需要用时可以去查看。而URI中包含的数据就比较多了，URI的结构如下：
> &lt;scheme&gt;://&lt;host&gt;:&lt;port&gt;/[&lt;path&gt;|&lt;pathPrefix&gt;|&lt;pathPattern&gt;]

具体示例：
`content://com.example.project:200/folder/subfolder/et`
` http://www.google.com:80/search/info`

通过两个具体示例和URI的结构示例对照看就很清楚URI的含义了。 
***scheme:*** URI的模式，比如http、file、content等，如果URI中没有指定scheme,那么整个URI的其他参数无效，这也意味着URI无效。
***host:*** URI的主机名，比如：www.google.com，如果host为指定，那么整个URI的其他参数无效，这也意味着着URI无效。
***port:*** URI的端口号，比如：80，仅当URI中指定了scheme和host参数的时候port参数才是有意义的。
***path、pathPattern、和pathPrefix:*** 这三个参数表述路径信息，其中`path`表示完整的路径信息； `pathPattern`也表示完整的路径信息，大师它里面可以包含通配符`(*)`,  通配符`(*)` 表示0个或多个任意字符，需要注意的是，由于正则表达式的规范，如果想表示真实的字符串，那么`*` 要写成`\\*` ,`\`要写成`\\\`;`pathPrefix` 表示路径的前缀信息。

`data` 的匹配规则和`action` 类似，它也要求Intent中必须含有`data` 数据，并且`data` 数据能够完全匹配过滤规则中的某一个`data`。这里的完全匹配是指过滤规则中出现的`data` 部分也出现在了Intent中的`data` 中。

 - 示例1：

```
<intent-filter>
    <data android:mimeType="image/*" />
    ...
</intent-filter>
```
这个示例当中，mimeType指定的媒体类型是图片，那么Intent中的mimeType属性也必须为”image/*“才能匹配，这种情况下虽然没有指定URI，但是确是有默认值，URI的默认值为`content`和`file`。也就是说，虽然没有指定URI,但是Intent中的URI部分的`scheme` 必须为`content`或则`file`才能匹配。所以Intent中的`scheme` 可以这么写：

> intent.setDataAndType(Uri.parse("file://abc"),"image/*")

注意的是：如果要为Intent指定完整的`data` ，必须用`setDataAndType` 方法，不能先调用`setData` 再调用`setType` ,因为这两个方法在调用时会彼此清除对方的值。

 - 示例2：

```
<intent-filter>
    <data android:mimeType="video/mpeg" android:schema="http" .../>
    <data android:mimeType="audio/mpeg" android:scheme="http" .../>
    ...
</intent-filter>
```
这种规则指定了两组`data` 规则，且每个`data` 都指定了完整的属性值，既有URI又有`mimeType`。为了匹配示例2中的规则，可以在写出如下示例：

> intent.setDataAndType(Uri.parse("http://abc"),"video/mpeg")

或则是

> intent.setDataAndType(Uri.parse("http://abc"),"audio/mpeg")

- 示例3：
```
<!--第一种-->
<intent-filter>
    <data
        android:scheme="file"
        android:host="www.google.com" />
    ...
</intent-filter>
<!--第二种-->
<intent-filter>
    <data android:scheme="file" />
    <data android:host="www.google.com" />
    ...
</intent-filter>
```
这是 `data` 特殊情况，如上两种写法，作用是一样的。

***

***参考书籍：*** Android开发艺术探索
***参考网站：*** Android官方文档