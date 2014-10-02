---
layout: post
title: javascript控制表单输入
category: think
description: 在项目中我们经常遇到控制表单输入的需求，比如英文名只允许输入英文字符；价格输入框只允许输入数字和逗号；同步输入框的值显示在其它元素上等等，下面就来思考如何优雅得实现这类需求。
---

@(temp)[javascrip, onkeydown, onkeyup, onkeypress]

> 在项目中我们经常遇到控制表单输入的需求，比如英文名只允许输入英文字符；价格输入框只允许输入数字和逗号；同步输入框的值显示在其它元素上等等，下面就来思考如何优雅得实现这类需求。

表单输入交互相关的事件有：onkeydown, onkeyup, onkeypress, onfocus, onblur。但是要监听输入时的事件只有onkeydown, onkeyup, onkeypress，下面一一分析这三个事件

可以从下面两个纬度思考：

* 不管按了什么键，只关注输入的值
* 不管输入什么值，只关注按的键

表现在代码层，就是：

* 前者只关注输入框的值`this.value`
* 后者关注事件对象的`event.keyCode/event.which`

##onkeyup
onkeyup 事件会在键盘按键被松开时发生，引申含义：

* 输入完成后触发，即刚刚输入的值已经写入输入框
* 通过`this.value`得到的值包含刚刚输入的值。
* 即使在事件响应函数头部就立即退出：`return false`，也不能阻止之前的输入
* 松开时才响应，因此即使常按也只能响应该事件一次

所以如果在这个事件中通过正则去过滤`this.value`，被过滤的文字会先出现一会儿，然后再消失，有瑕疵。

**适合做值同步，不适合做值过滤**

##onkeypress
onkeypress 事件会在键盘按键被按下并释放一个键时发生,这个事件很特殊：

* 如果常按一个键，这个事件会一直响应
* 与onkeyup不同的是，这个事件触发时，输入的值还没有写入到输入框中，即`this.value`得到的值不包括刚刚输入的值，所以通过`return false`是可以取消输入的
* 只监听可输入型的按键，比如字母、数字和符号。对于功能键比如tab、shift、F1、backspace都不响应。不响应退格键，意味着按下退格键，输入框中的字符被删除一格，但是不响应onkeypress事件。监听不到事件，也就不能做类似值同步的处理

**适合做值过滤，值同步慢一拍，退格不支持同步**

##onkeydown
onkeydown 事件会在用户按下一个键盘按键时发生。
很好理解，按下立马触发并且响度常按。基本上跟onkeyperss相同，只是它全键盘都响应。

**适合做值过滤，值同步慢一拍，支持退格同步**

##过滤且同步的实现
吐槽：我xx，难道没有一个事件可以完美得支持过滤且同步值！
是的，至少从上面的分析来看，简单单独使用一个事件没法优雅得同时支持过滤及同步。
等等，难道真的一点办法也没有了，再来看这三家伙，onkeyup事后发生，在做值过滤时会出现一闪，这一点不能忍受；onkeypress没法响应退格键，绝对pass！这样只剩下onkeydown了，而onkeydown本身很适合做过滤的，那我们就只需要分析在onkeydown的基础上做值同步

再来理一理从按下键到释放再到输入框显示新增输入值的细节过程：

    按键->onkeydown事件->onkeypress事件->释放->写入值->onkeyup事件

并且这一系列过程是在一个事件时钟内完成的，你懂的，setTimeout(0)出场的机会到了！可以在onkeydown事件中利用setTimeout(0)来使定时器里的函数在写入值之后才触发**（setTimeout(0)不是立即执行，而是在这个时刻的所有运行时代码都执行完后立即执行）**，所以过程将变成如下：

    按键->onkeydown事件(setTimeout(fn, 0))->onkeypress事件->释放->写入值->onkeyup事件->fn调用

回想起来，setTimeout(0)在很多难题中都起到了至关重要的作用，比如动画计算、大数据运算中等。

实现代码如下：

    ipt.addEventListener('keydown', function(e){

        var root = this;    //setimeout中作用域不指向this，所以先缓存this对象引用

        //过滤不能输入的字符
        if (e.keyCode==xxx){
            return false;
        }

        //等待输入完成
        setTimeout(function () {

            //同步值
            $('#target').text(root.val());

        }, 0);

    });


###输入法干扰
在使用非英文输入法时，上面的过程会有点不同，输入法只是在输入完成之后由输入法将字母替换成相应的字符，所以大致流程如下：

    按键->onkeydown事件(setTimeout(fn, 0))->onkeypress事件->释放->写入值->onkeyup事件->输入法替换---fn调用

输入法替换不是当前时刻的运行时代码，所以输入法替换与fn调用谁先触发将不可控，为了准确，我们可以适当加大延时

代码如下：

    ipt.addEventListener('keydown', function(e){

        var root = this;    //setimeout中作用域不指定this，所以先缓存this引用

        //过滤不能输入的字符(没法控制中文)
        if (e.keyCode==110){
            return false;
        }

        //等待输入完成
        setTimeout(function () {

            //控制不能输入中文
            var val = root.value;
            root.value= val.replace(/[^a-zA-Z]/g,'');

            //同步值
            $('#target').text(root.val());

        }, 50); // 加大延时

    });


##总结

说了这么多，我自己都晕了，恐日后忘记，列表如下：

|事件|是否全键盘响应|响应顺序|是否响应常按|return是否生效|适合场景|
|:--|:--|:--|:--|:--|:--|:--|
|onkeydown|是|1|是|是|过滤且同步值|
|onkeypress|否|2|是|是|过滤|
|onkeyup|是|3|否|否|同步值|

最后补足一点

onkeypress事件还有一个不好的是：它的事件对象的e.which值不准确，不管输入的是大写还是小写字母，都按大写字母的ascii码返回，onkeydown和onkeyup没有这个bug。

###相关文章
[制作个性化工牌-一个js全栈小项目](http://webfing.github.com/design-characteristic-of-workcard)
