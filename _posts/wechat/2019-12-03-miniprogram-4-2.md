---
layout: post
title: 【坚果商城实战系列学习】第 4-2 课：前后端交互之首页实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 4-2 课：前后端交互之首页实现
---

# 第 4-2 课：前后端交互之首页实现

## 1 逻辑处理

`client` 新建 `models/IndexModel.js`
```
import { CloudRequest } from '../utils/cloud-request.js'
class IndexModel extends CloudRequest {
  
  /**
   * 获取首页轮播
   * @param {*} callBack 
   */
  getBanner(callBack) {
    this.request({
      url: "getBanner",
      success: res => {
        callBack(res)
      }
    })
  }
  
  /**
   * 获取主题 
   * @param {*} callBack 
   */
  getTheme(callBack){
    this.request({
      url: "getTheme",
      success: res => {
        callBack(res)
      }
    })
  }

  /**
   * 获取最新商品
   * @param {*} callBack 
   */
  getProductNew(callBack){
    this.request({
      url: "getProductNew",
      success: res => {
        callBack(res)
      }
    })
  }

}

export { IndexModel }
```
> 大家看到这个有没有有点熟悉又是不熟悉，在其他里面我们可能一个 js 里面有很多方法，我们需要每一个方法导出才能使用，然后 require 引入直接使用，类是通过导出类名，import 引入实例化后我们才能使用。希望大家在实际的过程中不要搞混了。 callBack 是回调函数，将数据抛给上层，类是 ES6特性，如果不是很熟悉可以回头看下。

## 2 前台数据处理

回到我们 `pages/index/index.js` ，下面我贴出实现的代码，直接在方法上面注释说明
```
// pages/index/index.js
// 引入我们需要的类 
import { IndexModel } from "../../models/IndexModel.js"
// 使用钱需要实例化才能使用
let index = new IndexModel()
Page({

  /**
   * 页面的初始数据
   */
  data: {
    indicatorDots: true, //是否显示面板指示点
    autoplay: true, //自动轮播
    interval: 3000, // 自动切换时间间隔
    duration: 1000, // 滑动动画时长
    circular: true,//是否采用衔接滑动 
    themes:[
      { theme_icon: 'images/theme@1.png', theme_name: '新品糖果', theme_type:1},
      { theme_icon: 'images/theme@2.png', theme_name: '精品果干', theme_type: 2 },
      { theme_icon: 'images/theme@3.png', theme_name: '美味坚果', theme_type: 3 },
      { theme_icon: 'images/theme@4.png', theme_name: '优质推荐', theme_type: 4 },
    ],
    banners:[],
    products:[]

  },

  /**
   * 生命周期函数--监听页面加载
   */
  onLoad: function (options) {
    // _下划线开头 代表私有方法 我们将所有的进来加载数据存放在_init方法
    this._init()
  },
  // 页面初始化数据
  _init:function(){ 
    // 轮播图 在头部我们实例化 复制给index 在这里我们直接通过实例名称调用里面的方法，res=>{} 取出我们回调函数的值，取出我们想要的结果，在setData
    index.getBanner(res => {
      this.setData({
        banners :  res.result.data.data
      })
      
    })
    // 主题
    index.getTheme(res =>{
      this.setData({
        themes :  res.result.data.data
      })
      
    })
    // 最新商品
    index.getProductNew(res=>{
      this.setData({
        products :  res.result.data.data
      })
     
    })
  },
  /**
   * 主题跳转
   */
  themeNavigation: function (event) {
    let theme_type = indexModel.getDataSet(event, "theme_type")
    wx.navigateTo({
      url: '../theme/theme?theme_type=' + theme_type,
    })
  },
  // 跳转商品详情
  productDetails: function (e) {
    this._navProductDetail(e.detail.productId)
  },
  productBanner: function (event) {
    let product_id = indexModel.getDataSet(event, "productid")
    this._navProductDetail(product_id)
  },
  // 跳转详情
  _navProductDetail: function (product_id) {
    wx.navigateTo({
      url: '/pages/product/product?product_id=' + product_id,
    })
  }
})
```
> 在这里我没有一个的编写对接主要是我们在前端编写的时候静态页面已经完成，后台也完成并测试了，只剩下的数据交互了，取出数据渲染数据。可能篇幅就比较大，在这里我更多的时候采用注释的形式，在 `cloud-request` 封装的 getDataSet 只要继承了 `cloud-request` 我们就能直接使用，不需要再打印出来复制属性取值。大家在直接开发的过程中私有方法，一定记得使用 `_` 下划线开头，这样在我们实际开发中也能养成良好的开发习惯。

