---
layout: post
title: Ext4表格操作的优化和抽象
category: project
description: 项目背景：随着公司电商业务越做越大，原先购买的E店宝系统已满足不了公司的需要，上层决定开发自已的erp系统。因为是做电商的，所以系统主要围绕交易订单展开，主要模块包括订单处理，物流管理，仓库管理，店铺管理和权根管理，订单数据来自淘宝、京东和公司自己的b2c平台。因为淘宝方面的强硬，所以服务器放在了阿里云的聚石塔上，各平台订单数据通过api抓取，再通过这个Erp系统统一处理。因为之前有使用Ext做过商家系统，所以项目老大华姐安排我做交互较复杂的订单处理模块
---

@(blog)[Ext4]
> 项目背景：随着公司电商业务越做越大，原先购买的E店宝系统已满足不了公司的需要，上层决定开发自已的erp系统。因为是做电商的，所以系统主要围绕交易订单展开，主要模块包括订单处理，物流管理，仓库管理，店铺管理和权根管理，订单数据来自淘宝、京东和公司自己的b2c平台。因为淘宝方面的强硬，所以服务器放在了阿里云的聚石塔上，各平台订单数据通过api抓取，再通过这个Erp系统统一处理。因为之前有使用Ext做过商家系统，所以项目老大华姐安排我做交互较复杂的订单处理模块

Ext庞大且复杂，单单一个组件就可能有一百多个个方法和事件，我了解的也许只是冰山一角，把这次的开发经历总结下来，一来是通过总结理顺思路和加深印象，二来希望得到大牛们的点拔，如有更好的方法或不足的地方，欢迎吐槽。

## 代码结构及主体界面

### 代码结构

