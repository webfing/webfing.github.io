---
layout: post
title: Viewport相关知识
category: think
description: 上周处理游戏中心轮播图时遇到了一个bug，使用`window.innerWidth`取的值在部分安卓机型下不准确，后来试尝使用`document.documentElement.scrollWidth`解决了，问题虽然解决了，但不清楚背后的原因，心中发虚，于是周末整理总结一番。
---

@(temp)[viewport, html5, webapp]
> 上周处理游戏中心轮播图时遇到了一个bug，使用`window.innerWidth`取的值在部分安卓机型下不准确，后来试尝使用`document.documentElement.scrollWidth`解决了，问题虽然解决了，但不清楚背后的原因，心中发虚，于是周末整理总结一番。

##背景
在过去大多数网页都是针对桌面显示器开发和测试的，随着移动浪潮的到来，原先设计的网页在相对小屏的移动端该如何展示呢？
最简单粗暴的两种兼容方式：

* 保持原有的网页大小，出现滚动条，能看到细节，但频繁滚动很麻烦
* 不出现滚动条，缩放页面，可以看到全局布局却看不清细节

两种方案的有缺陷，于是Apple发明了viewport的meta标签（现在几乎所有手机浏览器都支持），更加精准得控制web页面在移动端的展示

先来看一下veiwport有哪些属性值

	<meta name="viewport"
	    content="
	        height = [pixel_value | device-height] ,
	        width = [pixel_value | device-width ] ,
	        initial-scale = float_value ,
	        minimum-scale = float_value ,
	        maximum-scale = float_value ,
	        user-scalable = [yes | no] ,
	        target-densitydpi = [dpi_value | device-dpi | high-dpi | medium-dpi | low-dpi]
	    "
	/>

|属性|备注|
|:-|:-|
|width|	设置宽度，为一个正整数，或字符串"width-device"|
|initial-scale|	设置页面的初始缩放值，为一个数字，可以带小数|
|minimum-scale|	允许用户的最小缩放值，为一个数字，可以带小数|
|maximum-scale|	允许用户的最大缩放值，为一个数字，可以带小数|
|height|	设置layout viewport  的高度，这个属性对我们并不重要，很少使用|
|user-scalable|	是否允许用户进行缩放，值为"no"或"yes", no 代表不允许，yes代表允许|
|target-densitydpi| 目标设备的密度等级，作用是决定css中的1px代表多少物理像素|

##两个viewport
* visual viewport
    visual viewport就是当前显示给用户内容的窗口
* layout viewport
    layout viewport 有网页的所有内容，他可以全部或者部分展示给用户。

`window.innerWidth`对应visual viewport的大小
`document.documentElement.clientWidth`对应layout viewport的大小

![Alt text](http://king-images.qiniudn.com/QQ20150111-1.jpg)

**Q: 现在问题来了，既然layout viewport是网页的所有内容，那如果这个网页是按流式布局来处理的（没有固定宽度），那此时的layout viewport是多宽呢？**

A: 不同的设备、不同的浏览器结果大同小异，但有一点肯定的是它的值是固定的，不会因为网页内容的变化而变化。对于css布局，特别是用宽度百分比做排版的时候，比率是正是按照layout viewport 来计算的。

**Q: 那用户改变视图的缩放是改变了谁的大小呢？**
A: visual viewport。前面说到对于css布局，相对的依据是layout viewport大小来计算的，如果视图的缩放是通过改变layout viewport的大小来起作用的，那缩放的过程中势必会引起重绘。

![Alt text](http://king-images.qiniudn.com/300958475557219.png)

总结：layout viewport是真正网页渲染视图的大小、每个系统的不同浏览器都有自己的固定大小，不会随内容而改变；visual viewport好比查看网页视图的放大镜，用户交互产生的缩放比例就是这个放大镜的缩放比例。

##width
|属性|备注|
|:-|:-|
|width|	设置宽度，为一个正整数，或字符串"width-device"|

**Q: 这个width属性映射谁的宽度呢？**
A: layout viewport。

**Q: 即然不同浏览器都有一个固的layout viewport，为什么还要设置一个width属性来控制它的大小呢？**
A: 这个固有的layout viewport值是为了迎合传统pc端960栅栏布局的页面，使这些页面不至于出现布局错乱，但是缩放的实现方式牺牲了页面的可读性。那如何想设计一个真正完美适配移动端的web页面呢，答案就是手动控制layout viewport的大小。以iphone4为例，他的layout viewport宽度是980px，打开一个没有经过处理的web页面时，页面会按照最大980px的宽度来渲染，在宽度只有320px的界面上就会出现滚动条，出于交互体验，iphone自动进行了缩放，使980px的内容塞进320px界面中。如果我们通过控制width属性，将layout viewport的大小设置成屏幕的大小，浏览器就会以给定的宽度进行渲染。这时`width-device`属性正好派上用场了，将width设置成width-device就不会出现滚动条，也不会产生缩放，一切关于css宽度的理解又回到了pc端时的简单美好。

**总结：width属性作用于layout viewport。**

##initial-scale
|属性|备注|
|:-|:-|
|initial-scale|	设置页面的初始缩放值，为一个数字，可以带小数|

与width属性相反，initial-scale是控制visual viewport大小的，缩放比例越大，呈现的内容就越少，visual viewport值也就越小。

##target-densitydpi
|属性|备注|
|:-|:-|
|target-densitydpi| 目标设备的密度等级，作用是决定css中的1px代表多少物理像素|


下面是 target-densitydpi 属性的 取值范围

`device-dpi` – 使用设备原本的 dpi 作为目标 dp。 不会发生默认缩放。
`high-dpi` – 使用hdpi 作为目标 dpi。 中等像素密度和低像素密度设备相应缩小。
`medium-dpi` – 使用mdpi作为目标 dpi。 高像素密度设备相应放大， 像素密度设备相应缩小。 这是默认的target density.
`low-dpi` - 使用mdpi作为目标 dpi。中等像素密度和高像素密度设备相应放大。
`<value>` – 指定一个具体的dpi 值作为target dpi. 这个值的范围必须在70–400之间。当值超过最大值400或小于最小值70的时候，所设置的自定义值将被忽略，系统将会使用默认值medium-dpi来显示。

**PS:**
1) 这是安卓的私有属性，iOS不支持
2) 标准webkit已经不再支持该属性

