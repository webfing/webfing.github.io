---
layout: post
title: javascript内存泄露研究
category: think
description: 过去web1.0时期的网站大多通过跳转页面来实现交互，页面跳转后内存全部释放，即使跳转前出现少量内存泄露也不易被大家发现，研究的也比较少。ajax大行其道之后，页面逻辑越来越复杂，交互时间也越来越长，甚至路由及页面切换也有js前端来完成，同一个页面存在大量业务，这时内存泄露问题就很突显了
---

@(blog)[memory leak,memory management]

##背景
过去web1.0时期的网站大多通过跳转页面来实现交互，页面跳转后内存全部释放，即使跳转前出现少量内存泄露也不易被大家发现，研究的也比较少。ajax大行其道之后，页面逻辑越来越复杂，交互时间也越来越长，甚至路由及页面切换也有js前端来完成，同一个页面存在大量业务，这时内存泄露问题就很突显了

##内存管理

JS运行的时候，会有栈内存（stack）和堆内存（heap），当我们用new实例化一个类的时候，这个new出来的对象就保存在heap里面，而这个对象的引用则存储在stack里。程序通过stack里的引用找到这个对象。例如var a = [1,2,3];，a是存储在stack里的引用，heap里存储着内容为[1,2,3]的Array对象。当栈中的变量被重新符值，原来在堆中存储的对象就会被释放，这个过程就叫垃圾回收。

垃圾回收常见的有两种方式:

###标记清除（mark and sweep）
网上说的很复杂，下面是我的理解：堆中没有被栈中可访问(全局)变量直接或间接引用的对象，就会被回收

直接引用：

    var a = {name: 'king'};

间接引用：

    var a = {name: 'king'};
    var b = {age: 24};
    a.meta = b;
    b = null;

这里虽然栈中的b变量被重新符值没有再指向`{age:24}`对象，但是它被堆中的`{name:'king'}`对象的meta属性所引用；而这个`{name:'king'}`对象又被栈中的a变量引用，所以`{age:24}`对象间接被引用着，所以不会释放。

再看一个很容易误解的栗子：

    function fn(){
        var elem = document.getElementById('myElem');
        elem.onclick = function(){
            //todo
        }
    }
    fn();

按照上面的逻辑，fn函数运行完之后，引用`myElem`元素的局部变量elem被释放，没有可访问的变量引用这个dom对象了，岂不是要被回收，那为什么点击该元素，还是会响应onclick事件呢？

细节：页面渲染完之后，dom对象都被加载到内存中了，并且通过dom树一级级引用，`document.body`document是全局变量，通过`document.body.children`就可以引用所有的dom对象，所以上面的栗子，从全局来看，调用过程是这样的:

