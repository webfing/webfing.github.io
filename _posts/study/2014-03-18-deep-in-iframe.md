---
layout: post
title: iframe深入了解
description: iframe是个奇葩，在很多属性上跟其它dom表现不一致，还有众多浏览兼容性问题，很多书籍和文章都建议不要使用iframe，但在实践运用中，即看到很多站点或技术细节在使用iframe，iframe还是有其用武之地的。
category: study
---

@(blog)[iframe, cross origin, same origin]

> iframe是个奇葩，在很多属性上跟其它dom表现不一致，还有众多浏览兼容性问题，很多书籍和文章都建议不要使用iframe，但在实践运用中，即看到很多站点或技术细节在使用iframe，iframe还是有其用武之地的。

## iframe标签属性

* iframe去除边框

    使用`border:0`在IE6-8不起作用，真正跨浏览器的属性应该是iframe标签属性frameborder，兼容全部浏览器

        <iframe  src="inner.html" frameborder=0></iframe>

* iframe是行内元素，width不会自动全铺满，默认大小为：300*150
* iframe去除背景色
    使用`background: transparent;`不起作用，兼容全部浏览器的做法是：

        <iframe allowtransparency="true" src="inner.html"></iframe>

* 在HTTPS中不指定，或指定空的src的iframe会出错，解决方案：

        <iframe src="JavaScript:''"></iframe>

所以为了全兼容性，如果是空的iframe，要指定src为`javascript:''`
如果iframe的src除了可以引用url，如果内容是动态生成的，那么src可以用javascript来生成内容：

        <iframe src="JavaScript:'<h1>title</h1>'" style="height:60px"></iframe>

## 获取iframe内部文档（同源下）

### DOM方法
一个iframe元素也是一个普通的DOM元素，唯一的区别是它存在一个关联的window对象，通过contentWindow属性关联，所以：

    iframe.contentWindow.document //获取到iframe内部文档对象
    iframe.contentWindow.window //获取到iframe内部window对象

### BOM方法
除了上面的DOM方法，还可以使用window.framesBOM方法，索引可以是序号也可以是iframe的name属性：

1. window.frames[0] - 序号
2. window.frames['iframeName'] - name属性

用法同上面的`iframe.contentWindow`

不过这种获取方法在**IE6-7**中针对**动态生成**的iframe时会出错:

    function addIframe(){
    	var innerframe = document.createElement('iframe');
    	innerframe.src = 'inner.html';
    	innerframe.name = 'innerframe';
    	document.body.appendChild(innerframe);
    }
    addIframe();
    alert(window.frames['innerframe']) //undefined
动态符值的name，不会同步绑定到window.frames中，针对这个问题的解决方案：

#### 用id来获取iframe
既然name不会同步到`window.frames`对象中，那就用动态获取的`document.getElementById`来获取这个iframe不就解决问题了，代码如下:

    function addIframe(){
    	var innerframe = document.createElement('iframe');
    	innerframe.src = 'inner.html';
    	innerframe.id = "innerframe"; //用Id来取代name定位问题
    	document.body.appendChild(innerframe);
    }
    addIframe()
    innerframeObj = document.getElementById('innerframe').contentWindow;

#### IE特有createElement方法
IE6-8的createElement方法除了跟其它的浏览器一样可以传入标签元素外，还可以直接传入标签格式的内容，不过其它浏览器这样传参数会报错，所以兼容的解决方案是：

    function addIframe(){
    	var innerframe;
    	try{
    		innerframe = document.createElement('<iframe name="innerframe" src="inner.html">');   //IE6-8写法
    	}catch(e){
    		innerframe = document.createElement('iframe');
    		innerframe.name = 'innerframe';
    		innerframe.src = 'inner.html';
    	}
    	document.body.appendChild(innerframe);
    }
    addIframe();
    innerframeObj = window.frames['innerframe'];

## iframe事件
因为获取iframe对象有DOM和BOM两种方法，同样这个事件属性也有两种定义方法：

    //法一：DOM事件
    document.getElementById('innerframe').onload = function(){
    	alert('frame elememt loaded');
    }

    //法二：BOM事件
    window.frames['innerframe'].window.onload = function(){
    	alert('frame window loaded');
    }

其中法一在所有浏览器都可以生效，法二在IE6下绑定不成功。除了兼容性，还有一个重大区别是：法一事件只是绑在iframe元素上，不涉及同源策略的限制，而法二是要进入iframe内的window对象，这个肯定是受到了同源策略的限制，综上唯一标准稳定的方法是法一：**绑定在iframe元素上**

## html5特性
由于iframe的危险性，html5对iframe加入了一个新属性(sandbox)来控制iframe内的权限，相关资料：

[http://www.html5rocks.com/en/tutorials/security/sandboxed-iframes/?redirect_from_locale=zh](http://www.html5rocks.com/en/tutorials/security/sandboxed-iframes/?redirect_from_locale=zh)
[http://msdn.microsoft.com/en-us/hh563496.aspx](http://msdn.microsoft.com/en-us/hh563496.aspx)

## 跨域通信
上面研究过，在跨域的情况下是读不到iframe内的文档的，但是不是完全一点信息都读不到，location是可以读取并设置的:

    //父读子
    outerframe = window.frames['frame'];
	outerframe.window.location = 'http://www.baidu.com';

	//子读父
	if (window !=window.top){
		window.top.location = 'http://www.baidu.com';
	}

上面两者都可以生效，不过要注意一点，只能读到location节点，以下不能成功：

    outerframe.window.location.href = 'http://www.baidu.com';
    outerframe.window.location.hash = 'abc';

