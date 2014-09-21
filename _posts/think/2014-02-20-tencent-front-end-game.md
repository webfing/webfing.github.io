---
layout: post
title: 另类思维攻破腾讯《前端特工》
category: think
description: 在网上看到腾讯的挑战游戏，本着试试的态度，题目挺有意思，我的斗志上来了，硬搞三小时通关。想着快速通关，使用了很多小聪明小技巧走了捷径，改天再按正常逻辑思维走一遍。
---

@(blog)
游戏网址：<http://codestar.alloyteam.com/>

> 在网上看到腾讯的挑战游戏，本着试试的态度，题目挺有意思，我的斗志上来了，硬搞三小时通关。想着快速通关，使用了很多小聪明小技巧走了捷径，改天再按正常逻辑思维走一遍。

## 第一关：潜入

> “开个门而已，竟然还要抓包……”

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/d91171b0-9a10-4810-8083-45494123d3ad/1.1.jpg?resizeSmall&width=832&alpha=)


点击“闯关”，弹出一个表单，先不管代码啥的，按正常逻辑输入个人信息试试有啥反应，输入完弹出提示：

> 打不开？抓一下包看看……

程序提示我们用抓包工具，这是要看http请求啦？使用**小提琴(wins)** or **charles(mac)**，太麻烦了，咱先使用chrome的DevTools看看

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/7679b6ce-2700-4490-bca3-1212f11551b2/1.2.jpg?resizeSmall&width=832&alpha=)


请求返回的数据结果告诉我们：没有设置隐藏域的值，好吧，看来有隐藏的input，查看下表单的dom结构：

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/c378e46b-ee8d-46e4-b65e-afa14117985d/1.3.jpg?resizeSmall&width=832&alpha=)

果然有一个隐藏输入框，name属性是timestamp(时间戳)，这么说应该是要在这里输入当前时间，在DevTools的控制台输出看下当前时间戳：

> new Date().getTime();</br> 1392813114217

好的，再把这个值写入到隐藏的输入框中

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/1e7dfbf4-ea20-442e-b1f1-31345700868a/1.4.jpg?resizeSmall&width=832&alpha=)


然后再点打开提交表单，yes! 成功了，第一关通过

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/51df808b-6792-4e9f-a29c-367c1492d688/1.5.jpg?resizeSmall&width=832&alpha=)


## 第二关：意外

> “哥们是不是注定要拯救世界，这种事也被我碰上 = = 2:162 旋转原点，那是左上方吧?...又好像不是 2:162 好像在哪看过这个CSS3企鹅……”

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/90546357-d9d6-40bb-b10b-2a1bc20578fc/2.1.jpg?resizeSmall&width=832&alpha=)


第二关是使用css3画腾讯的logo，左边的圆很容易就想出来了，可是右边的那个衣角，我的妈呀，这可是一堆css3属性才能写出来的呀，还可能要带一堆前缀，时间一秒秒跑，我的个急啊，没办法，硬写是来不急了，先从DevTools看看源代码吧，看看有啥提示不

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/d7a76bf0-5e02-4975-82a4-edea435fbf0d/2.2.jpg?resizeSmall&width=832&alpha=)


真是柳暗花明又一村，出现注释了，果断打开注释中的网址，原来是腾讯AlloyTeam团队博客的一篇文章，讲解很详细，粗略看了下，文章中没有清淅得给出衣角的实现代码，想了想，不如直接看logo的css代码：

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/2b593866-5a43-4914-b595-c9c2cc8390ea/2.3.jpg?resizeSmall&width=832&alpha=)


    border-top: 20px solid transparent;
    border-bottom: 20px solid transparent;
    border-right: 8px solid black;
    -webkit-transform-origin: top right;
    -webkit-transform: rotate(-60deg);
    -moz-transform-origin: top right;
    -moz-transform: rotate(-60deg);
    -o-transform-origin: top right;
    -o-transform: rotate(-60deg);
    transform-origin: top right;
    transform: rotate(-60deg);


把这又长又难写难记的代码复制and粘贴，果断通关了

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/51df808b-6792-4e9f-a29c-367c1492d688/1.5.jpg?resizeSmall&width=832&alpha=)



## 第三关：交锋


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/a0877273-07f5-4593-b5a8-85b953f60ec1/3.1.jpg?resizeSmall&width=832&alpha=)


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/84d0fc45-df56-4c55-ab87-5c5481c05aca/3.2.jpg?resizeSmall&width=832&alpha=)


从界面来看，这一关是要打游戏啰，点击跳到坦克机器人大战游戏，之前也有了解过，知道是AlloyTeam团队开发的html5游戏，要注册太麻烦了，而且之前也没玩过，直觉告诉我这是要通过写代码来玩游戏，我着急啊，难不成英雄止步于此了，好吧，还是一直即往得从页面的源代码入写看有什么收获


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/08956fc0-c07d-48b8-8cb7-db4c9d05a182/3.3.jpg?resizeSmall&width=832&alpha=)


