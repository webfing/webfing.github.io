---
layout: post
title: 未知文字图片高宽的水平垂直居中研究
description: 图片垂直水平居中的使用境场较多，这里先研究图片的实现方案，后面再延申文字的解决方案
category: study
---


@(blog)[垂直居中, 水平居中]

图片垂直水平居中的使用境场较多，这里先研究图片的实现方案，后面再延申文字的解决方案

## 图片

### 法一(背景图片居中)
透明gif图片+背景定位的方法，利用了background-position:center实现图片居中显示。这是个很实用也是很聪明的办法，对于维护控制成本都很不错。微软必应图片搜索的图片排列就是使用的这种方法。
方法的原理很简单，使用一个透明的gif图片做覆盖层，高宽拉伸至所需要的大小，然后给这个gif图片一个background- position:center center的属性。而background-image建议写在页面上，因为实际项目中，这肯定是个动态的URL地址，css文件似乎不支持动态URL 地址。

HTML结构部分：

    <div id="box">
	    <img src="pixel.png" alt="" style="background-image:url(../common/img.jpg);" />
    </div>

CSS样式部分：

    #box{
		width:300px;height:300px;
		text-align:center;
		vertical-align:middle;
		border:1px solid #d3d3d3;background:#fff;
	}

	#box img {
		display: block;
		background-repeat: no-repeat;
		background-position: center center;
		width:100;
		height:100%;
	}

效果图：

![Alt text](http://king-images.qiniudn.com/vertical-1.png)


### 法二(图片包裹一层)

该方法是将外部容器的显示模式设置成display:table，img标签外部再嵌套一个span标签，并设置span的显示模式为display:table-cell，这样就可以很方便的使用vertical-align象表格元素那样对齐了，当然这只是在标准浏览器下，IE6/IE7还得使用定位。

HTML结构部分：

	<div id="box">
		<span><img src="images/demo.jpg" alt="" /></span>
	</div>

CSS样式部分：

	<style type="text/css">
		#box{
			width:500px;height:400px;
			display:table;
			text-align:center;
			border:1px solid #d3d3d3;background:#fff;
		}
		#box span{
			display:table-cell;
			vertical-align:middle;
		}
		#box img{
			border:1px solid #ccc;
		}
	</style>

	<!--[if lte IE 7]>
	<style type="text/css">
		#box{
			position:relative;
			overflow:hidden;
		}
		#box span{
			position:absolute;
			left:50%;top:50%;
		}
		#box img{
			position:relative;
			left:-50%;top:-50%;
		}
	</style>
	<![endif]-->

注意在IE6-7的处理中：

* 三层定位分别是：相对-》绝对-》相对
* 内三层位置正负不能换，即不能是

		#box span{
			position:absolute;
			left:-50%;top:-50%;
		}
		#box img{
			position:relative;
			left:50%;top:50%;
		}

	因为本来left,top就是相对左上角的，所以相拆半就得先往右下角走，不然就不是正中了！

* 把最内层的图片改成如下在ie7可行，ie6不行

		#box img{
			margin-left:-50%;margin-top:-50%;
		}

### 法三：图片前加一兄弟元素

标准浏览器还是将外部容器#box的显示模式设置为display:table-cell，IE6/IE7是利用在img标签的前面插入一对空标签的办法。

HTML结构部分：

	<div id="box">
		<i></i><img src="images/demo.jpg" alt="" />
	</div>

CSS样式部分：

	<style type="text/css">
	#box{
		width:500px;height:400px;
		display:table-cell;
		text-align:center;
		vertical-align:middle;
		border:1px solid #d3d3d3;background:#fff;
	}
	#box img{
		border:1px solid #ccc;
	}
	</style>
	<!--[if IE]>
	<style type="text/css">
		#box i {
			display:inline-block;
			height:100%;
			width:5px; //为了展现效果，给个5px，运用中改为0即可
			vertical-align:middle;
			background: #eee; //为了展现效果，加个颜色
			}
		#box img {
			vertical-align:middle
		}
	</style>
	<![endif]-->

效果图：

