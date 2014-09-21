---
layout: post
title: 基于Ext4的表格排序功能
category: project
description: 需求背景：电商内部BS版ERP系统-订单管理界面。系统中使用订单管理的用户角色有很多，譬如：物流部，仓库发货员，财务等等，每种角色的操作内容和关注重点不同，仓库发货员更关心订单的收货地址，财务则关注订单金额；同一角色的不同用户也有各自的细分工作和操作习惯，所以有必要开发表格列排序记忆功能，这样用户可以把自己经常操作的列拖到表格的前面，方便查看，用户再登录时，浏览器能够根据用户之前的设置来渲染对应的列顺序。
---

> 需求背景：电商内部BS版ERP系统-订单管理界面。系统中使用订单管理的用户角色有很多，譬如：物流部，仓库发货员，财务等等，每种角色的操作内容和关注重点不同，仓库发货员更关心订单的收货地址，财务则关注订单金额；同一角色的不同用户也有各自的细分工作和操作习惯，所以有必要开发表格列排序记忆功能，这样用户可以把自己经常操作的列拖到表格的前面，方便查看，用户再登录时，浏览器能够根据用户之前的设置来渲染对应的列顺序。

![表格排序](https://app.yinxiang.com/shard/s1/sh/d95978c2-05b5-4bab-97cb-ed490b914909/150042fdf0b73a0199f7a40506edccfa/res/89b51f73-c647-49fc-be91-ce5c33347f79/sort.jpg?resizeSmall&width=832&alpha=)

## 构思

**Ext**已经有列拖动功能，省去了很多开发，那关键就是在拖动事件中能够获取到视图中列的顺序变化值，并通过一种算法得到新的序列，再把新序列ajax保存到服务器上；等表格视图再次渲染时，根据用户id从服务器获取排序，渲染输出表格。

## 最初的实现

### 查找grid组件的api

第一反应应该查找**gird**组件的事件，在**Events**中找到了**columnmove**事件，看了看api，应该就是要找的，columnmove事件还直接给出了formIdx和toIndx参数，哦？顺着这个思路，岂不意味着要相应的算法来重排。（这里其实有更好的实现方法，后面会提到，先按照现在的思路走一遍）

### 编写数组重排序算法

这个算法，可以理解为，把某一个数组元素从一个位置移到另一个位置，得到新数组

    /**
     * 数组移位算法
     * @param arr {Array} 要计算的数组
     * @param form {Number}
     * @param to {Number}
     */
    changeArrPos: function (arr, form, to) {
        var temp, i,
            step = form - to;
        if (step === -1 || step === 0) {
            return;
        }
        //向后移
        if (step < 0) {
            temp = arr[to - 1];
            arr[to - 1] = arr[form];
            for (i = form; i < to - 2; i++) {
                arr[i] = arr[i + 1]
            }
            arr[to - 2] = temp;
        //向前移
        } else {
            temp = arr[form];
            for (i = form; i > to; i--) {
                arr[i] = arr[i - 1];
            }
            arr[to] = temp;
        }
    }


这样在**columnmove**事件中，把**formIdx**,**toIndx**参数传给changeArrPos函数，就可以得到一个排序后的数组。

### 从服务器获取表格排序

页面加载时，先从服务器获取表格排序的数组，等表格视图渲染时，再使用这个排序数组重新生成新的布局，实现如下：

    /**
     * 从服务器获取表格列的顺序
     * @param gridId {String/Object} 表格id或表格对象
     * @returns {Array}
     */
    getGridColumnsData: function (gridId){
        var columns = null;
        Ext.Ajax.request({
            url: Config.getOrderGridColumns+'?'+Config.UserId,
            async: false,
            success: function (response){
                columns = response.responseText;
            }
        })
        return Ext.JSON.decode(columns);
    }


### 当用户发生交互时，把表头排序保存到服务器

    /**
     * 保存表格列的顺序
     * @param gridId {String/Object} 表格id或表格对象
     * @param fromIdx {Number}
     * @param toIdx {Number}
     */
    saveGridColumnsData: function (gridId, fromIdx, toIdx) {

        //获取表格对应的列排序数据
        var columns = this.getGridColumnData(gridId);

        //跟据用户的修改，重新计算排序数组的顺序
        this.changeArrPos(columns, fromIdx, toIdx);

        //ajax数据库存一份列排序数组
        Ext.Ajax.request({
            url: Config.postOrderGridColumns+'?'+Config.UserId,
            params: {
                columns: columns
            }
        });
    }


## 查错及优化

按照上面的思维快速写完了代码，但是途中，感觉很多细节上都还有bug和待改进优化的，于是思路上又捋了一遍，以下为这次的优化。

1.  使用localStorage改进数据存储和读取

    每次渲染表格时都要发一次ajax请求，网格有延迟时用户体验将很不好，何不把用户设置在本地包留一份，用cookie存表头数据，好像大了一些，做为内部系统（用户指定使用chrome），何不使用ie8及以上能够使用的html5 localStorage来储存呢，既使浏览器不支持localStorage或用户换台电脑登录也可以做个判断再从服务器读取,局部如下代码：

        if (window.localStorage && （localColumns = localStorage[gridId] ） && localColumns.length > 0) {
            //尝试读localStorage
            columns = localColumns;
        } else {
            //再尝试读服务器
            Ext.Ajax.request({
                url: Config.getOrderGridColumns +'?'＋ gridId,
                async: false,
                success: function (response) {
                    columns = response.responseText;
                }
            });
        }


2.  getGridColumnsData和saveGridColumnsData方法，都支持传入一个字符串类型的gridId参数,但也许在方法调用的上下文已经取得了这个表格对象，而且以后的同事来修改我的代码时也容易误会以为可以接收对象做参数，所以还得做个参数类型的处理。正如jquery的$方法，可以同时接受对象和字符串做为参数。

        getGridColumnData: function (gridId) {
            //...

            //可以有多种方法
            if (grid.isComponent) {
                gridId = gridId.getId() ||  gridId.getItemId();
            }

            //...

        ｝


3.  表头数组排序的思考。

    之前的数组排序中，使用的算法是：目标元素从form位置移到to位置，form和to之间的元素则依次循环退一位或进一位。直觉告诉我后面的循环应该是可以优化的，想了想，咱的目的就是把数组的某个元素换个位置而已，何不直接把数组截断，然后再从新的位置插入，js内部的算法肯定比咱自己写的算法要高效。虽然这里的优化微乎其微，但做为一种思维方式，我个人觉得还是有必要思考思考的。以下为部分代码：

        //原代码
        if (step < 0) {
            temp = arr[to - 1];
            arr[to - 1] = arr[form];
            for (i = form; i < to - 2; i++) {
                arr[i] = arr[i + 1]
            }
            arr[to - 2] = temp;

        } else {
            temp = arr[form];
            for (i = form; i > to; i--) {
                arr[i] = arr[i - 1];
            }
            arr[to] = temp;
        }

        //改进后
        temp = arr.splice(form,1);
        if (step < 0) {
            arr.splice(to-1,0,temp[0]);
        } else {
            arr.splice(to,0,temp[0]);
        }


改进后的代码不但简洁易懂，而且效率也高了，呵呵，这也只是我的主观判断，未必一定对，而且跟数据的length应该也是有关系。有兴趣的朋友，可以写个例子，相信统计耗时的数据更加有说服力。

## 再次优化

### 表头排序的再思考

1.  取表头的更好方法。

    之前的代码中还有一点感觉不太对，在表格的移动事件中，ext官方给了fromIdx和toIdx参数，我没有太多思考就激进得去想算法的事，现在想来，强大且完善的Ext应该不可能没考虑到这种需求吧。再来看看这个表格移动事件：columnmove ，参数共有五个，第一个肯定是最重要的一个，ct：Ext.grid.header.Container，继续查看这个headercontainer类的方法，有一个getGridColumns方法，api说明:

    > "Returns an array of all columns which appera in the grid's View"

    应该就是这个了，console打印试了试，得到的数据正好是表格视图定义时的columns属性，而且每次移动后得到的这个数组的顺序都会相应的变化,下面是部分代码：

        columnmove: function (ct, column, formIdx, toIdx){
            var colums = ct.getGridColumns();
        }


2.  表头数据的优化

    新的问题来了，上一步中**ct.getGridColumns**方法取的数据正好是gird视图定义时columns属性的全部数据:

        [
            {dataIndex: "id", text: "ID", width: 70, … },
            {dataIndex: "name", text: "商品名字", width: 200,…},
            …,
            {dataIndex: "brand", text: "商品品牌", width: 150,…}
        ]


    显然，这样的数据太大了，我们希望的数据结构应该是这样子的

        ['id', 'name', 'class', ……, 'brand']


    所以还需要一个方法来格式化数据:

        /**
        * 根据表头的详细数组生成某一字段的新数组
        * @param columns {Array} 详数组
        * @param type {String} 要截取的字段
        * @returns {Array} 返回的简单数组
        */
        getGridHead: function (columns, type){
            var arr = [];
            Ext.each(columns, function (col, index, cols){
                arr.push(col[type]);
            });
            return arr;
        }

        …

        var columns = getGridHead(ct.getGridColumns(), 'dataIndex');


### 共用函数的提取及接口的实现

写表格的操作时，发现有很多部分的逻缉都是一样的，只有部分代码不同，完全可以通过抽象来优化。
这部分涉及到其它功能的解讲和整个系统代码的结构优化，将在[Ext4表格操作的优化和抽象][2]给出详细分析。

 [1]: http://www.webfing.com/wp-content/uploads/2013/12/sort.jpg
 [2]: http://www.webfing.com/ext4-grid-optimize/

[BeiYuu]:    http://beiyuu.com  "BeiYuu"
[1]:    {{ page.url}}  ({{ page.title }})