![Alt text](https://app.yinxiang.com/shard/s1/sh/a44f2a45-0486-4273-8593-888e2e822017/9909dc30b92f203dff520b058bdac75a/res/ce035766-d6f8-4fb3-b7b7-e1a3e16d1f7a/tree2.jpg?resizeSmall&width=832&alpha=)


订单管理页要展示的数据比较多，交互也比较频繁，为了尽可能多得显示订单，决定把这个订单管理模块独立成为一个子app，代码放于order文件夹下。

### 主界面

![Alt text](https://app.yinxiang.com/shard/s1/sh/a44f2a45-0486-4273-8593-888e2e822017/9909dc30b92f203dff520b058bdac75a/res/f1807bc9-3320-49e5-ac7b-2ec5d3dafbae/order.jpg?resizeSmall&width=832&alpha=)


可以看出来，这个模块分为上中下三块内容，顶部是订单搜索表单，可以跟据时间和订单的各列信息搜索；中间的一块是最重要的订单表格，有很多行和列，一屏显示不完，所以有滚动条，订单表格上有一排工具按扭，多数的交互在这里；底部在选中某一订单后才显示，展示的是所选订单对应的商品信息。 主界面的视图：

    Ext.define('Supplier.view.Order', {
        extend: 'Ext.panel.Panel',
        alias: 'widget.order',
        title: '订单管理',
        id: 'order',
        fixed: true,
        layout: 'border',
        initComponent: function (){
            this.items = [
                {xtype: 'orderSearch'}, // North 订单搜索表单
                {xtype: 'orderList'},    // Center 订单表格
                {xtype: 'orderItem'}     // South 订单对应商品表格
            ];
            this.callParent(arguments);
        }
    });


## 业务逻缉

### 工具栏：

1.  拆分： 把一个订单折分成多个订单。公司的商业模式是代理式，商品在各自商家的仓库中，分布大江南北，客户下的一个订单可能包含来自多个仓库的商品，所以需要把不能打包的订单做拆单处理，分别发货。

2.  合并：把多个订单合并成一个订单。如果一个用户在很短的时间内下了多个订单，并且订单的商品来自同一个仓库，收货信息也是一样的情况下，多个订单就可以合并成一个订单来处理

3.  批量删除：删除订单

 ![Alt text](https://app.yinxiang.com/shard/s1/sh/a44f2a45-0486-4273-8593-888e2e822017/9909dc30b92f203dff520b058bdac75a/res/f023f83a-6bfb-4171-99af-3beebcde0693/comfirm.jpg?resizeSmall&width=832&alpha=)


4.  批量改状态：点击这里后可以弹出一个窗口，批量修改订单信息

 ![Alt text](https://app.yinxiang.com/shard/s1/sh/a44f2a45-0486-4273-8593-888e2e822017/9909dc30b92f203dff520b058bdac75a/res/cece189b-b1d8-4078-96a3-1653fbddb06f/batch.jpg?resizeSmall&width=832&alpha=)


5.  联想单号：快速公司的物流单号都有一定的规侓，如果每次手动一个个输入，效率太低了，所以可以跟据一个开始单号，联想出后续的单号

 ![Alt text](https://app.yinxiang.com/shard/s1/sh/a44f2a45-0486-4273-8593-888e2e822017/9909dc30b92f203dff520b058bdac75a/res/ee53f919-d24d-4e89-90c5-3319c5a6bb6f/express.jpg?resizeSmall&width=832&alpha=)


6.  导入进销存：审单人员审核某订单通个后可以点击这个按钮来提交订单，这样，下一个环节的物流人员就能处理这个订单了

7.  加产品: 给订单增加赠品

 ![Alt text](https://app.yinxiang.com/shard/s1/sh/a44f2a45-0486-4273-8593-888e2e822017/9909dc30b92f203dff520b058bdac75a/res/a72a8d9e-820b-48f4-b6a4-e6a7d944e7b8/good.jpg?resizeSmall&width=832&alpha=)


### 通用逻缉单元提取

在写代码的过程中，发现以上各业务的逻缉代码有很多相似的地方，这此逻缉单元可以提取出来复用。

1.  很多操作都是在选取某一表格对象的基础上丰富的，Ext自带的getCom()方法只能接受字符串的做为参数，有必要编写一个获取表格的方法，可以同时接收字符串和对象做为参数，并返回对应表格对象，这样就不用频繁重新获取表格了：

        /**
        * 根据表格id获取表格对象
        * @param gridId {String/Object} 表格id或表格对象
        * @returns {Object}
        */
        getGridObj: function (gridId) {
            gridId = typeof gridId.isComponent ? gridId : Ext.getCmp(gridId);
            return gridId;
        },


2.  基本所有的操作都是要获取到表格中打勾行的某一字段所组成的数组，所以可以抽象一个获取这个数组的方法：

        /**
         * 选出表格所有选中项的某一字段组成的数组，并返回
         * @param gridId {String/Object} 表格id或表格对象
         * @param type {String} 字段名
         * @returns {Array}
         */
        getGridSels: function (gridId, type) {
            var arr = [],
                sels = this.getGridObj(gridId).getSelectionModel().getSelection();

            Ext.each(sels, function (sel, index, sels) {
                arr.push(sel.get(type));
            });
            return arr;
        },


3.  接着2中的思路，2中写的方法是获取某一字段的数组，而这其中，获取id字段的操作又是频率最高的，所以为了简化，还可以在2的基础上再写一个方法：

        /**
         * 选出表格所有选中项的id字段，组成数组，并返回
         * @param gridId {String/Object} 表格id或表格对象
         * @returns {Array}  表格选中项的唯一字段组成的数组，如[1,2,3,4]
         */
        getGridSelsId: function (gridId) {
            return this.getGridSels(gridId, "id");
        },


4.  除了获取到选中项，有时还需要取消表格的选中项

        /**
         * 根据表格id取消表格选中的项
         * @param gridId {String/Object} 表格id或表格对象
         */
        removeGridSel: function (gridId) {
            this.getGridObj(gridId).getSelectionModel().clearSelections();
        },


5.  有不少的操作是在表格至少有一项选中的情况下才通过的，也有只允许有且只有一项选中，这些逻缉段也可以分隔出来

        /**
         * 判断表格的选中项是否为一
         * @param gridId {String/Object} 表格id或表格对象
         * @returns {boolean}
         */
        isGridSingleSel: function (gridId) {
            return this.getGridObj(gridId).getSelectionModel().getSelection().length === 1;
        },

        /**
         * 判断选取表格是否有选择项
         * @param gridId {String/Object} 表格id或表格对象
         * @returns {boolean}
         */
        isGridSel: function (gridId) {
            return this.getGridObj(gridId).getSelectionModel().getSelection().length > 0;
        },


6.  接着5的思路，在不满足5的条件时，需要给用户一个提示，这一层也是可以通用的，为什么不把5，6的代码写在一个函数里？这里，我考虑到有的场景是只需要判断是否有选中项，而不需要给出提示，所以这二段逻缉需要拆分独立成二块逻缉单元。

        /**
         * 做操作前，检查表格是否有选择项，并弹出提示
         * @param gridId {String/Object} 表格id或表格对象
         * @param msg {String} 没有选中项时的提示信息
         * @returns {boolean}
         */
        checkGridSel: function (gridId, msg) {
            msg = msg || '请至少在表格中选择一项';
            if (!this.isGridSel(gridId)) {
                this.showGridSelErr(msg);
                return false;
            }
            return true;
        },


7.  刷新表格时，很多时候是带了查询参数的，参数来自某一表单，Ext默认的表格行为是刷新后之前的选中项也被选中的了，有时，这是不需要的。

        /**
         * 重载表格数据
         * @param gridId {String/Object} 表格id或表格对象
         * @param formId {String} 表单id
         * @param isRetSel {boolean} 是否取消之前gird中的选择项
         */
        reLoadGird: function (gridId, formId, isRetSel) {
            isRetSel = isRetSel || true;
            this.getGridObj(gridId).getStore().reload({
                params: Ext.getCmp(formId).getValues()
            });
            isRetSel && this.removeGridSel(gridId);
        },


8.  有时传参的地址需要跟据所选项的不同，路径也有一定的变化，比如：

    > /order/1/item /order/2/item /order/1/good /order/2/good

变化的只有订单的id，这里可以写一个返回匿名函数的方法来复用

        /**
         * 通用修改store地址
         * @param reg {Object} 路径匹配正则对象
         * @returns {Function}
         */
        changeStoreUrl: function (reg) {
            return function (store, fld) {
                store.getProxy().url = store.getProxy().url.replace(reg, '/' + fld);
            }
        },


后面就可以这样来使用

        /**
         * 根据某一字段取出订单对应商品信息
         * @param fld {String} 字段名
         */
        getOrderItem: function (fld) {
            //生成修改商品store地址函数
            var changeGoodStoreUrl = Espide.Common.changeStoreUrl(/\/(\w)+(?=\/items)/);
            var store = Ext.getCmp('orderItem').getStore();
            changeGoodStoreUrl(store, fld);
            store.load();
            changeGoodStoreUrl = null;
        },


1.  为了防止误操作，像删除，修改状态等操作在执行前，都需要弹出确认窗品，然后再跟据用户的交互做出相应的响应，除了提示的内容和选择“是”之后的动作不同，其它部分代码都是一样的，所以这里也需要抽象

        /**
         * 通用操作前提示
         * @param options
         */
        commonMsg: function (options) {
            //默认数据
            var defaults = {
                title: '操作确认?',
                msg: '你确定要处理订单吗，处理后不可复原?',
                fn: function () {
                }
            };

            options = Ext.apply(defaults, options);

            Ext.Msg.show(
                {
                    title: options.title,
                    msg: options.msg,
                    buttons: Ext.Msg.YESNO,
                    icon: Ext.Msg.QUESTION,
                    fn: options.fn
                }
            );
        },


2.  接着9的思路，在用户选择“是”按钮后的逻缉也是可以复用的，这里可以写一个通用的doAction方法

        /**
         * 通用表格toolbar操作响应
         * @param options {Object} 传参
         *  options.url {String} 响应路径
         *  options.params {Object} post数据
         *  options.successCall {Function} 返回成功后要制行的代码
         *  options.successTipMsg {String} 返回成功后的提示信息
         * @returns {Function}
         */
        doAction: function (options) {
            var root = this,
                defaults = {
                    url: '/assets/js/order/data/orderList.json',
                    params: {
                        orders: root.getGridSelsId('OrderList')
                    },
                    successCall: function () {
                    },
                    successTipMsg: '订单操作成功'
                };
            //参数合并
            options = Ext.apply(defaults, options);

            return function (btn) {
                //点击”取消“，则返回，不做任何处理
                if (btn != 'yes') return;

                Ext.Ajax.request({
                    url: options.url,
                    params: options.params,
                    success: function (response) {
                        var data = Ext.decode(response.responseText);
                        if (data.success) {
                            root.tipMsg('操作成功', options.successTipMsg);
                            options.successCall();
                        } else {
                            Ext.Msg.show({
                                title: '错误',
                                msg: data.msg,
                                buttons: Ext.Msg.YES,
                                icon: Ext.Msg.WARNING
                            });
                        }

                    },
                    failure: function () {
                        Ext.Msg.show({
                            title: '错误',
                            msg: '服务器错误，请重新提交!',
                            buttons: Ext.Msg.YES,
                            icon: Ext.Msg.WARNING
                        });
                    }
                })
            }
        },


下面的一个例子展示如何在控制器中结合使用这两个方法：

        //导入进销存
        "#importOrder": {
            click: function () {
                var root = this,
                    com = Espide.Common;

                if (!com.checkGridSel('OrderList', '请至少选择一项订单'))  return;

                com.commonMsg({
                    msg: '你确定要把这前订单导入进销存吗?',
                    fn: com.doAction({
                        url: '/assets/js/order/data/orderList.json',
                        successTipMsg: '所选订单导入成功',
                        successCall: root.orderComCallback
                    })
                });
            }
        },


这样做，就把不重要的信息隐藏了，展现出来的就是最重要的交互逻缉代码，简单清淅

1.  为了让别的控制器也能够复用这些代码，我又继续把以上的代码单独抽出来放在common.js文件里，这样，别的控制器也能使用，以下为common.js的结构:

        //定义顶级命名空间
        var Espide = window.Espide || {};

        //表格通用接口，不涉及具体表格操作，只做抽象
        Espide.Common = {

            getGridObj: function (){…},

            getGridSelsId: function (){…},

            …
        }


以上只是部分代码，不全不好理解，有需要的朋友，可以找我要完整的代码

## 后记

在开发这个erp系统时，不断发现有重复的逻缉单元，有的代码则自己写着都觉得恶心不能直视，所以就一步步的优化和抽象，最终就成这样了，以后再有什么新增或要修改的业务使用一段很简短的代码就可以搞定了，过个几个月再修改，代码的逻缉也比较好理解。


[BeiYuu]:    http://beiyuu.com  "BeiYuu"
[1]:    {{ page.url}}  ({{ page.title }})
