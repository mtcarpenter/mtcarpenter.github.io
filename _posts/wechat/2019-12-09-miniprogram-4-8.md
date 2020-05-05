---
layout: post
title: 【坚果商城实战系列学习】第 4-8 课：前后端交互之个人中心实现  
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 4-8 课：前后端交互之个人中心实现  
---

# 第 4-8 课：前后端交互之个人中心实现  

## 1 逻辑处理

打开 `client` 新建 `models/OrdelModel.js` ,新增
```
import { CloudRequest } from '../utils/cloud-request.js'
class OrderModel extends CloudRequest {

    /**
     * 查询订单
     * @param {*} callBack 
     */
    getOrderList(callBack) {
        this.request({
            url: "getOrderList",
            success: res => {
                callBack(res)
            }
        })
    }

}

export { OrderModel }
```

##  2 前台数据处理

回到我们 `pages/my/my.js`
```
// pages/my/my.js
import {
  OrderModel
} from '../../models/OrdelModel.js'
let orderModel = new OrderModel();
Page({

  /**
   * 页面的初始数据
   */
  data: {
    userInfo: [],
    defaultImg: '../../images/my/header.png',
    orders: []
  },
  /**
   * 生命周期函数--监听页面显示
   */
  onShow: function () {
    this._init();
  },
  // 初始化
  _init: function () {
    wx.getSetting({
      success:(res)=> {
        if (res.authSetting['scope.userInfo']) {
          // 获取订单信息
          orderModel.getOrderList(res => {
            this.setData({
              orders: res.result.data.data
            })
            console.log(this.data.orders)
          })

          // 获取用户信息  
          wx.getUserInfo({
            success: (res) =>{
             this.setData({
              userInfo:res.userInfo
             })
            }
          })
        }
      }
    })
  },
  // 订单页面
  pay: function (event) {
    let id = orderModel.getDataSet(event, 'id')
    wx.navigateTo({
      url: '/pages/order/order?id=' + id
    })
  },
  // 用户信息获取
  getuserinfo:function(event){
    this.setData({
      userInfo:event.detail.userInfo
    })
  }

})
```

`pages/my/my.wxml`
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
          <view class="pay" data-id="{{item._id}}" bind:tap="pay">立即支付</view>
        </view>
      </view>
    </block>
  </view>
</view>
```
>  复用组件 `/components/title-bar/index` 和 `/components/image-button/index`

## 3 代码分解

### 3.1 初始化数据

在订单获取之前判断用户是否授权，如果授权则调用后台获取信息，并获取用户信息。

```
  // 初始化
  _init: function () {
    wx.getSetting({
      success:(res)=> {
        if (res.authSetting['scope.userInfo']) {
          // 获取订单信息
          orderModel.getOrderList(res => {
            this.setData({
              orders: res.result.data.data
            })
          })

          // 获取用户信息  
          wx.getUserInfo({
            success: (res) =>{
             this.setData({
              userInfo:res.userInfo
             })
            }
          })
        }
      }
    })
  },
```

### 3.2 订单页面

获取页面的订单 id ，跳转到商品详情。

```
 // 订单页面
  pay: function (event) {
    let id = orderModel.getDataSet(event, 'id')
    wx.navigateTo({
      url: '/pages/order/order?id=' + id
    })
  },
  // 用户信息获取
  getuserinfo:function(event){
    this.setData({
      userInfo:event.detail.userInfo
    })
  }
```

### 3.3 用户信息获取 

用户信息和地址信息一样，调用需要微信 `open-type` 传入对应的类型才可以。 

```
  // 用户信息获取
  getuserinfo:function(event){
    this.setData({
      userInfo:event.detail.userInfo
    })
  }
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)