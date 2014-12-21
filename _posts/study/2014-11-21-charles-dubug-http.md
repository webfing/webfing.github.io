---
layout: post
title: Mac下使用Charles调试http
description: 在wins下使用Fiddler加上插件Willow可以无痛调试http请求，但是在mac下呢？fiddler官网上虽然有提供mac版本的支持，但是基于跨平台的Mono框架开发的似乎不如原生的好
category: study
---

@(work)[http, charles]
在wins下使用Fiddler加上插件Willow可以无痛调试http请求，但是在mac下呢？
fiddler官网上虽然有提供mac版本的支持，但是基于跨平台的Mono框架开发的似乎不如原生的好

![](http://king-images.qiniudn.com/623DCF4B-FA04-46F1-BC58-88227630F076.jpg)

网上还有一种建议是通过虚拟机的代理来实现，在wins虚拟机里安装fiddler，然后通过设置代理，捕获mac下的请求来调试，但是如果想通过本地代理来调测代码时又无能为力了。

##Charles也是神器
习惯了fiddler界面及交互的同学可能一下不适用mac下的charles，但是深入了解它之后，你会发现它的好，下面从几个工作中常用的几点调试来熟习charles的使用

###代理本地代码
线上的静态资源都是经过加密压缩过的，通过代理映射到本地源代码来调试会非常高效。右键要代理的资源，选择【Map Local】弹出映射窗口，注意在path中可以通过*来指向一个文件夹

![](http://king-images.qiniudn.com/74B37770-3A52-47A3-AEAE-8870A6B04EAA.jpg)

![](http://king-images.qiniudn.com/53C89276-1473-406F-8E99-9FAC39FA7ADA.jpg)

###验证是否代理成功
选择要查看的请求，从右侧overview面板Notes字段可以看出是否有映射

![](http://king-images.qiniudn.com/947BD6CC-129C-4B76-9B56-5DF2E8B4A209.jpg)

###请求过滤
####捕获记录控制
捕获的请求太多，容易产生干扰，Charles可以对捕获记录进行过滤。
![](http://king-images.qiniudn.com/BC8A06FA-EB6F-4CEC-99B3-A149AD3F0BC8.jpg)
![](http://king-images.qiniudn.com/DC2E0513-593D-4C7E-A81D-A70D256EFD31.jpg)

####请求查找
Sequence面板展示的是每一条请求，可以在【Filter】输入想要的请求（支持正则表达式），相比fillder的命令过滤简单直观快速
![](http://king-images.qiniudn.com/6W0Q83BO85OCR1VP0.jpg)

###模拟慢速
在pc端很少提到慢速的优化，但在移动端，网络环境复杂多变，慢速的优化非常有必要，在charles中如何模拟呢？

慢速的标准是什么呢？先来看一下国内的网络环境

####国内网络环境
![](http://king-images.qiniudn.com/2957481_20120920141455.png)

按照`1Mbps=1/8MB/s`的换算比例：
2G平均速度25KB/s ，3G平均速度200kb/s， 4G平均速度1M/s

下面是iPhone4手机的wifi及3G网络下的测速
![](http://king-images.qiniudn.com/F52FCE4A-38DE-4882-BC01-AA92D5814A58.jpg)

####配置网速
菜单-》Proxy -》Throttle Settings

![](http://king-images.qiniudn.com/UXEA1U3K$FU~4NN6H.jpg)

在阀门窗口可以自己定义参数

![](http://king-images.qiniudn.com/0E59ED0E-7A6A-494C-8765-98C0C4B27A00.jpg)

####实践应用
上周游戏中心有一个loading改版的需求，重构的loading图（data.png）有25k，25k的loading图影响到底有多大呢？

2G模拟（25KB/s）下，平均下载耗时1.5s
![](http://king-images.qiniudn.com/E2097A03-FAD8-4093-9989-D38200115414.jpg)

3G模拟（200kb/s）下，平均下载耗时
![](http://king-images.qiniudn.com/ECB81839-A003-441D-9509-3A7E13299A34.jpg)

显然一张25k的loading图在2g网格环境下使用户的等待时间更长了，加上webview的打开时间及html的加载时间，用户很可能还没有看到loading图就已经失去耐心离开了

###加载分析
charles还可以对单个session（一个浏览tab）进行加载分析，图表展示非常直观
![](http://king-images.qiniudn.com/C2ED5E32-ED8D-48B3-A4AA-15EEB0F00B05.jpg)
![](http://king-images.qiniudn.com/F0116B60-3529-4391-84BD-13DA2305A4B5.jpg)

###断点调试
与后台CGI的调试中，我们需要测试接口的各种边界情况，比如出错、超时等表现，charles的断点+随意篡改，非常方便测试。

如图，要查看不同平台下cgi返回的数据，不需要切换真机，也不需要在浏览器一堆编码后的参数中查找想要的参数

![](http://king-images.qiniudn.com/9C24C2C7-98FD-4387-8BE8-AA96A76F5D76.jpg)