---
date : 2017-11-26T13:47:08+02:00
tags : 
  - osx
  - 流氓广告
title : osx 解决右上角通知广告问题
sulg: osx_not_ad
---
# osx 解决右上角通知广告问题

> 新入了mac，安装视频播放软件时候，遇到了流氓软件MplayX,这个软件有官网，我是从官网下载。安装打开后，它让你在线安装，然后失败。

![MplayX](http://upload-images.jianshu.io/upload_images/2197548-d623c5376de7dcce?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

之后，你的mac就会有推送广告了。在右上角弹出通知，自动用浏览器打开appstore的软件页面。

![通知广告](http://upload-images.jianshu.io/upload_images/2197548-128eed03ae85d4c9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通知栏截图来自于网友

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2197548-9a709c2abb798d07?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


----------



> 我在网上查询此问题的解决办法，并没有得到结果。咨询苹果客服，给出的解决办法是新建一个用户，在删除当前用户。以后使用新的用户。我并不喜欢这种方式。
> 在研究半天后，发现了该流氓的实现原理。 该流氓的实现方式：添加一个mac的定时任务，到执行的时间立即执行一个py的命令。



## **现在给大家列出这个问题的解决办法**
先记下让下载的软件名字。
我的这个是：The Sims 2:Super Collection
不同人的安装，被装上的软件名字是不一样的

### **打开终端** 

``` shell
# 进入用户自定义命令
cd   ~/Library/LaunchAgents/
ls
```

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2197548-6aa8b8eb26c583af?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
能看到显示出几个文件，第一个名字和弹出的软件The Sims 2:Super Collection是一样的。
### **现在卸载执行的流氓任务**
``` shell
# 卸载定时任务
launchctl unload com.asoffertest.the-sims-2.agent.plist
```
### **现在通知广告就被关闭了。接下来是清除流氓的垃圾文件**
打开任务文件，查看命令是存放在哪里的

``` shell
# 查看文件
cat  com.asoffertest.the-sims-2.agent.plist
```

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2197548-cf2c61d472d1ebea?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

能看到有一个路径指向了一个.py的文件。
/Users/yihy/Library/Application Support/com.asoffertest.the-sims-2/asoffer.py

### **删除流氓的目录**
``` shell
# 删除流氓创建命令的目录
rm –rf '/Users/yihy/Library/Application Support/com.asoffertest.the-sims-2'
# 删除定时任务的文件
rm –f com.asoffertest.the-sims-2.agent.plist
```
**现在流氓已经滚蛋了。**
