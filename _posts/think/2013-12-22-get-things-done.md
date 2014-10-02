---
layout: post
title: 记一次发散性思维与时间管理的冲突
category: think
description: 周末的一次总结，过程中不断扩展进入新的知识点，一方面离要做的事越来越远，别一方面确实也学到了很实用且能提高效率的新知识，在学习技术的路上不只一次遇到这种情况，这也许正是为什么搞it的人大多有拖延症，计划做A事，但是不可逼免得要去做b事，两方面原因：一方面可能是因为做完b事才能继续做a事，有依懒关系；另一种可能是做完b事，再做a事效率会更高，正所谓磨刀不误砍柴功。但是计划总是赶不上变化啊，原计划一上午做完的事，最终搞了两天。还好是周末，有充足的时间来做扩展和细分；如果是工作时间，那就得在完成任务的前提下就行适当的扩展了。
---

@(blog)
> 周末的一次总结，过程中不断扩展进入新的知识点，一方面离要做的事越来越远，别一方面确实也学到了很实用且能提高效率的新知识，在学习技术的路上不只一次遇到这种情况，这也许正是为什么搞it的人大多有拖延症，计划做A事，但是不可逼免得要去做b事，两方面原因：一方面可能是因为做完b事才能继续做a事，有依懒关系；另一种可能是做完b事，再做a事效率会更高，正所谓磨刀不误砍柴功。但是计划总是赶不上变化啊，原计划一上午做完的事，最终搞了两天。还好是周末，有充足的时间来做扩展和细分；如果是工作时间，那就得在完成任务的前提下就行适当的扩展了。

## 原计划

九点起来，喝杯热水，按照时间管理的计划，周末用来整理和总结一周所查所学所思，准备把这周开发ERP中遇到[关于Ext4的表格排序功能][1]的问题总结写成博文，计划用时一上午。

![alt erp][2]

## wp-markdown

正提指时，突然想起要在wp的后台上直接写文章体验肯定会很差，而且不能同时在本地保存一份，预览也不方便，mac下的**MarsEdit**虽然能够同步服务器上的博文，但在格式编缉和跨环境通用性上也不实用，联想到最近看到的一种文档格式：**MarkDown**，这是个好方法，之前忙一直没空去了解，何不用点时间学习下呢，于是股沟一下，"wordpress markdown"，搜到“墙外的梯子”的一篇文章，介始了一款叫**wp-markdown**的插件。

![alt MarsEdit][3]

## markdown语法

wp-markdown这款插件很好用，安装和设置也很方便，但想高速无障碍的写博文，还得把markdown的语法记牢，继续股沟，查到了wowUbuntu的一篇“markdown语法说明”，介绍得详细且易懂，文章最后还推荐了各平台的markdown免费编缉器。

## markdown编缉器mou

选择mac平台下的**mou**，官方网站的简介中的一段话吸引了我：

> “When current available Markdown editors are almost all for general writers, Mou is different: It's for web developers…I know, it's exactly the app you want.”

软件简介很风趣啊，还有很实用的快捷键和皮肤，真是写博文的神器啊。很快得看完了mou的使用说明，不过注意到作者文章中的截图风格一致且很美观，我的兴趣又来了。

![alt mou][4]

## mac截图

这回先不股沟了，先自己思考下。不会是截窗口然后用ps加的阴影吧，不行，这样即使使用动作，效率也很低；那不会是截图时把桌面背景调成白色的吧，也不对，同样效率很低下，而且从网站中的源图片可以看出是png透明的。那还有一种可能是截图软件自带有给截图加阴影的功能，像win下的**winsnap**这款软件就是可以设置阴影等效果。查了几款mac下的截图软件，发现都有同样一个问题：mac软件窗口都是圆角的，这些截图软件截出的图四角都有黑色的色块，相信对于追求完美的mac用户而言这点小色块也是不能忍受的。自己尝试无果，还是股沟一下吧，有答案了，什么工具也有不用，使用mac自带的截图快捷键就可以换定了，com+shift+4，再接空格键，出现一个小相机图标后选择想要截图的窗口就可以了，good，不但可以截取叠在下面的窗口，而且也自带了阴影，非常完美，想来正是mou官网上使用的方法吧。大功告成。

![alt radius][5]

## web图片选择

还不能完，做为关注性能和优化的前端工程师而言，要思考的还不只这些。mou官网到是可以直接使用png图片无可厚非，但是做为博文配图，带阴影的png24图片显示太大了，何不使用css3来模似阴影，那就只需要截取到窗口图片就形了，继续股沟，mac果然有这样的截图设置选项，

1.  打开“**终端**”（应用程序－>实用工具文件夹），输入以下命令: `defaults write com.apple.screencapture disable-shadow -bool true`

    PS：如想恢复阴影，则输入:

    `defaults write com.apple.screencapture disable-shadow -bool false`

2.  重启 SystemUIServer ，同样在终端中输入以下命令：

    `killall SystemUIServer`

现在使用Mac截图功能截取出来的图片就不带阴影了。而且窗口圆角也有透明的。

 [1]: http://webfing.github.io/ext4-grid-sort/
 [2]: https://app.yinxiang.com/shard/s1/sh/d5b4e4fa-e327-46a9-b31c-a8309fb20615/7e39e2517aabe032310df67014d4d477/res/fc8ece06-c225-47b4-833b-75f5eec1a80b/erp.jpg?resizeSmall&width=832&alpha=
 [3]: https://app.yinxiang.com/shard/s1/sh/d5b4e4fa-e327-46a9-b31c-a8309fb20615/7e39e2517aabe032310df67014d4d477/res/a0cb82f1-f1c6-4f61-95ee-5cd49fcff3c7/marsedit.png?resizeSmall&width=832&alpha=
 [4]: https://app.yinxiang.com/shard/s1/sh/d5b4e4fa-e327-46a9-b31c-a8309fb20615/7e39e2517aabe032310df67014d4d477/res/cc1a33f8-50b5-4d4a-b348-54fcb55b9ba7/mou.png?resizeSmall&width=832&alpha=
 [5]: https://app.yinxiang.com/shard/s1/sh/d5b4e4fa-e327-46a9-b31c-a8309fb20615/7e39e2517aabe032310df67014d4d477/res/18c7ddfe-8069-4164-a305-3b39294b596a/radius.jpg?resizeSmall&width=832&alpha=