![Alt text](http://king-images.qiniudn.com/vertical-2.png)

注意：

* 这里是使用了一个兄弟元素来撑大了行内元素的排版区域，一个关键点就是`height:100%;`

* 若不加这个兄弟元素，单给img加个`vertical-align:middle`是没有效果的，因为这个vertical-align是跟据文档流来定位的，而默认的文档流高度是根据内容定的，所以为了很这个文档流高度跟外框一致，就引入了这个撑高度的前置兄弟元素

以上三种方法，都一定程度的改变了HTML的构造，在有的应用环境下，前端sir可能没有这个权限，下面看看第四种方法

### 方法四(字体大小的妙用)

该方法针对IE6/IE7，将图片外部容器的字体大小按照所选字体类型比较设备大小，标准浏览器还是使用上面的方法来实现兼容，并且结构也是比较优雅。

HTML结构部分：

    <div id="box">
        <img src="images/demo.jpg" alt="" />
    </div>

CSS样式部分：

    #box{
		width:500px;height:400px;
		text-align:center;
		border:1px solid #d3d3d3;background:#fff;

		/* 兼容标准浏览器 */
		display: table-cell;
	    vertical-align:middle;

		/* 兼容IE6/IE7 */
		*display:block;
		/* TD (见下)*/
	}

	#box img{
		vertical-align:middle;
	}

这里把字段大小设置这块单独独立出来分析，因为这块跟字体类型选择有关

CSS样式TD部门：

方案一：

    *font-family:Arial;
    *font-size:349px;
    /* 选英文arial字体时字体大小约为容器高度的0.873倍 400*0.873 = 349 */


方案二：

    *font-family: "宋体";
    *font-size:400px;
    /* 选择宋体时，字体大小设置与容器高度相等即可 */

**与“法三：图片前加一兄弟元素”相比这里没有加内部元素同样也达到了撑大容器的作用，说明调节空器字体大小也是可以起到撑大文档流高度的作用**

注意：上面跟据字体选择的不同，采用的比例也不同，说明同样的字体大小，选择的字体类型不同，渲染后视角上占用的大小是不同的，至于这个比例的详细研究，得另起一篇文章来说明了

## 文字
可以推想，文字的垂直居中肯定用不了了上面图片垂直居中的第四种方案，所以文字的垂直居中解决方案中必须给文字再包装一层
实现的关键是把文字当图片处理，用一个span标签把文字包装起来，不就像图片了，上代码：

HTML结构部分：

    <div id="box">
        <span>
        是个成本很低，效果惊人的方法。适用于多图显示的情况。基本上用裸标签就实现了想要达到的效果。一般而言，图片阵列排列显示的时候，外面都有一个a标签的，起到链接的作用。而本处的方法就只要这一个a标签就足以实现图片垂直且居中的显示效果。其关键是将a标签默认的inline属性设置为inline-block
        </span>
    </div>

CSS样式部分：

    #box{
        width:400px;height:400px;
        text-align:center;
        border:1px solid #d3d3d3;background:#fff;
        overflow: hidden;
        font-size: 12px;

        /* 兼容标准浏览器 */
        display: table-cell;
    	vertical-align:middle;

        /* 兼容IE6/IE7 */
        *display:block;
        *font-size:400px;
        *font-family: '宋体';
    }
    #box span {
    	vertical-align: middle;
    	font-size: 12px;
    	*display: inline;
    	zoom:1;
    }

可以看出，包层容器的css代码基本都是一样的，只是给这个包装层span加下样式，使其表现出类型img元素的特性

## css3弹性盒模型
移动端最佳解决方案
看了上面这么多别扭的解决方案，是不是觉得很恶心。来看看css3解决方案吧，因为浏览器支持的原因，目前该方案仅适用于移动端产品:

HTMl代码：

    <div id="box">
    	<img src="../common/img/img.png" alt="" />
    </div>

CSS代码：

    #box{
    	width:400px;height:400px;
    	border:1px solid #d3d3d3;background:#fff;
    	/* flexbox, por favor */
        display: -webkit-box;
        -webkit-box-orient: horizontal;
        -webkit-box-pack: center;
        -webkit-box-align: center;

        display: -moz-box;
        -moz-box-orient: horizontal;
        -moz-box-pack: center;
        -moz-box-align: center;

        display: box;
        box-orient: horizontal;
        box-pack: center;
        box-align: center;
    }

弹性盒模型布局兼容性：

![Alt text](http://king-images.qiniudn.com/vertical-3.png)

