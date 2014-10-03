---
layout: post
title: sku算法思考
category: project
description: SKU=Stock Keeping Unit（库存量单位），一个商品的属性会有若干个纬度，比如iphone手机，颜色纬度有：黑、白；存储容量纬度有：8GB、16GB、32GB；运营商纬度有：电信、联通、移动。一串完整的属性：黑色-8GB-电信的iphone称为一个sku，数据库中商品信息是按sku来存储的，在商品详情页的前端的交互中，如何确定一个商品的哪些sku属性是可选的，及他们的价格是多么，将是一件复杂的事情
---

@(blog)[sku, arithmetic]

> [SKU](http://baike.baidu.com/view/276922.htm?fr=aladdin)=Stock Keeping Unit（库存量单位），一个商品的属性会有若干个纬度，比如iphone手机，颜色纬度有：黑、白；存储容量纬度有：8GB、16GB、32GB；运营商纬度有：电信、联通、移动。一串完整的属性：黑色-8GB-电信的iphone称为一个sku，数据库中商品信息是按sku来存储的，在商品详情页的前端的交互中，如何确定一个商品的哪些sku属性是可选的，及他们的价格是多么，将是一件复杂的事情


![Alt text](http://king-images.qiniudn.com/sku-select.png)
参考网址：http://www.yixinshang.com/product/1258

##问题分析

###基本数据

后台返回的skumap信息：

    var skuMap = {
      "227633277860,236223212497": {    //属性，逗号分隔
        "id": 3238, //ID
        "sellPrice": "298.00",  //售价
        "skuImageList": [
          "http://img01.yixinshang.com/1258_main_18.jpg" //对应图片
        ],
        "stock": 10 //库存
      },
      "227633277860,236223212496": {
        "id": 3239,
        "sellPrice": "298.00",
        "skuImageList": [
          "http://img01.yixinshang.com/1258_main_12.jpg"
        ],
        "stock": 10
      }
    }

页面html代码：

![Alt text](http://king-images.qiniudn.com/sku-select-code.png)

对应属性id：

    红色： 236223212496
    黑色： 236223212497
    均码： 227633277860

###思路分析
最直观的方法就是在用户选择属性后，查出到前已选择属性，然后遍历未选择属性，算出所有可能的组合，然后去skumap查对应的sku，求出价格和库存，比如：
当用户选择了“均码”，遍历其它纬度的属性，就可以枚举出“均码-黑色”及“均码-红色”的组合，然后跟据其属性id组合，在skumap对象中找相应的sku信息。

好像也很简单，没有必要上升到算法的层面上，但是，想想，如果商品的属性纬度更大些，比如三层或四层那么匹配组合时的循环和遍历会特别多，而且很次的交互都会有重复的计算。

##算法推算
###思路
上述说到，难点在于，用户的选择是无序随意的，会产生很多组合，在计算组合的库存和价格时会消耗很多时间并且计算会出现重复。那么我们何不在页面初始化后，交互前就先把所有可能的组合一次都计算好，存为一个对象（数据字典），这样，在用户交互时，只需遍历一次，然后把组合值去数据字典里找就可以了，大至思路是：

由完整的sku信息：

    var skuMap = {
      "227633277860,236223212497": {    //属性，逗号分隔
        "id": 3238, //ID
        "sellPrice": "298.00",  //售价
        "skuImageList": [
          "http://img01.yixinshang.com/1258_main_18.jpg" //对应图片
        ],
        "stock": 10 //库存
      },
      "227633277860,236223212496": {
        "id": 3239,
        "sellPrice": "298.00",
        "skuImageList": [
          "http://img01.yixinshang.com/1258_main_12.jpg"
        ],
        "stock": 10
      }
    }

枚举出所有可能的组合值：

    var skuDD = {
    	"227633277860": {stock: 20,...},
    	"236223212497": {stock: 20,...},
    	"236223212496": {stock: 20,...},
    	"227633277860,236223212497": {stock: 10,...},
    	"227633277860,236223212496": {stock: 10,...}
    }

属性纬度越大，这种算法的效率越明显

###实践

根据上面的推演，把复杂的问题抽象成如下模型：

数据原型：`[a, b, c, d]`分别对应不同纬度里的一个值，通过算法枚举出如下组合字典：

    a, b, c, d,
    ab, ac, ad, bc, bd, cd,
    abc, abd, acd, bcd,
    abcd

观察上面的值，整理如下：

    a -> ab -> abc -> abcd
            -> abd
      -> ac -> acd
      -> ad
    b -> bc -> bcd
      -> bd
    c -> cd

    d

算法实现如下：

    var arr = ['a', 'b', 'c', 'd'];
    var len = arr.length;

    var sku = [];

    function skuRe(item, index){
        item = item || '';
        index = index || 0;
        if (index==len) return;
        for (var i=index; i<len; i++){
            var newItem = item+arr[i];
            sku.push(newItem);
            skuRe(newItem, ++index);
        }
    }

    skuRe();
    alert(sku); // ["a", "ab", "abc", "abcd", "abd", "ac", "acd", "ad", "b", "bc", "bcd", "bd", "c", "cd", "d"]