对比下前三关的源代码，发现在这块script标签中的代码都不一样，而且都是变量名混淆了，应该是程序的主要逻辑段，到webStorm里格式化代码看下有啥：


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/a765bf0c-7281-4b16-b678-53e9d5586dbd/3.4.jpg?resizeSmall&width=832&alpha=)


虽然代码混淆了，但是代码的大体意思还是能了解一二的，

最外层是一个自运行函数，第一段是个ajax请求，应该就是每关的通关验证请求，而且这个请求函数封装在f函数中，这样只要能够执行这个f函数就可以发请求了，但是这个在函数内的局部变量，外部是不能调用的，如何执行呢？

继续往下看，唯一的事件异步闭包中没有用到f函数，再往下看，亮瞎了，大名鼎鼎的window出现了:

    window["pass" + e] = function () {
        f();
        a && a.close()
    };


原来在这个函数内部，给全局变量window加了个方法，这个方法内就有执行f函数，这个方法名叫什么呢，再看e的申明：

     var e = +new Date


显然这个方法名应该就是类似window[pass13000****]这样的结构，具体是叫什么，就得看当时执行这个代码的时间了，没法知道，不过，chrome DevTools有自动提示的功能，不妨先输个window.pass试试，果然输到pass就出现了一个唯一提示，顺势加个括号执行呗

    window.pass1392814668594();


成功了，触发了ajax请求，第三关就这么愉快得通关了？

思考：难道是腾讯这个游戏的设计漏洞，应该也不可能，写得这么明显的window；那跳到坦克大战是为何，而且即使坦克大战赢了又怎么把信息传递回来呢？代码中也没有写，只有个window.open的弹出新窗口代码，没有回应代码，好吧，只能理解成这关是故意这样设计的，一是为了考考游戏者的hack思维；二是顺便给游戏做下推广。我个人是这样思解的。

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/51df808b-6792-4e9f-a29c-367c1492d688/1.5.jpg?resizeSmall&width=832&alpha=)


## 第四关：房门

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/8304e455-d7ae-446f-ac65-aa42c6ccc6e3/4.1.jpg?resizeSmall&width=832&alpha=)


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/94cfd776-299d-47f2-8327-1780e3aba11b/4.2.jpg?resizeSmall&width=832&alpha=)


这一关是考js基础知识了

1.  第一题：数组复制。很简单，使用数组的slice方法，不带参数，就相当于复制了

        arr.slice();


2.  第二题：字符串去首尾空格。这个使用正则很方便就完成了

        s.replace(/^\s+|\s+$/g, "");


3.  第三题：将Nodelist对象转换为数组对象。使用Array原生的slice方法来调用

        Array.prototype.slice.call(list);


ok，第四关打通

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/51df808b-6792-4e9f-a29c-367c1492d688/1.5.jpg?resizeSmall&width=832&alpha=)


## 第五关：资料

> “这丫的是谁建的文件夹？ 5:147 看来要按一定顺序打开，使得保密等级最高，才能找到我要的资料。”

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/1a371663-730b-4fe8-a28f-d39b98419b19/5.1.jpg?resizeSmall&width=832&alpha=)


咯，不会是考数学思维吧，这么多年了，排列组合都还给了老师；上大学时有们叫《运筹学》的课，好像就有很多研究这种最短路线，最长路线的问题，可惜我没学好，没办法，只能硬攻！

### 第一次进攻-强攻：

数学公式不太行，咱就走代码这条路，老地方：


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/16f3cfe1-f107-49d1-b5bd-cb799b1ff57c/5.2.jpg?resizeSmall&width=832&alpha=)


格式化代码，代码有70多行，比较多，这里列出关键的部分：

ajax请求函数t:

        function t() {
        ajax("/pass", {method: "POST", data: '{"q":1,"s":5,"_t":' + u(v) + "}", contentType: "application/json", onSuccess: function () {
            hideBoard();
            document.getElementById("btnNext").className += " show";
            alert("\u8fc7\u5173\uff01\u4e0b\u4e00\u5173\u7684\u5165\u53e3\u5df2\u6253\u5f00")
        }})
    }


