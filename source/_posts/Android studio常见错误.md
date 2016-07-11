title: 怎么解决Android studio导入项目卡死
date: 2016-01-16 17:29:24
tags: 踩过的坑
comments: true
banner: http://i.imgur.com/H5vlBf1.png
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在使用Android studio的时候常常遇到这样的问题，从github或是其他地方导入项目，Android studio呈现卡死的现象！当遇到这种情况时，可以看看是下面那种情况，在按照方法来解决！
<!--more-->
### 一、首次启动studio卡死
当我们安装完studio，首次启动时如果卡死在这个画面：    

![](http://i.imgur.com/jdKiv1t.png)  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这时，我们只要到android studio的安装目录的bin目录下去找这个文件：idea.properties 文件，在文件的最后追加这样一句话即可：**`disable.android.first.run=true`**  
另一种方法时翻墙让它自己去慢慢下载！
***
### 二、导入其他地方的项目卡死（很慢）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从github或其他地方导入项目如果studio很久也是像卡死的状态，这时你可以先将进程杀死，沿着这个路径：**`项目/gradle/wrapper`**找到这个文件： **`gradle-wrapper.properties`** ，将它打开看看这里： 
 
![](http://i.imgur.com/KDJQrId.png)  

* 一种方法是，去这个网站（**[<font color=#0000FF>Gradle Distributions</font>](http://services.gradle.org/distributions)**）下好对应的版本。或者去**[<font color=#0000FF>gradle官网（需要翻墙，下载还很慢）</font>](http://gradle.org/gradle-download/)**下载也可以！  
然后将下载好的这个文件（**gradle-2.10-all.zip**）拷贝到这个目录下： **`C:\Users\Administrator\.gradle\wrapper\dists
\gradle-2.10-all\a4w5fzrkeut1ox71xslb49gst`**  
#### **注：**
**a4w5fzrkeut1ox71xslb49gst**这个文件夹名是随机数所以很可能不是一样的，然后再导入项目！

* 另一种方法是，将上面那个文件的版本改成你自己那个目录有的版本，然后再导入项目。
***
### 三、studio升级后，新建项目卡死
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;今天在将studio 升级到最新版本后，新建了个醒目结果一直卡在这个界面：

![](http://i.imgur.com/cLtq9kE.png)  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我发现它又是去给我下载：**gradle-2.10-all.zip**去了，于是我按照上面的第二种方法修改了gradle的版本，然后再导入编译，结果又出现下面的问题：

![](http://i.imgur.com/FTZDR3B.png) 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后我就知道是gradle的版本高了，于是就到：项目的根目录找到**build.gradle**这个文件，做如下图的修改：![](http://i.imgur.com/jP1rHT9.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;修改后我又编译了一次，结果又冒出这样一个错误，如图：

![](http://i.imgur.com/2lu2bK9.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;于是我又找到我又打开app目录下的**build.gradle**打开发现:**`compileSdkVersion 'Google Apis:Google Apis:23'`**,最后将他做了如下图的修改：

![](http://i.imgur.com/o6gH25d.png)

这样项目才算是编译通过，能部署到手机上！

最后我又按照上面的前一种方法做了一遍，发现一次性解决问题，什么文件也不用修改！
***
### 四、最后总结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不管是导入项目还是新建项目卡死，发现它是去下载：**`gradle-xxx-all.zip`**的时候最好的解决方案就是按照上面的方法自己去下载对应的版本，然后放到那个目录下，这样做省去了修改这，修改那的麻烦！