[https://lists.webkit.org/pipermail/webkit-dev/2012-June/020914.html](https://lists.webkit.org/pipermail/webkit-dev/2012-June/020914.html)

> I think folks agree that these are important use cases.  The
disagreement is about how to address them.  For example, another
approach is to use responsive images (e.g., srcset and image-set)
together with device units in CSS.  Now that WebKit has support for
subpixel layout, we can provide a high quality implementation of these
features.

**实测**

|系统/浏览器|表现|
|:--|:--|
|iOS|跟本不支持|
|安卓(>4.0)/原生|支持|
|安卓/UC(最新版)|支持|
|安卓/Chrome|已放弃，不支持|

由于[安卓碎片化](http://baike.baidu.com/link?url=rfUHFmP1Z_B4PfYIjG090m4zrob_ZCF2_J4q0-TVRvXO7X41p8jexRVA91QLKiALYS7Un_Ua4Be00tMVYvVpAq)，以上测试仅限于几台中高端机型

尽管iOS的不支持以及webkit的放弃，但是在某些中低端安卓机型下，使用`target-densitydpi`来解决`initial-scale`缩放不生效问题也是很有用的。

##device-width
上面我们分析了使用`width=device-width`可以很好的做wap的适应，那这个`device-width`到底是什么呢？
要获取这个值很简单：

    //html
    <meta name="viewport" content="width=device-width"/>

    //js
    alert(window.innerWidth);

实测：

|设备|分辨率|屏幕尺寸|device-width|
|:--|:------|:--------|:----------|
|iphone4|960x640|3.5|320|
|红米 2|1280x720|4.7|360|
|OPPO X907|800x480|4.3|320|
|Galaxy Mega 2|1280x720|6|360|

这是一个非常nice的网站，可以查询各机型的`device-width`值：
[http://viewportsizes.com/](http://viewportsizes.com/)

通过上面的数据可知：
* 设备的`device-width`与设备的分辨率、屏幕尺寸没有必然联系
* 不同设备的分辨率、屏幕尺寸相差很大，但是`device-width`值相差不会太多，这也是为什么wap在众多移动端设备上的展现不会相差很大的根本原因

##wap下js获取文档宽度
学了以上知识后就不难理解文章开头说的bug了：

    window.innerWidth   //获取不准确
    document.documentElement.scrollWidth    //获取准确

在wap中，`window.innerWidth`指向的是visual viewport的大小；`document.documentElement.clientWidth`才是指向页面渲染时的layout viewport大小。虽然我们通过设置viewport的属性`width=device-width,initial-scale=1.0`使visual viewport等于layout viewport，但基于[安卓碎片化](http://baike.baidu.com/link?url=rfUHFmP1Z_B4PfYIjG090m4zrob_ZCF2_J4q0-TVRvXO7X41p8jexRVA91QLKiALYS7Un_Ua4Be00tMVYvVpAq)的前提，使两个视图相等的生效时间可能会有差异并不可通

**zepto有做兼容处理吗,`$(window).width()`会取得准确的值吗？**
上周，考虑项目进度，在没有阅读源码及可靠验证的情况下，只是简单的尝试了几次并取得正常值就断定zepto对获取文档有做兼容处理。其它这样的推断是没有依据，并且是错误的，看下面的分析

    //zepto 源码803行
    $.fn[dimension] = function(value){
      var offset, el = this[0]
      if (value === undefined) return isWindow(el) ? el['inner' + dimensionProperty] :
        isDocument(el) ? el.documentElement['scroll' + dimensionProperty] :
        (offset = this.offset()) && offset[dimension]
      else return this.each(function(idx){
        el = $(this)
        el.css(dimension, funcArg(this, value, idx, el[dimension]()))
      })
    }

上面是zepto关于尺寸方面的通用方法接口, 当调用`$(window).width()`时，上面函数的参数值分别为：

    dimension   -> width
    value   -> undefined

返回值是：`el['inner' + dimensionProperty]` 即`window[innerWidth]`

**原来`$(window).width()`就是`window.innerWidth`**，那这个值肯定也是不准确的，之所以在之前的验证中取得了正确的值，可能是因为函数的层层包装，使取值时的执行时间延迟了。

从上面的源码可以看出，如果想通过zepto获取准确的文档宽度，应该使用document：

    $(document).width();

##最佳实现
* 视图设置

        <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">

* 获取dom根节点宽度

        document.documentElement.scrollWidth    //原生
        $(document).width() //基于zepto