`pages/index/index.wxml`
```
<!--pages/index/index.wxml-->
<view class='container'>
   <!--1、轮播图 -->
  <swiper class='swiper-container'  indicator-dots="{{indicatorDots}}" indicator-active-color='#fff'
    autoplay="{{autoplay}}" interval="{{interval}}" duration="{{duration}}" circular="{{circular}}">
    <block wx:for="{{banners}}" wx:key="">
      <swiper-item class='swiper-item' data-productId='{{item.product_id}}' bind:tap="productBanner">
        <image  class='swiper-img' src="{{item.image}}"  mode='scaleToFill'  />
      </swiper-item>
    </block>
  </swiper>
  <!-- 2、主题栏 -->
  <view class='theme-container' >
    <block wx:for="{{themes}}" wx:key="index" >
     <view class='theme-box' data-themeType="{{item.theme_type}}" bind:tap='themeNavigation'>
        <view class='theme-icon'>
          <image src='{{item.theme_icon}}'></image>
        </view>
        <text>{{item.theme_name}}</text>
     </view>
    </block>     
  </view>
  <!-- 分割线 -->
  <view class='dividing-line'></view>
  <!-- 3、最新上线 -->
  <view class='products-latest-container'>
    <title-bar-comp title='最新上线'></title-bar-comp>
    <block wx:for="{{products}}" wx:key="key">
        <product-cloumn-comp product="{{item}}" bind:productDetails="productDetails"></product-cloumn-comp>
    </block>
  </view>
</view>

```
```
<product-cloumn-comp product="{{item}}" bind:productDetails="productDetails"></product-cloumn-comp>`
product="{{item}}" product：组件属性对应，{{item}} ：参数传递
bind:productDetails="productDetails" 
bind:productDetails:中的productDetails取自triggerEvent(组件事件通信)，
后面的productDetails：表示wxml的对应的js中方法
```
triggerEvent如果还不了解的，这里是官方[组件间通信与事件组件间通信](https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/events.html)

` components/behaviors/product-behavior.js` 商品信息 Behavior ，在这里我们之前的新增 `properties` 和 `methods`
```
module.exports = Behavior({
  behaviors: [],
  properties: {
    product: { // 属性名
      type: Object,
      value: '', // 属性初始值（可选），如果未指定则会根据类型选择一个
      observer: function (product) {
      }
    }
  },
  methods: {
    // 商品详情
    productDetails: function () {
      // 返回 
      this.triggerEvent("productDetails", {
        productId: this.data.product._id
      }, {})
    }
  }
})

```


`components/product-column/index.wxml`
```
<!--components/product-column/index.wxml-->
 <!-- 商品展示左右分隔 product-left product-left product-right  -->
<view class='container'>
  <!-- 商品展示左边 -->
  <view class='product-left'>
    <view class='product-img' >
      <image src="{{product.product_img}}"></image>
    </view>
  </view>
  <!-- 商品展示右边 -->
  <view class='product-right'>  
    <!-- 商品基本信息上部分 -->
    <view  ><text class='product-title'>{{product.product_name}}</text></view>
    <!-- 商品基本信息下部分 -->
    <view class='product-content' >
      <!-- 市场价  -->
      <view class='market-price-container'>
         <text>市场价:</text>
         <text>￥</text>
         <text>{{product.product_price}}</text>
      </view>
      <!-- 优惠价 -->
      <view class='discount-price-container'>
        <view class='discount-price-left'>
           <text class='discount-price-desc' >优惠价:</text>
           <text class='discount-price-symbol' >￥</text>
           <text class='discount-price' >{{product.product_sell_price}}</text>
        </view>
        <view class='discount-price-right' bind:tap='productDetails'>
            <button class='go'>立即购买</button>
        </view>
      </view>
    </view>
  </view>
</view>

```
> 样式在之前静态页面已经完成

- 类的继承和方法的实现，callback 回调返回结果，引用后实例化之后调用方法。
- 页面也组件的通信，页面参数传递组件 properties 需要对应，组件事件通过 triggerEvent 通信。

## 3 代码分解

### 3.1 静态数据分析

```
themes: [
	{ theme_icon: 'images/theme@1.png', theme_name: '新品糖果', theme_type: 1 },
	{ theme_icon: 'images/theme@2.png', theme_name: '精品果干', theme_type: 2 },
	{ theme_icon: 'images/theme@3.png', theme_name: '美味坚果', theme_type: 3 },
	{ theme_icon: 'images/theme@4.png', theme_name: '优质推荐', theme_type: 4 },
]
```

因为在很多的时候，比如菜单和分类都是前台静态固定的，在这里我先静态在动态获取，这样我后台修改数据就不需要前台重新打包上传，也能在页面加载的时候缓解页面的渲染时间。

### 3.2 初始化 _init

页面加载初始化数据处理使用 _init 方法，提取之后我们的 onLoad 就看起来简洁。 

```
 // 页面初始化数据
  _init:function(){ 
    // 轮播图 在头部我们实例化 复制给index 在这里我们直接通过实例名称调用里面的方法，res=>{} 取出我们回调函数的值，取出我们想要的结果，在setData
    index.getBanner(res => {
      this.setData({
        banners :  res.result.data.data
      })
      
    })
    // 主题
    index.getTheme(res =>{
      this.setData({
        themes :  res.result.data.data
      })
      
    })
    // 最新商品
    index.getProductNew(res=>{
      this.setData({
        products :  res.result.data.data
      })
     
    })
  }
```

首页的轮播、主题、最新商品都是通过后台读取，上面的代码里面 res 则是我们在 model 中 callback 回调的值，然后在通过小程序的 `this.setData` 将返回的结果赋值，在页面上通过则组要 `{{}}` 渲染数据。

### 3.3 页面跳转

首页一般为展示关键性数据，作为软件的一个引导，进一步的操作我们都需要跳转，页面与页面的跳转我们需要参数，在小程序中获取参数的值如下：

```
data-自定义名 = 值   // data 为微信小程序提供的一种赋值的格式

data-themeType="{{item.theme_type}}"
```

通过绑定进行方法的调用, event 获取页面 data 的值

```
 //wxml 
 bind:tap='themeNavigation'
 // themeNavigation => themeNavigation 对应
 // js
 themeNavigation:fucntion(event){
     
 }
```

 使用 `cloud-request` 的 getDataSet 方法获取事件绑定的值。

```
  /**
   * 主题跳转
   */
  themeNavigation: function (event) {
    let theme_type = indexModel.getDataSet(event, "theme_type")
    wx.navigateTo({
      url: '../theme/theme?theme_type=' + theme_type,
    })
  }
```

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)