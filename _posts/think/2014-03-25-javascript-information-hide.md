---
layout: post
title: javascript信息隐藏技术思考
category: think
description: 大多数码农应该都会有这样的成长经历：刚开始技术还不熟练时，就是一堆的copy，然后修修改改就成了自己的代码；过段时间，对代码熟练后，开始动手自己写代码，并试着用各种模式方法使代码看起来牛逼；再接着，和团队一起做项目时，就会意识到代码规范易用的重要性，这时就会思考如何设计优雅的api。
---


> 大多数码农应该都会有这样的成长经历：刚开始技术还不熟练时，就是一堆的copy，然后修修改改就成了自己的代码；过段时间，对代码熟练后，开始动手自己写代码，并试着用各种模式方法使代码看起来牛逼；再接着，和团队一起做项目时，就会意识到代码规范易用的重要性，这时就会思考如何设计优雅的api。

## 常见的构造函数

整篇文章将以写一个弹窗组件为思路进行，写组件，我们一般的思路是这样的：

    function Dialog (opts){
        this.opts = $.extend({
            ......
        }, opts);
        this.wrapElem = ...;
        this.init(opts);
    }

    Dialog.prototype = {
        constructor: Dialog,
        init: function(){},
        bindEvent: function(){},
        createShadow: function(){},
        show: function(){},
        hide: function(){},
        destroy: function(){}
    }

    var myDailog = new Dialog();
    myDialog.show();
    myDialog.hide();


如果这个类是自己构建、自已使用，倒也是无可非议，但是在团队协作中，让同事使用这个类，同事肯定不清楚到底哪些方法和属性在哪些情况下是可以使用的，不能跟上作者的思路，作者也不能禅述清楚哪些是外部可以调用的，哪些是内部使用到的，这样肯定会加大团队的沟通成本。

发现有的同事使用下划线来做区分，指出凡是加了下划线前缀的属性和方法都是内部使用，代码如下：

    function Dialog (opts){
        this._opts = $.extend({
            ......
        }, opts);
        this._wrapElem = ...;
        this.init(opts);
    }

    Dialog.prototype = {
        constructor: Dialog,
        _init: function(){},
        _bindEvent: function(){},
        _createShadow: function(){},
        show: function(){},
        hide: function(){},
        destroy: function(){}
    }


通过读源码倒也是能够分辨出哪些是内部成员哪些是可以对外使用的成员，但是这样也难以避免与新来同事的沟通成本，并且通过命名来处理代码本身的性质也是不妥的。

## 思考

上面提到仅仅通过命名规范来达到信息隐藏是欠妥的，那么我们就来思考，看有什么办法可以做到真正的信息隐藏。

使用构造函数最基本的结构是这样的

    function Dialog (){     }

    Dialog.prototype = {
        constructor: Dialog
    }


### 尝试1

因为js没有类，只有原型链，而原型属性的声明一般又在构造函数的同级作用域，何不把原型的申明写在构造函数里，这样原型里的方法就可以直接访问构造函数作用域里的变量：

    function Dialog(opts){
        opts = opts || {};
        var width = opts.width || 100;
        var height = opts.height || 200;
        if (!this.getWidth){
            Dialog.prototype = {
                constructor: Dialog,
                getWidth: function(){
                    return width;
                }
            }
        }
    }


注意：在原型属性赋值前先判断下该对象的原型属性是否已经初始化，确保只在第一次的实例化中给原型属性赋值，之后的实例化就不会再重新赋值了。

实例化两个对象：

    var first = new Dialog();
    var second = new Dialog();


发现first实例没有getWidth方法、而second实例有这个方法

回想起《javascript高级程序设计》一书中的相关知识：

> 构造函数初始化的四个步骤：（p145） 1. 创建一个新对象 2. 将构造函数的作用域赋给新对象（因些this就指向了这个新对象） 3. 执行构造函数中的代码（为这个新对象添加属性） 4. 返回新对象
>
> 理解原型对象：(p148) 当调用构造函数创建一个新实例后，该实例的内部将包含一个指针（内部属性），指向构造函数的原型对象

因些上面实例化first对象时，在执行原型属性赋值前，first实例就有了一个内部属性[[prototype]]指向构造函数的原型对象，即这个first对象的一个内部属性指向了构造函数的原型对象。当执行到原型属性赋值时，构造函数的prototype属性被重新赋值了，切断了构造函数与原本原型对象的连接，这样之后实例化的对象也与这个最初的原型对象没有了关系。因些first对象的原型与second对象的原型不是一个对象。

### 尝试2

