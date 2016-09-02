---
title: Nougat多窗口支持
date: 2016-08-29 17:12:18
tags: Android
comments: true
categories: Android
---
在Android7.0中引入了类似PC上多窗口的支持，屏幕上能同时显示多个，用户在与另一个应用交互的同时可以继续播放视频。如果是用N Preview SDK构建应用,则可以配置应用的多窗口显示方法。
<!--more-->
## 一、多窗口预览

 **1. 预览图如下：**

![预览图](http://img.blog.csdn.net/20160826145421231)
 
 **2. 用户切换到多窗口模式的方法：**
 
 - 若用户打开 Overview 屏幕并长按 Activity 标题，则可以拖动该 Activity 至屏幕突出显示的区域，使 Activity 进入多窗口模式。
 - 若用户长按 Overview 按钮，设备上的当前 Activity 将进入多窗口模式，同时将打开 Overview 屏幕，用户可在该屏幕中选择要共享屏幕的另一个 Activity。

***
## 二、多窗口模式配置
在清单文件（AndroidManifest.xml）中可以对应用的Activit是否支持多窗口显示以及显示方式进行配置。

 **1. 配置是否支持多窗口：**

在清单文件的 	&lt;activity&gt;  或  &lt;application&gt; 节点中设置该属性，启用或禁用多窗口显示：

```
android:resizeableActivity=["true" | "false"]
```
如果该属性设置为 true，Activity 将支持多窗口模式。 如果此属性设置为 false，Activity 将不支持多窗口模式,且用户尝试在多窗口模式下启动 Activity，该 Activity 将全屏显示。如果是用Android N SDK构建应用，但未对该属性指定值，则该属性的值默认设为 true。

 **2. 对多窗口显示方式进行配置：**

Android N 在清单文件的 &lt;Activity&gt; 节点下增加了一个 &lt;layout&gt; 标签。该标签支持一下几种属性：

```
android:defaultWidth //多窗口模式下Activity 的默认宽度
android:defaultHeight //多窗口模式下Activity 的默认高度
android:gravity //多窗口模式下Activity 的初始位置
android:minHeight //多窗口模式下Activity 的最小高度
android:minWidth //多窗口模式下Activity 的最小宽度

/*ps: android:minimalWidth 已经被替换成 android:minWidth
      android:minimalHeight 已经被替换成 android:minHeight 
	如果还是使用：android:minimalHeight 和 android:minimalWidth 编译不会通过。
*/
```
例如：以下节点显示了如何指定Activity在多窗口模式中显示的大小、位置和最小尺寸：

```
<activity
       android:name=".MyActivity"
       android:resizeableActivity="true">
       <layout 
	       android:defaultHeight="500dp"
           android:defaultWidth="600dp"
           android:gravity="top|end"
           android:minHeight="450dp"
           android:minWidth="300dp" />
   </activity>
```

**3. 多窗口模式禁用的功能：**

 - 无法隐藏状态栏
 - 系统将忽略对 android:screenOrientation 属性所作的更改

**4. 多窗口模式下常用的方法：**

 - Activity.isInMultiWindowMode()； 
  调用该方法以确认 Activity 是否处于多窗口模式。
 - Activity.onMultiWindowModeChanged()；  
 Activity 进入或退出多窗口模式时系统将调用此方法。 在 Activity 进入多窗口模式时，系统向该方法传递 true 值，在退出多窗口模式时，则传递 false 值。

**5. 在多窗口模式中启动新 Activity**
如果当前应用处于多窗口模式，想要在该应用启动的Activity显示在当前 Activity 旁边，则需要使用标志位：Intent.FLAG_ACTIVITY_LAUNCH_ADJACENT 和 Intent.FLAG_ACTIVITY_NEW_TASK
例如：

```
Intent intent = new Intent();
intent.setClass(this, MyActivity.class);
intent.setFlags(Intent.FLAG_ACTIVITY_LAUNCH_ADJACENT|Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);        
```
启动后两个Activity处于分屏模式，如图：
![启动新Activity](http://img.blog.csdn.net/20160828155812448)

如果处于多窗口模式下，可以通过调用 ActivityOptions.setLaunchBounds() 指定新 Activity 的尺寸和屏幕位置，如果不处于多窗口模式则该方法无效。
示例代码：

```
 Rect rect = new Rect(0,400,800,1000);
 ActivityOptions options = ActivityOptions.makeBasic();
 options.setLaunchBounds(rect);
 
 Intent intent = new Intent();
 intent.setClass(this, MyActivity.class);
 intent.setFlags(Intent.FLAG_ACTIVITY_LAUNCH_ADJACENT|Intent.FLAG_ACTIVITY_NEW_TASK);
 
 ActivityCompat.startActivity(this,intent,options.toBundle());
```
***
## 三、多窗口的生命周期
多窗口模式下Activity的生命周期并没有改变，只是在切换到多窗口和退出多窗口模式时有所不同。在这两种切换过程中Activity会被销毁后重新创建，所以在这两种模式下应该通过：onSaveInstanceState 和 onRestoreInstanceState 这两个方法做好数据的保存和恢复。

> 如果在多窗口模式下播放视频，应该在 onStop() 和 onStart() 这两个方法中去暂停和继续播放，这样会有更好的用户体验。

***

## 四、多窗口模式下数据拖放

多窗口模式下的拖放，仅仅是数据的拖放，并不能将一个Activity的View拖放到另一个Activity。跨Activity拖放需要：View.DRAG_FLAG_GLOBAL 这个标志。
例如从MainActivity拖放一个按钮中的数据到MyActivity中：

MainActivity:
```
btn = (Button) findViewById(R.id.my_button);
        btn.setOnLongClickListener(new View.OnLongClickListener() {
            @Override
            public boolean onLongClick(View view) {
                ClipData data = ClipData.newPlainText(view.getClass().getName(),
                        ((Button) view).getText());
                View.DragShadowBuilder builder = new View.DragShadowBuilder(view);
                view.startDragAndDrop(data, builder, view, View.DRAG_FLAG_GLOBAL);//跨Activity拖放需要这个标志位
                return true;
            }
        });
```
MyActivity：

```
textView = (TextView) findViewById(R.id.text);
View parent = (View) textView.getParent();
 parent.setOnDragListener(new View.OnDragListener() {
     @Override
     public boolean onDrag(View v, DragEvent event) {
         switch (event.getAction()) {
             case DragEvent.ACTION_DROP:
                 ClipData.Item item = event.getClipData().getItemAt(0);
                 textView.setText(item.getText());
                 break;
         }
         return true;
     }
 });
```
实现效果图：
![多窗口拖放](http://img.blog.csdn.net/20160828190400237)