延续打第三关时的思路，查找调用f的代码段，在q函数内有调用，这里先是一个判断，大致的意思是看是否12行都进行了选择，如果是就执行p()，并且当s全等于e时再执行我们想要的t()函数

    function q(b) {
        r.style.visibility = "hidden";
        for (var a = Number(b.parentNode.id.slice(4)), d = Number(b.id.slice(7)), c = a * (a - 1) / 2 + 1, x = (a + 1) * (a + 1 - 1) / 2 + 1; c < x; c++)document.getElementById("folder_" + c).classList.remove("open");
        b.classList.add("open");
        if (12 !== a) {
            for (c = (a + 1) * (a + 1 - 1) / 2 + 1; 78 >= c; c++)document.getElementById("folder_" + c).className = "folder";
            document.getElementById("folder_" + (a + d)).classList.add("able");
            document.getElementById("folder_" + (a + d + 1)).classList.add("able");
            p()
        } else p(), s === e ? t() : (r.style.visibility = "visible", f--, m())
    }


看来这关的关键就是让s全等于e，再调用q函数就能够发通关请求了，顺着这个思路查了查s和e的申明和运算代码，看来又智商着急了，一坨没有大括号的for循环，完全读不明白，死路一条！

    (function () {
        for (var b, a, d = [], c = 12; 0 < c; c--) {
            b = c * (c - 1) / 2 + 1;
            for (var e = 0; e < c; e++)a =
                b + e, d[a] = 12 === c ? g[a] : g[a] + (d[a + c] > d[a + c + 1] ? d[a + c] : d[a + c + 1])
        }
        s = d[1]
    })();


难道真的死路一条了？！ 既然直观的读代码是看不出s和e的值，不如看看运行期的值，但是这所有的代码都是写在function内部做为局部变量，外层代码运行完，就从内存中释放了；不对，这里有click事件，应该会产生闭包，看看代码

    l.onclick = function (b) {
        b = b.target;
        b.classList.contains("able") && q(b)
    };


这个事件回调函数里引用了变量b和q，按照闭包的机制，为了能够访问到外部的变量，这些外部变量在执行完不会被内存释放，那就在运行期通过DevTools来看看，选择一个数值按钮，查看这个dom绑定有哪些事件:


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/f8c17411-5999-4acd-9f52-6210037e76d6/5.3.jpg?resizeSmall&width=832&alpha=)


一层层展开，果然发现了<function scope>闭包，展开，豁然开朗局部变量e和s都出来了；可以看出这个最大值就是2114（通过源码可以知道，这个值是随机生成的，每次刷新页面得到的值是不一样）

目前只得到了s的值，冰山一角，又陷入了困境

### 第二次进攻-巧攻

上一次进攻得到了s和e的值，但是没有办法修改这个值，就没法触发ajax请求，这次进攻，着重思考如何修改s和e的值，但是这些值在运行期没法改，不如直接改源代码，然后使用小提琴代理来运行代码，很快得到了结果，网络错误，呵呵，我异想天开了，如果这样也行，那所以关卡都可以这样了，游戏也就没意义了，巧攻也失败了。

### 第三次进攻-黄昏前的决战

眼看天快黑了，再过半小时就要下班。 第一、二次进攻都是从代码层入手的，现在换个角度思考问题，回到界面，这一关主要考查数学思维、算法能力，哈哈，这里肯定有捷径的

> 想起了高中时做数学题的情景，经常有些求距离、角度啥的选择题，计算太麻烦了，直接画个草图，一度量，再使用各种反正，逆向思维一排除，ABCD四个答案跃然纸上

把高中时做数学题的思维运用在这里试试，要找到路径总值最大的线路，那么这条路径中肯定有很多个点都是比较大的值，何不先把每层的最大值先标注出来，看看有啥规律

![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/cdcda51d-3b06-465d-8e0c-4453b43b2be2/5.4.jpg?resizeSmall&width=832&alpha=)


很明显路线应该是顶往左下来，跟据每行最大值分布，可以试着走走，发现还是有很多组合项，看来还不行，继续观察，既然断言了右边的路是走不通的，何不把右边那些不可能的点去掉，然后在那一行重新选择一个尽量靠左边些的较大的点，通过进一步缩小范围并取两个可能的点选最优的中间路线，最终可以得出如下三条线路（虽然每次刷新页面各个点的值是随机生成的，但是这个分析原理都可以适用）


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/95abb6cf-f936-47f1-ac0c-3a08da725f5a/5.5.jpg?resizeSmall&width=832&alpha=)


比较三条线路值，最大值就出来了，最大值路线如下


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/49181da5-a62c-4fef-9401-6b41035cb8c0/5.6.jpg?resizeSmall&width=832&alpha=)


ok，大功告成，过五关斩六将啊


![Alt text](https://app.yinxiang.com/shard/s1/sh/394cf147-f7d0-4405-bb4e-07d987a93271/b0602f788d2f8f2b1aee3869c2ebc435/res/7d4357e4-522b-4f41-9112-528a90ee26db/6.0.jpg?resizeSmall&width=832&alpha=)

