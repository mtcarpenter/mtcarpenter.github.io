---
layout: post
title: 【坚果商城实战系列学习】第 4-4 课：前后端交互之主题实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 4-4 课：前后端交互之主题实现
---

# 第 4-4 课：前后端交互之主题实现

> 在这里我为了和底部的菜单栏区分，我把首页轮播下面的四个菜单称为主题，在日常的开发中我们商品有分类是必不可少的。为了展示数据这里我新建的集合 `productTheme` 随机关联几条商品信息。

## 1 逻辑处理

`client` 新建 `models/productModel.js`
```
import { CloudRequest } from '../utils/cloud-request.js'
class ProductModel extends CloudRequest {

  /**
   * 根据主题类型获取商品信息
   * @param {*} theme_type  主题类型
   * @param {*} callBack 
   */
  getThemeProduct(theme_type,callBack){
    theme_type = parseInt(theme_type) 
    this.request({
      url:"getThemeProduct",
      data:{theme_type:theme_type},
      success:res=>{
        callBack(res)
      }
    })
  }
  
}
export { ProductModel }

```

## 2 前台数据处理

回到我们 `pages/theme/theme.js`
```
// pages/theme/theme.js
import {  ProductModel } from '../../models/productModel.js'
let productModel = new ProductModel();
Page({

  /**
   * 页面的初始数据
   */
  data: {
    products:[]
  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    this._init(options.theme_type)
  },
  // 初始化
  _init:function(theme_type){
    // 根据类型获取商品
    productModel.getThemeProduct(theme_type,res=>{
      this.setData({
        products : res.result.data.data
      })
    })
  },
   // 跳转商品详情
   productDetails:function(e){
    //TODO: 跳转详情
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + e.detail.productId,
    })
  }
  
})
```


`pages/theme/theme.wxml`
```
<!--pages/theme/theme.wxml-->
<view class='container'>
  <block wx:for="{{products}}" wx:key="key">
    <product-row-comp product="{{item}}" bind:productDetails="productDetails"></product-row-comp>
  </block>
</view>
```
> wxml：当前页面和组件通信，见前后端交互之首页实现

` components/behaviors/product-behavior.js`
> 商品信息Behavior公用，跟前后端交互之首页实现一样，这里就不在贴出代码

`components/product-row/index.wxml`
```
<!--components/product-row/index.wxml-->
<view class='container'  bind:tap='productDetails'>
  <view class='product-top' >
    <image src="{{product.product_img}}"></image>
  </view>
  <view class='product-title' >
    <text>{{product.product_name}}</text>
  </view>
  <view class='market-price' >
    <text >￥</text>
    <text>{{product.product_price}}</text>
  </view>
  <view class='product-container'>
    <view class='product-price'>
      <text >￥</text>
      <text>{{product.product_sell_price}}</text>
    </view>
    <view class='product-cart' >
      <icon class='iconfont icongouwuche'></icon>
    </view>
 </view>
</view>


```

> 样式在之前静态页面已经完成

## 3 代码分解

这个章节想对简单获取类型商品、跳转商品详情。

### 3.1 初始化数据

在页面加载的时候根据商品类型，`theme_type` 为主页跳转页面的路径参数，路径参数取出来的时候是字符，而后台主题类型是数值型，所以在这里参数的时候我们需要通过 `parseInt` 改变类型。

```
 // 初始化
  _init:function(theme_type){
    // 根据类型获取商品
    productModel.getThemeProduct(theme_type,res=>{
      this.setData({
        products : res.result.data.data
      })
    })
  }
  
  // molde 层数据
   getThemeProduct(theme_type,callBack){
   // 类型改变
    theme_type = parseInt(theme_type) 
    this.request({
      url:"getThemeProduct",
      data:{theme_type:theme_type},// 将 thme_type 传值到后台
      success:res=>{
        callBack(res)
      }
    })
  }
```

### 3.2 跳转商品详情

```
   // 跳转商品详情
   productDetails:function(e){
    //TODO: 跳转详情
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + e.detail.productId,
    })
  }
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)