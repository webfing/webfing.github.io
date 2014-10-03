---
layout: post
title: IE本地存储方案
description: IE8+及现代浏览器都有了localStorage对象，所以本方案只是针对IE6-7的解决方案，在做全兼容的功能时，先检测localStorage对象，再来检测本方案，注意先后顺序
category: study
---

@(blog)[userData]

> IE8+及现代浏览器都有了localStorage对象，所以本方案只是针对IE6-7的解决方案，在做全兼容的功能时，先检测localStorage对象，再来检测本方案，注意先后顺序

## 代替cookie的原因

* Cookie保存的数据量非常小，只有4kb
* 有禁用Cookie的风险
* Cookie加大网格请求字节

## 简介

* 语法
 HTML声明式:

        <ELEMENT style="behavior:url('#default#userData')">
 Script声明式:

        object.style.behavior = "url('#default#userData')"
        //或者
        object.addBehavior ("#default#userData")

* 属性
 expires 设置或者获取 userData behavior 保存数据的失效日期。

* 方法:

        getAttribute()      //获取指定的属性值。
        load(name)        //从 userData 存储区载入存储的对象数据。
        removeAttribute()   //移除对象的指定属性。
        save(name)        //将对象数据存储到一个 userData 存储区。
        setAttribute()      //设置指定的属性值。

要使用userData存储功能，必须先建立一个HTML标签，然后将behavior:url('#default#userData')样式属性加上去，等于说userData是寄存于HTML标签的，当然不是所有标签都是可以的，仅限于部分标签。

## 封装

下面封装一个userData单体: `userData.js`

    var userData = (function(){

        var store,
            name = 'webfing';

        try{
            init();
        }catch(e){
            return;
        }

        function init(){
            store = document.createElement('input');
            store.type = "hidden";
            store.style.display = "none";
            store.addBehavior("#default#userData"); //关键一句
            var date = new Date();
            date.setDate(date.getDate()+365);   //注意时间加一年的技巧
            store.expires = date.toUTCString();
            document.body.appendChild(store);
        }

        function setItem(key, value){
            store.load(name);
            store.setAttribute(key, value);
            store.save(name);
        }

        function getItem(key, value){
            store.load(name);
            return store.getAttribute(key);
        }

        function removeItem(key){
            store.load(name);
            store.removeAttribute(key);
            store.save(name);
        }

        return {
            setItem: setItem,
            getItem: getItem,
            removeItem: removeItem
        }

    })();

用法:

    if (userData){
        if (!userData.getItem('search')){
            userData.setItem('search', '我来了');
        }else{
            var text = document.createTextNode(userData.getItem('search'));
            document.body.appendChild(text);
        }
    }



参考资料：


[https://github.com/marcuswestin/store.js](https://github.com/marcuswestin/store.js)

[http://mao.li/tag/userdata/](http://mao.li/tag/userdata/)

[http://wangye.org/blog/archives/65/](http://wangye.org/blog/archives/65/)

[http://www.planabc.net/2008/08/05/userdata_behavior/](http://www.planabc.net/2008/08/05/userdata_behavior/)