既然给构造函数的原型属性重新赋值会切断与原来真正的原型对象的联系，那就换下面这种做法试试

    function Dialog(opts){
        opts = opts || {};
        var width = opts.width || 100;
        var height = opts.height || 200;
        if (!this.getWidth){
            Dialog.prototype.getWidth = function(){
                return width;
            }
            Dialog.prototype.setWidth = function(val){
                width = val;
            }
        }
    }


现在各实例的原型都指向同一个对象，并且原型中的方法访问内部私有属性也不用绕，直接引用。 但是，看下面的调用：

    var first = new Dialog();
    var second = new Dialog();
    first.setWidth(1);
    second.getWidth();


结果发现，first对象调用设置值后，second对象取值方法输出的值也变了。 因为原型方法只定义了一次，并且当时的上下文是first对象实例化时的环境。 通过上面的尝试可以得出结论，把原型的申明放在构造函数里是不妥的。

其实想想上面的想法也是矛盾的，我们的目的是想把一些不该对外暴露的属性用var申明为局部变量，因些原型不能访问到这些局部变量，于是我们试着把方法全部写在构造函数里作为特权方法来使用，但这样又达不到公用的目的；然后我们又尝试把原型也写到构造函数里，发现原型的上下文只是第一次实例时的对象，也失败了。

### 尝试3

既然原型放在构造函数里是行不通的，那何不逆向思维，把局部变量的申明写在外面，并在构造函数里赋值，代码如下：

    //内部成员
    var _ = {}

    function Dialog(opts){
        opts = $.extend({}, Dialog.config, opts);
        _.width = opts.width;
    }

    Dialog.prototype = {
        constructor: Dialog,
        getWidth: function(){
            return _.width
        }
    }
    Dialog.config = {
        width:100
    }


试了试还是不行的，因为每次实例化都会把内部成员“\_"的属性重写，这样原型方法被调用时访问的是最后一次实例时赋值的“\_”对象

虽然一次次尝试的结果都是失败，但至少思路清淅了：一个实例的属性既不想对外暴露又希望原型方法能够直接访问到是行不通的，鱼和熊掌不可兼得

### 取经

尝试了多种方案都无果，那就向大牛取经吧。看了看夜雨带刀的[easyjs][1]，下面是其dialog组件的部分代码：

    //私有方法
    var easyDialog = {

        appendIframe : function( elem ){},

        createOverlay : function( zIndex ){},

        createDialogBox : function( zIndex, overlay ){},

        createDialogContent : function( o ){},
    }

    //构造函数
    var Dialog = function( target, options ){
        o.target = target;
        this.__o__ = o;
    };

    //公共方法
    Dialog.prototype = {

        destroy : function(){

            var o = this.__o__;

            if( o.elem ){
                o.elem.hide();
                $body.append( o.elem );
            }
       }
    }


疑问豁然开朗，其中非常重要的三点经验是：

1.  不想对外暴露的方法和函数段放在与构造函数同级的作用域中，这样构造函数和原型都可以直接调用，并且这些方法和函数只做抽象，具体的属性和值使用参数传入，这样就可以多个实例公用，互不影响
2.  使用特殊标识法this.\_\_o\_\_来保存参数options的值，这样原型方法不通过传值就能使用参数；使用两个下线来做标识，虽然也会对外暴露，但对使用者而言也很清淅明了
3.  不想对外暴露的属性可以挂载到option上，如上面的target

## 总结

跟据之前的思考和牛人的经验，下面整理出自己在组件开发中使用到的信息隐藏技术，还是以dialog组件为例：

    var Dialog = (function(){

        //公有内部静态成员
        var _ = {
            index: 10000,
            indexArr: [],
            fixIe6: function (){},
            initPostion: function(opts, index){},
            resize: function(wrapElem, pos){},
            createDialog: function(opts, index, that){}
            ......
        };

        //构造函数
        function Dialog (opts){
            opts = $.extend(true, {}, Dialog.config, opts);
            _.index = _.index+10;
            this.index = _.index;
            this._o_ = opts;
            var that = this,
                index = this.index;
            init();
        }

        //原型-对外公共方法
        Dialog.prototype = {
            constructor: Dialog,
            show: function (){},
            hide: function (){},
            ......
        };

        //配置项，做为类静态成员，用户可自定义初始化值
        Dialog.config = {
            width: 400,                     //弹窗内容区宽度
            height: 300,                    //弹窗内容区高度
            title: 'Title',                 //弹窗标题
            content: 'Content',             //弹窗内容
            onshow: function (){},          //打开事件
            onhide: function (){}           //关闭事件
            ......
        };

        return Dialog;

    })();

 [1]: http://easyjs.org/

[BeiYuu]:    http://beiyuu.com  "BeiYuu"