![Alt text](http://king-images.qiniudn.com/js-memory-dom.png)

执行fn函数时，给引用的`myElem`dom对象增加了一个onclick属性指向一个匿名函数，fn结束后，内部局部变量elem被释放，好像断开了栈中与myElem对象的联系，其实不然，window全局对象还有一跟线联系着myElem对象，所以myElem对象不会被释放，onclick事件也会一直有效

当执行以下代码后呢？

    function clearDom(){
        var elem = document.getElementById('myElem');
        document.body.removeChild(elem);
    }

![Alt text](http://king-images.qiniudn.com/js-memory-dom-rm.png)

当调用document.body.removeChild(elem) 后，切断了document.body与elem的联系，这时，堆里的myElem对象与事件闭包都没有被栈里的可访的变量直接或间接引用，所以会被回收(注意，这里讲的是标记清除模式，如果是引用计数在这种情况下会出问题，见下面分析)


###引用计数(reference counting)
引用计数的策略是跟踪记录每个值被使用的次数，当声明了一个变量并将一个引用类型赋值给该变量的时候这个值的引用次数就加1，如果该变量的值变成了另外一个，则这个值得引用次数减1，当这个值的引用次数变为0的时候，说明没有变量在使用，这个值没法被访问了，因此可以将其占用的空间回收，这样垃圾回收器会在运行的时候清理掉引用次数为0的值占用的空间。

看起来也不错的方式，为什么很少有浏览器采用，还会带来内存泄露问题呢？主要是因为这种方式没办法解决循环引用问题。比如对象A有一个属性指向对象B，而对象B也有有一个属性指向对象A，这样相互引用

    function test(){
        var a={};
        var b={};
        a.prop=b;
        b.prop=a;
    }

在标记清除的策略下是没有问题的，离开环境并且没有直接或间接的就被清除，但是在引用计数策略下不行，因为这两个对象的引用次数仍然是1，不会变成0，所以其占用空间不会被清理，如果这个函数被多次调用，这样就会不断地有空间不会被回收，最后造成内存泄露。

再拿上面切断dom联系的栗子来说明一下：

![Alt text](http://king-images.qiniudn.com/js-memory-dom-can-not-rm.png)
在执行`document.body.removeChild(elem);`后堆中的myElem和闭包虽然与外界断开了联系，但是因为彼此引用，引用计数大于0，所以内存无法被释放。

###浏览器使用
大部分浏览器都是使用标记清除进行垃圾回收，只有低版本IE，低版本IE同时使用了两套垃圾回收机制，不出所料，又是IE！

在低版本IE中虽然JavaScript对象通过标记清除的方式进行垃圾回收，但BOM与DOM对象却是通过引用计数回收垃圾的，也就是说只要涉及BOM及DOM就会出现循环引用问题。

##例子实证

###js对象循环引用
html文件

    <button id="start">start</button>
    <button id="end">end</button>

js文件

    function test(){
        var a={};
        var b = {};
        a.largeStr = new Array(1000000).join('xxoo');
        b.largeStr = new Array(1000000).join('xxoo');
        a.prop=b;
        b.prop=a;
    }

    window.onload = function(){
       document.getElementById('start').onclick = function() {
        myEvent = setInterval(function(){
          test();

        }, 50)
      }
      document.getElementById('end').onclick = function(){
        clearInterval(myEvent);
      }
    }


* chrome
 ![Alt text](http://king-images.qiniudn.com/js-memory-chrome.png)

* IE9+
 ![Alt text](http://king-images.qiniudn.com/js-memory-ie.png)

* IE6-8
 ![Alt text](http://king-images.qiniudn.com/js-memory-ie6.png)

实例证明，对于标准js对象，无论是否循环引用，只要变量退出了环境，不能达到，引用的内存区域就会被释放

###js对象、dom对象非循环引用

    function test(){
        var a={};
        var dom = document.createElement('div');
        a.largeStr = new Array(1000000).join('xxoo');
        dom.largeStr = new Array(1000000).join('xxoo');
    }

表现跟之前一样：函数中引用的对象被释放，没有异常

###js对象与未加入body中的dom对象循环引用

    function test(){
        var a={};
        var dom = document.createElement('div');
        a.largeStr = new Array(1000000).join('xxoo');
        dom.largeStr = new Array(1000000).join('xxoo');
        a.prop=dom;
        dom.prop=a;
    }

虽然有循环引用，并且之一是DOM对象，但是所有浏览器也都表现跟之前一样，没有泄露。所以之前的总结不完善，DOM循环引用倒致内存泄露的前提条件是这个DOM对象是已进加入到页面中的。

###js对象与body中的dom对象循环引用

    function test(){
        var a={};
        var dom = document.createElement('div');
        a.largeStr = new Array(1000000).join('xxoo');
        dom.largeStr = new Array(1000000).join('xxoo');
        document.body.appendChild(dom);
        a.prop=dom;
        dom.prop=a;
        document.body.removeChild(dom);
    }

这时除IE6-8表现不正常，出现内存泄露外，其它正常。下图是IE8实测，内存一直在增长

![Alt text](http://king-images.qiniudn.com/js-memory-ie6-2.png)

解决办法：**切断循环引用**

    function test(){
        var a={};
        var dom = document.createElement('div');
        a.largeStr = new Array(1000000).join('xxoo');
        dom.largeStr = new Array(1000000).join('xxoo');
        document.body.appendChild(dom);
        a.prop=dom;
        dom.prop=a;
        document.body.removeChild(dom);
        a.prop=null;
        dom.prop=null;
    }

###XMLHttpRequest对象泄漏

####问题再现
在IE低版本中，除了上面说的BOM和DOM对象的循环引用会造成内存泄露外，还有一个js对象也会产生泄漏，就是XMLHttpRequest对象，XMLHttpRequest在IE<9会造成内存泄露，看下面的代码：

    window.onload = function() {
      setInterval(function(){
         var xhr = new XMLHttpRequest();
         xhr.largeStr = new Array(100000).join('xxoo');
         xhr.open('GET', '/users', true);
         xhr.onreadystatechange = function() {
            if(this.readyState == 4 && this.status == 200) {
               document.getElementById('test').innerHTML++
            }
         };
         xhr.send();
      }, 50)
    }

按照之前的逻辑，即使在IE中，对于非DOM及BOM对象，垃圾回收采用的是标记清除的方式，那么在每个定时器函数运行完之后，xhr及事件闭包都没有直接的引用了，按道理说，这两个对象应该会被释放，但实际上在IE7-8（ie6除外，ie6没有XMLHttpRequest对象）没有释放，看下面的实证（为了使数据更明显，这里把一个大字符串绑定在xhr的一个属性上），看下面在各浏览器的表现。

* 在chrome下检查内存变化
 ![Alt text](http://king-images.qiniudn.com/js-memory-chrome-2.png)

 可以看到，xhr对象被定期回收了

* IE10中(为了在IE中更详细的检查内存变化，选择IE10)
 ![Alt text](http://king-images.qiniudn.com/js-memory-ie-2.png)


* IE8(请使用ietester或虚拟机里的原生IE8)

 ![Alt text](http://king-images.qiniudn.com/js-memory-ie6-3.png)

 ![Alt text](http://king-images.qiniudn.com/js-memory-ie6-4.png)
可以看到运行中时内存一直在增加，得不到释放。关闭浏览器之后，内存降下来了。

####原因分析
关于这个在低版本中XMLHttpRequest对象内存泄露问题，网上资源很少，英文资料也特别少，搜到一篇分析不错的文章：
http://nullprogram.com/blog/2013/02/08/

结合作者的理解：

按照正常情况，在结束定时器闭包时，xhr及绑定事件的闭包应该被标计清除法清除掉，但是想像一下，如果真清除掉了，这个XHR实例也就没有意思了，因为后面即使请求成功了，也没有XHR来响应了。猜测浏览器内部应该是把这个xhr及闭包函数放在了一个不能被外部直接访问空间里来响应回调。并且通过一套机制来等响应成功后清除掉这块内存空间，但是早期的IE在这个设计上存在不足，没法回收。总结一句就是：XMLHttpRequest特殊就在于它是异部执行的，所以不能立马从内存中释放。

####解决办法
上面再现了问题及分析了问题，下面想想如何解决XMLHttpRequest对象内存泄露问题

     window.onload = function() {
      setInterval(function(){
         var xhr = new XMLHttpRequest();
         xhr.largeStr = new Array(100000).join('xxoo');
         xhr.open('GET', '/users', true);
         xhr.onreadystatechange = function() {
            if(this.readyState == 4 && this.status == 200) {  //注意点！
               document.getElementById('test').innerHTML++
            }
         };
         xhr.send();
         xhr = null; //重点！
      }, 50)
    }

不确定内部具体原因，但是直觉感觉就是栈中还有一块引用了这个xhr对象，所以把它的引用切断就可以了。
不过要注意的一点是：xhr对象的状态事件闭包中对xhr自身的引用改用`this`，不要使用xhr了，因为闭包内的环境是该闭包结束时的状态，这时的xhr被重署为了null。注意，是xhr变量被重新符值了，栈中xhr对象还是在的，所以状态回调还是可以处理的，this还是有意义的！

##易出现泄露的场景

* XMLHttpRequest
 泄漏发生在IE7-8

* DOM&BOM等COM对象循环绑定
 泄漏发生在IE6-8

* 定时器(严格上说不能算是泄露，是被闭包持有了，是正常的表现)

###优化闭包
我们知道闭包会把外部的环境变量通通包有，这样会加大内存占有量，可以把作用域中没用的属性删除，以减少内存消耗。

##内存监控
### chrome

#### 宏观分析-Timeline

![Alt text](http://king-images.qiniudn.com/js-memory-chrome-2.png)


#### 微观分析-Profiles

分析步骤：

![Alt text](http://king-images.qiniudn.com/js-memory-watch-chrome-1.png)

一、不进行任何交互，存一次快照
二、对要检查的功能区就行交互，然后再存一次快照
三、选择compare模比，对比两快照

![Alt text](http://king-images.qiniudn.com/js-memory-watch-chrome-2.png)

上图中，可以很清楚的知道哪些对象发生了改变，并且从列中也可以看出是新增了还是删除了，并且占用了多少节点也可以看到；如果对比结果中数据有很多条，不直关，也可以在"class filter"中输入想要查看的对象，来过滤：

![Alt text](http://king-images.qiniudn.com/js-memory-watch-chrome-3.png)

### IE

* ie dev tools
    只针对高版本IE

* sIEve
    对DOM树的分析较详细，对内存的分析还不如任务管理器直观

##总结
为了预防内存泄露及持有不必要的内存，做到如下几点：

* 在使用XMLHttpRequest，定时器等异步对象时，在函数结尾对不需要引用的对象进行释放

* 在DOM循环引用中，记得在函数结束时对循环引用切断

