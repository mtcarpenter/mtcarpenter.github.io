---
layout: post
title: 【坚果商城实战系列学习】第 2-8 课：前端之个人中心实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 2-8 课：前端之个人中心实现
---

# 第 2-8 课：前端之个人中心实现

> 所在路径 `client/pages/my/my`

## 1 个人中心基本实现

`my.js`  data 数据如下，目前我们是没有调后台的数据，默认给出默认值方便页面的展示

```
data: {
    userInfo: [],
    defaultImg: '../../images/my/header.png',
    orders: [{
      _id: "9afd9b6a5d297bc20",
      buyer_address: "广东省广州市海珠区新港中路397号",
      buyer_name: "张三",
      buyer_phone: "020-81167888",
      create_time: "2019-07-13T06:35:46.242Z",
      order_amount: 0.1,
      order_status: 0,
      orderdetail: [{
        order_id: "7217ba20a53811e9825141fe9cfd302d",
        product_count: 1,
        product_id: "5cf526aaa87a1a18b6624ae6",
        product_img: "cloud://release-prod.7265-release-prod/product/product-nux@1.png",
        product_name: "花生 300g",
        product_price: 0.1,
      }]
    }]
  }
```

`my.wxml`

```
<!-- pages/my/my.wxml -->
<wxs module="statusModule" src="../../common/wxs/status.wxs"></wxs>
<wxs module="filtersModule" src="../../common/wxs/filters.wxs"></wxs>
<view class='container'>
  <view class='head-container'>
    <view class='header-img'>
      <image src="{{userInfo.length!=0?userInfo.avatarUrl:defaultImg}}"></image>
    </view>
    <img-btn-comp open-type="getUserInfo" bind:getuserinfo="getuserinfo">
      <view class='head-title' slot="info">
        <view class='header-name'>
          <text>{{userInfo.length!=0?userInfo.nickName:'点击登录'}}</text>
        </view>
      </view>
    </img-btn-comp>
  </view>
  <!-- 订单部分 -->
  <view class='my-order-container'>
    <!-- 标签 -->
    <title-bar-comp title='我的订单'></title-bar-comp>
    <!-- 订单列表 -->
    <block wx:for="{{orders}}" wx:key="index">
      <view class="order-item">
        <view class="order-header">
          <view class="order-header-left">
            <text>订单编号:</text>
            <text class="order-no">{{filtersModule.substringto16(item._id)}}</text>
          </view>
          <view class="order-header-right">
            <text class="order-status">{{statusModule.order(item.order_status)}}</text>
          </view>
        </view>
        <view class="order-main">
          <view class="item-left">
            <image src="{{item.orderdetail[0].product_img}}"></image>
          </view>
          <view class="item-right">
            <view>{{item.orderdetail[0].product_name}}</view>
            <view>{{item.orderdetail.length}}件商品</view>
          </view>
        </view>
        <view class="order-bottom">
          <text>实付:￥{{item.order_amount}}</text>
          <view class="pay" data-id="{{item._id}}" >立即支付</view>
        </view>
      </view>
    </block>
  </view>
</view>
```

`my.wxss`

```
/* pages/my/my.wxss */
.container {
  align-items: flex-start;
}

.head-container {
  width: 750rpx;
  height: 266rpx;
  display: inline-flex;
  align-items: center;
  background: linear-gradient(#ff7062, #ff7062);
}

.header-img {
  width: 130rpx;
  height: 130rpx;
  margin: 22rpx 0 0 26rpx;
}

.header-img image {
  width: 130rpx;
  height: 130rpx;
  border-radius: 50%;
}

.head-title {
  margin: 0rpx 0 0 50rpx;
}

.header-name {
  font-size: 48rpx;
  font-weight: 600;
  color: rgba(255, 255, 255, 1);
  line-height: 66rpx;
  text-align: left;
}

.order-item {
  width: 690rpx;
  margin-bottom: 15rpx;
  color: #777;
  background-color: #fff;
  font-size: 28rpx;
  margin-left: 30rpx;
  box-shadow: 0rpx 0rpx 14rpx 0rpx rgba(233, 233, 233, 1);
  margin-top: 20rpx;
}

.order-header {
  display: flex;
  justify-content: space-between;
  border-bottom: 2rpx solid #f5f5f5;
  padding: 20rpx 20rpx;
}
.order-status{
  color: #FC2C1D;
}

.order-main {
  display: flex;
  height: 150rpx;
  color: #666;
  padding: 20rpx 30rpx;
}

.order-main .item-left {
  flex-basis: 150rpx;
  height: 100%;
  background-color: #f5f5f5;
  border-radius: 10rpx;
}

.order-main .item-left image {
  height: 100%;
  width: 100%;
}

.order-main .item-right {
  display: flex;
  margin-left: 20rpx;
  flex-direction: column;
  justify-content: center;
}
.order-main .item-right  image{
  width: 100%;
  height: 100%;
}
.order-bottom {
  border-top: 2rpx solid #f5f5f5;
  padding: 20rpx;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.order-bottom .pay{
  padding: 5rpx 20rpx;
  border-radius:28rpx;
  color: #FF4F69;
  border:2rpx solid #FF4F69;
  text-align: center;
}
```

`my.json`

```
{
  "usingComponents": {
    "title-bar-comp": "/components/title-bar/index",
    "img-btn-comp": "/components/image-button/index"
  }
}
```

## 2 个人中心组件实现

`title-bar-comp` 和 `img-btn-comp` 在前面章节这个已经是实现过，在这里我们就可以直接复用，封装组件的节约我们我们的开发时间。

## 3 wxs的介绍

WXS（WeiXin Script）是小程序的一套脚本语言，结合 `WXML`，可以构建出页面的结构。

WXS 与 JavaScript 是不同的语言，有自己的语法，并不和 JavaScript 一致。 wxs 很多 JavaScript 的类似方法和函数他都没有，目前只支持语法也是相对比较少，但是在实际开发中也够用了。大家可以在[官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxs/ )中了解更多，这个知识点不算很难，大家看下应该就知道怎么用了。

**小案例**

```
<!--wxml-->
<wxs module="filter">
  toFix = function(value) {
    if (value == 0 || !value) {
      return 0
    }
    return parseFloat(value).toFixed(2)
  }
  module.exports.toFix = toFix;
</wxs>
<view> {{filter.toFix(12.568)}} </view>
```

页面输出：

```
12.57
```

`filters.wxs`

```
var filters = {
  toFix: function (value) {
    if (value == 0 || !value){
      return 0
    }
    return parseFloat(value).toFixed(2)
  },
  substringto16:function(value){
    return value.substring(0,16)
  }



}
module.exports = {
  toFix: filters.toFix,
  substringto16: filters.substringto16
}
```

`status.wxs`

```
var status = {
  order:function(num){
    var value = ""
    switch (num) {
      case 0:
        value = "未支付"
        break
      case 1:
        value = "已支付"
        break
      case 2:
        value = "已发货"
        break
      case 3:
        value = "已支付，但库存不足"
        break
    }    
   return value
  }
}
module.exports = {
  order: status.order
}
```

> wxs 还可以实现更多复杂的过程，在这里只是一个很小很小的实现

完成效果如下：

![1563016891721](https://nux.oss-cn-beijing.aliyuncs.com/doc/045.png)



## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)