
---
title: Hexo+github搭建个人博客
date: 2016-01-14 10:43:11
description: 这篇文章主要讲解怎么利用Hexo和github来搭建自己的博客！如果你正想搭建一个属于自己的博客！
comments: true
banner: http://i.imgur.com/aBvSCNs.jpg
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这篇文章主要讲解怎么利用Hexo和github来搭建自己的博客！如果你正想搭建一个属于自己的博客，那么就不妨动手跟我一起来搭建 ！
<!-- more --> 

### 一.环境搭建    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在搭建之前我们首先要搭建环境，首先要做的是安装[<font color=#0000FF>Git</font>](http://git-scm.com/download/)和[<font color=#0000FF>Node.js(最好翻墙后打开)</font>](http://nodejs.org/)，先去这两个网站官网下载并安装好，安装的过程很简单，就是一路下一步即可！其次要做的是打开你的**git Bash**窗口，输入如下命令： 
```bash
$ npm install -g hexo-cli
```
完成以上操作后，你就算将**Hexo**也安装好了，安装**Hexo**也可以参看[<font color=#0000FF>Hexo的官网</font>](https://hexo.io/zh-cn/docs/)。
至此，环境搭建已经完成！  
***
### 二.创建本地博客  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在本地磁盘新建一个文件夹，我这里命名为：MyBlog，然后鼠标右击该文件夹，打开：**Git Bash Here**。在弹出的命令窗口中输入以下命令:  
```bash
$ hexo init yourname.github.io
```
需要**注意**的是：这里需要将命令中的**yourname**替换成你的github的账户名，必须要是和账户名一致！
执行完这条命令后就会在你刚才新建的文件夹下生成一个名字为：yourname.github.io的文件夹。然后，又执行如下几条命令：  
```bash
$ cd yourname.github.io  //cd到刚才自动创建的目录下
$ npm install
$ hexo g
$ hexo s
```
执行完以上命令后，不要关闭**Git Bash**窗口，用浏览器打开：[<font color=#0000FF>http://localhost:4000/</font>](http://localhost:4000/) 你就会看见自己的本地博客页面了，如图：
![](http://i.imgur.com/s3erhTc.png) 
***
### 三.将本地博客部署到github上  
##### 1. 在github上创建一个仓库
按照下图创建：
![github上创建仓库.png](http://upload-images.jianshu.io/upload_images/1440183-2c63ea8a268dfda5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
填写完成，点解**Create repository**完成创建即可！  
##### 2. 将刚创建的远程仓库更新下来
步骤如下：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;依然是鼠标右击：yourname.github.io打开**Git Bash Here**,执行如下命令：
```bash
$ git init    //初始化git仓库
$ git remote add origin git@github.com:yourname/yourname.github.io.git  //将yourname替换//给远程仓库添加一个名为：origin 的引用
$ git fetch origin   //获取远程仓库的内容
$ git merge origin/master //将获取的内容合并到master分支
```
##### 3. 修改配置文件  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接着用[<font color=#0000FF>sublime</font>](http://www.sublimetext.com/)或者[<font color=#0000FF>notepad++</font>](https://notepad-plus-plus.org/)打开：yourname.github.io文件夹下的这个文件：**_config.yml**，在文件最后做如下修改：
```
deploy:
  type: git
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master
```
**以下两步是有自己域名的看：**  
如果你有自己的域名，可以将上面修改如下：  
```
deploy:
  type: git
  repo: git@github.com:tiandawu/tiandawu.github.io.git
  branch: master
  plugins: -hexo-generator-cname
```
另外还需要修改的是这里：
```
url: http://tiandawu.com       //这里改成自己的域名
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```
好了，至此配置文件修改完毕。
##### 4. 部署到github  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;依然来到刚才的**Git Bash**窗口，执行如下命令：
有自己域名的需要先执行这条命令：
```bash
$ npm install hexo-generator-cname --save  //这条命令执行过程较慢，耐心等待！(需要翻墙快)
```

下面的命令都需要执行：
```bash
$ git add .
$ git commit -m "输入自己的描述信息"
$ git push origin master
```
执行完上面的命令，没有配置自己域名的再执行下面两条命令就可访问自己的博客了：(在浏览器中输入：`yourname.github.io`访问)
```bash
$ hexo g
$ hexo d
```
***
### *注意：*
在执行：`$ hexo d`的时候如果出现下面的错误：
![发布出错.png](http://upload-images.jianshu.io/upload_images/1440183-7cb3c497fd382d00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
这时候只需要先执行这条命令:
```bash
$ npm install hexo-deployer-git --save
```
然后后再执行下面这条部署命令就行了！ 
```bash
$ hexo d
```
配置了自己域名的还需要执行完下面的命令：
```bash
$ git fetch origin
$ git merge origin/master
```
配置了自己域名的还需要修改**CNAME**文件，public文件夹下的**CNAME**也要修改，都填写上自己的域名即可！可以参考我填写的[<font color=#0000FF>**CNAME文件**</font>](https://github.com/tiandawu/tiandawu.github.io/blob/master/CNAME)。  
配置了自己域名的最后执行下面命令也能访问了：（可以通过：`yourname.github.io`或`你的域名`访问）
```bash
$ git add .
$ git commit -m "输入自己的描述信息"
$ git push origin master
$ hexo g
$ hexo d
```
不管你有没有自己的域名，现在如果你能通过以上两种方式看到这个页面就算你搭建成功了（恭喜！）：

![Ok.png](http://upload-images.jianshu.io/upload_images/1440183-019d0909d2027364.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

***
### 四.配置Next主题  
Hexo提供了很多的主题，可以参考[<font color=#0000FF>丰富多彩的Hexo主题</font>](https://hexo.io/themes/)，这里我选用的是配置Next主题，[<font color=#0000FF>我的博客样式</font>](http://tiandawu.github.io/)。具体配置详情，请参考：[<font color=#0000FF>Next主题官方文档</font>](http://theme-next.iissnan.com/)。  

***
### 五.搭建遇到的坑  
我在将主题设置成Next后，在将仓库向github提交了一遍，结果就收到一封这样的邮件：
![erro info.png](http://upload-images.jianshu.io/upload_images/1440183-be87e8829b5b2cee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

于是我就点了链接，看到给的解决方案是这样的：  

![解决方案.png](http://upload-images.jianshu.io/upload_images/1440183-821266a9450dba75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
于是我就按照这个方法做，结果出现下面的错误：

![submodule error.png](http://upload-images.jianshu.io/upload_images/1440183-3282635780f355e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

经过几番尝试，我将之前更新下来的主题全删掉，然后输入下面的命令去下载主题：
<pre><code>git submodule add https://github.com/iissnan/hexo-theme-next</code></pre>
如图：  

![添加submodule.png](http://upload-images.jianshu.io/upload_images/1440183-28ddd196af0bd752.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

这样更新下主题后，发现在：`yourname.github.io`文件夹下会多出一个**.gitmodules**的文件，然后我在按照之前的方法尝试，结果就不出现那个错误，将文件push到远程仓库也不会收到刚才那邮件了，这个坑就这样被填上了！