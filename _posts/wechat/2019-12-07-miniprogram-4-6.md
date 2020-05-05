---
layout: post
title: 【坚果商城实战系列学习】第 4-6 课：前后端交互之商品详情实现 
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 4-6 课：前后端交互之商品详情实现 
---

# 第 4-6 课：前后端交互之商品详情实现 

## 1 逻辑处理

打开 `client` 新建 `models/productModel.js` ,新增
```
import { CloudRequest } from '../utils/cloud-request.js'
class ProductModel extends CloudRequest {
  
  /*********** 新增 *********/  
  /**
   * 根据商品ID 获取商品信息
   * @param {*} product_id 
   * @param {*} callBack 
   */
  getProductById(product_id,callBack) {
    this.request({
      url: "getProductById",
      data:{product_id:product_id},
      success: res => {
        callBack(res)
      }
    })
  }

}
export { ProductModel }
```

##  2 前台数据处理

回到我们 `pages/product/product.js`
```
// pages/product/product.js
import {  ProductModel } from '../../models/productModel.js'
import { CartModel } from '../../models/CartModel.js'
let productModel = new ProductModel()
let cartmodel = new CartModel()
Page({

  /**
   * 页面的初始数据
   */
  data: {
    currentTab: 0,      // tab选项卡
    product:''
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    let product_id = options.product_id
    this._init(product_id)
  },
  // 详情和参数切换
  clickTab: function (e) {
    const currentTab = e.currentTarget.dataset.current
    this.setData({
      currentTab
    })
  },
  _init:function(product_id){
    // 获取商品信息
    productModel.getProductById(product_id,res=>{
      this.setData({
        product:res.result.data.data
      })
    })
  },
  // 购物车
  goCart:function(){   
    wx.switchTab({    
      url: '/pages/cart/cart',
    })
  },
  // 回到首页
  goHome:function(){
    wx.switchTab({
      url: '/pages/index/index',
    })
  },
  // 关注
  focus:function(e){
      wx.showToast({
        icon: 'none',
        title: '暂未开通'
      })
  },
  joinCart:function(){

    cartmodel.add(this.data.product,1,res=>{
      wx.showToast({
        icon:"success",
        context:"添加成功"
      })
    })
  },
  immediately:function(){
    // 数量默认为1
    let count = 1
    wx.navigateTo({
      url: '../order/order?count=' + count + '&productId='+this.data.product._id+ '&from=product'
    });
  }
})
```

`pages/product/product.wxml`
```
<!--pages/product/product.wxml-->
<view class='container'>
  <!-- 商品图片 -->
  <view class='swiper-container'>
       <image src="{{product.product_img}}" class="slide-image" />
  </view>
   <!-- 商品标题  -->
  <view class='title-container'>
    <view class="title">{{product.product_name}}</view>
  </view>
    <view class='price-container'>
    <text class='symbol'>￥</text>
    <text class='price'>{{product.product_sell_price}}</text>
    <text class='market-price'>{{product.product_price}}</text>
  </view>
  <!-- 基本参数 -->
  <view class='parameter-container'>
    <text>快递:49包邮</text>
    <text class='parameter-color'>剩余:{{product.product_stock}}</text>
    <text>总销量:999</text>
  </view>
   <!-- tab选项卡 -->
  <view class='tab-container'>
    <view class="swiper-tab">
        <view class="swiper-tab-item {{currentTab==0?'active':''}}" data-current="0" bindtap="clickTab">商家详情</view>
        <view class="swiper-tab-item {{currentTab==1?'active':''}}" data-current="1" bindtap="clickTab">规格参数</view>
    </view>
    <view class='swiper-tab-show'>
      <view class='dealer-details' wx:if="{{currentTab==0}}">
         <text>商品详情正在更新中！</text>
      </view>
      <view class='format-container' wx:else>
          <text>详情正在更新中!</text>
      </view>
    </view>
  </view>
  
  <!-- 底部 -->
    <view class='bottom-container'>
      <view class='icon-container home' bind:tap='goHome' >
        <icon class='iconfont iconziyuan'></icon>
        <text>首页</text>
      </view>
      <view class="icon-container customer {{focusStatus?'focusActive':''}}" bind:tap='focus'>
        <icon class="iconfont  iconshoucang1 "></icon>
        <text>关注</text>
      </view>
      <view class='icon-container cart' bind:tap='goCart'>
        <icon class='iconfont iconicon01'></icon>
        <text>购物车</text>
      </view>
      <view class='go-container'>
        <view class='joinCart' bind:tap='joinCart'><text>加入购物车</text></view>
        <view class='immediately' bind:tap='immediately'><text>立即购买</text></view>
      </view>
  </view>
</view>

```
## 3 代码解析

#### 3.1 初始数据

商品详情进入的时候需要拿到商品的 id ，通过 id 获取商品的详情。

```
  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    let product_id = options.product_id
    this._init(product_id)
  },
    _init:function(product_id){
    // 获取商品信息
    productModel.getProductById(product_id,res=>{
      this.setData({
        product:res.result.data.data
      })
    })
  },
```

#### 3.2 添加购物车和立即购买

购物车的实现是每次用户点击一次商品加 1 ，并给出用户提示信息。立即购买目前没有数量选择的功能，则是默认给出商品数为 1 并把数量和商品的 id 在跳转的时候一起带入。

```
 // 添加购物车
  joinCart:function(){
    cartmodel.add(this.data.product,1,res=>{
      wx.showToast({
        icon:"success",
        context:"添加成功"
      })
    })
  },
  // 立即购买
  immediately:function(){
    // 数量默认为1
    let count = 1
    wx.navigateTo({
      url: '../order/order?count=' + count + '&productId='+this.data.product._id+ '&from=product'
    });
  }
```

#### 3.3 其他跳转

这里关注因为后台没有实现关联，这个功能就只能目前给出一个提示。

```
 // 购物车
  goCart:function(){   
    wx.switchTab({    
      url: '/pages/cart/cart',
    })
  },
  // 回到首页
  goHome:function(){
    wx.switchTab({
      url: '/pages/index/index',
    })
  },
  // 关注
  focus:function(e){
      wx.showToast({
        icon: 'none',
        title: '暂未开通'
      })
  },
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)