---
layout: post
title: 【坚果商城实战系列学习】第 2-2 课：前端之首页实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 2-2 课：前端之首页实现
---

# 第 2-2 课：前端之首页实现

> 所在路径`client/pages/index/index`

## 1 轮播实现

`index.js` data 数据如下，目前我们是没有调后台的数据，默认给出默认值方便页面的展示

```
 data: {
    indicatorDots: true, //是否显示面板指示点
    autoplay: true, //自动轮播
    interval: 3000, // 自动切换时间间隔
    duration: 1000, // 滑动动画时长
    circular: true, //是否采用衔接滑动 
    banners: [{
        image: "cloud://release-prod.7265-release-prod/banner/banner@1.png",
        name: "精美花生",
        product_id: "5cf38858a87a1a18b65aefbe"
      },
      {
        image: "cloud://release-prod.7265-release-prod/banner/banner@2.png",
        name: "精美开心果",
        product_id: "5cf38858a87a1a18b65aefca"
      },
      {
        image: "cloud://release-prod.7265-release-prod/banner/banner@3.png",
        name: "精美杏仁",
        product_id: "5cf38858a87a1a18b65aefc2"
      }
    ]
  },
```

`index.wxml`

```
<!--pages/index/index.wxml-->
<view class='container'>
   <!--1、轮播图 -->
  <swiper class='swiper-container'  indicator-dots="{{indicatorDots}}" indicator-active-color='#fff'
    autoplay="{{autoplay}}" interval="{{interval}}" duration="{{duration}}" circular="{{circular}}">
    <block wx:for="{{banners}}" wx:key="">
      <swiper-item class='swiper-item'>
        <image  class='swiper-img' src="{{item.image}}"  mode='scaleToFill'  />
      </swiper-item>
    </block>
  </swiper>

</view>

```

`index.wxss`

```
/* pages/index/index.wxss */
.container{
  width:750rpx;
  min-height: 100%;
  display:flex;
  flex-direction: column;
  background-color: #fff;
}
/* 轮播图 */
.swiper-container{
  height: 360rpx;
  width: 100%;
}
.swiper-item{
  width: 750rpx;
  height:100%;
  border-top-left-radius: cover;
}
.swiper-item image{
  width: 100%;
  height: 100%;
}

```

这时我们就完成我们的轮播了，运行效果图图下：

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/036.png)

## 2 主题实现

主题所需图片存放在 `pages/index/images` 下面

`index.js`

```
data:{
   /**
   * 其他部分省略
   */
   themes: [
      { theme_icon: 'images/theme@1.png', theme_name: '新品糖果', theme_type: 1 },
      { theme_icon: 'images/theme@2.png', theme_name: '精品果干', theme_type: 2 },
      { theme_icon: 'images/theme@3.png', theme_name: '美味坚果', theme_type: 3 },
      { theme_icon: 'images/theme@4.png', theme_name: '优质推荐', theme_type: 4 },
    ],
}
```

`index.wxml`

```
<!--pages/index/index.wxml-->
<view class='container'>
  <!-- 轮播图代码 -->
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
</view>
```

`index.wxss`

```
/* 主题栏样式 */
.theme-container{
  width: 660rpx;
  height: 180rpx;
  display: inline-flex;
  background-color: #fff;
  justify-content: space-between;
}
.theme-box{
  display: flex;
  flex-direction: column;
  justify-content: center;  
  align-items: center; 
}
.theme-icon{
  width: 90rpx;
  height: 90rpx;
}
.theme-icon image{
  width: 90rpx;
  height: 90rpx;
}
.theme-box text{
  height:28rpx;
  font-size:24rpx;
  color:rgba(83,82,83,1);
  margin-top: 8rpx;
}
```

此段代码运行效果如下：

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/037.png)

## 3 最新上线

这部分包括标题栏和商品展示信息。会复用的东西我们写一个组件方便后面的扩展。

新建 `components` 文件夹在 miniprogram 下面存放所有的组件。

首先我们实现标题栏组件，右击 `components` 新建一个文件夹 `title-bar` ，并选择新建 `component` ，输入 `index`

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/038.png)

以相同的方式创建一个商品信息相关得组件，组件名为 `product-column` ,如果还不了解组件的同学可以看下[官方实例](https://developers.weixin.qq.com/miniprogram/dev/reference/api/Component.html )，组件跟我们的页面列表的 js 是有所不同的，在上句的链接官方的文档中有所介绍，接下来我们也会通过实际开发给大家进一步了解。

`product-column` 商品的相关得组件会有多个，但是我们的数据可能是一样的，这个时候我们就可以使用官方的`Behavior` ([官方说明文档](https://developers.weixin.qq.com/miniprogram/dev/reference/api/Behavior.html ))，后面在时间引用的时候大家可以多注意下。

`title-bar/index.js`

```
 properties: {
    title: String
  }
```

`titleBar/index.wxml`

```
<view class='container'>
  <view class='line'></view>
  <text class='text'>{{title}}</text>
</view>
```

`titleBar/index.wxss `

```
/* components/titleBar/index.wxss */
.container{
  display: flex;
  align-items: center;
  justify-content: flex-start;
  padding-left: 32rpx;
  margin-top: 35rpx;
}
.line{
  width:6rpx;
  height:32rpx;
  background:rgba(60,185,66,1);
  border-radius:6rpx;
}
.text{
  padding-left: 19rpx;
  line-height:30rpx;
  font-weight:400;
  font-size:32rpx;
  color:rgba(56,56,57,1);
}
```

商品信息相关的组件公用的数据部分

`product-behavior.js`

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

  }
})
```

`product-column/index .js` 并引入上面的 `behavior`

```
// components/product-column/index.js
let productBehavior = require('../behaviors/product-behavior.js')
Component({
  /**
   * 组件的属性列表
   */
  properties: {

  },
  behaviors: [productBehavior],
  /**
   * 组件的初始数据
   */
  data: {

  },

  /**
   * 组件的方法列表
   */
  methods: {

  }
})
```

`product-column/index .wxml`

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

`product-column/index .wxss`

```
/* components/product-column/index.wxss */
/* 容器盒子 */
.container{
  display: inline-flex;
  width:683rpx;
  margin-left: 32rpx;
  border-bottom:2rpx solid rgba(220,220,220,1);
  background:#fff;
}
/* 展示商品左边布局 */
.product-left{
  margin: 34rpx 0 30rpx 0;
}
.product-img image{
  width:180rpx;
  height:180rpx;
  border-radius:10px;
}
/* 商品右边部分显示 */
.product-right{
  width: 100%;
  height:180rpx;
  margin:34rpx 0 0 30rpx;
  display: inline-flex;
  flex-flow: column;
  justify-content: space-between;
}
/* 商品名 */
.product-title{
  font-size:30rpx;
  font-weight:400;
  color:#161514;
}
/* 市场价样式 */
.market-price-container{
  margin-bottom: -10rpx;
}
.market-price-container text{
  height:23rpx;
  font-size:24rpx;
  font-weight:400;
  color:#9B9B9B;
 
}
/* 优惠价格 */
.discount-price-container{
  width: 100%;
  display: inline-flex;
  justify-content: space-between;
  align-items: flex-end;
}
.discount-price-desc{
  height:23rpx;
  font-size:24rpx;
  font-weight:400;
  color:#424242;
}
.discount-price-symbol,.discount-price{
  font-size:30rpx;
  font-weight:400;
  color:#FF5251;
}
.go{
  width:156rpx;
  height:56rpx;
  line-height: 56rpx;
  background:rgba(255,79,80,1);
  border-radius:10rpx;
  color: #fff;
  font-size: 24rpx;
}
```

接下来在 index 中引入组件

`index.json`

```
{
  "usingComponents": {
    "title-bar-comp": "/components/title-bar/index",
    "product-cloumn-comp": "/components/product-column/index"
  }
}
```

`usingComponents`冒号前面的为当前页面需要使用的组件名称，后面的是组件的路径。引入之后我们直接就可以在页面上使用了。

`index.js`

```
 data: {
    products: [{
        _id: "5cf526aaa87a1a18b6624ae6",
        product_description: "",
        product_img: "cloud://release-prod.7265-release-prod/product/product-nux@1.png",
        product_name: "花生 300g",
        product_price: 0.1,
        product_sell_price: 0.1,
        product_stock: 100
      },
      {
        _id: "5cf526aaa87a1a18b6624ae8",
        product_description: "",
        product_img: "cloud://release-prod.7265-release-prod/product/product-nux@2.png",
        product_name: "夏威夷果 120g",
        product_price: 0.1,
        product_sell_price: 0.1,
        product_stock: 100
      },
      {
        _id: "5cf526aaa87a1a18b6624aea",
        product_description: "",
        product_img: "cloud://release-prod.7265-release-prod/product/product-nux@3.png",
        product_name: "杏仁 120g",
        product_price: 0.1,
        product_sell_price: 0.1,
        product_stock: 100
      },
      {
        _id: "5cf526aaa87a1a18b6624aec",
        product_description: "",
        product_img: "cloud://release-prod.7265-release-prod/product/product-nux@4.png",
        product_name: "黑桃 180g",
        product_price: 0.1,
        product_sell_price: 0.1,
        product_stock: 100
      }

    ]
  }
```

`index.wxml`

```
 <!-- 分割线 -->
  <view class='dividing-line'></view>
  <!-- 3、最新上线 -->
  <view class='products-latest-container'>
    <title-bar-comp title='最新上线'></title-bar-comp>
    <block wx:for="{{products}}" wx:key="key">
      <product-cloumn-comp product="{{item}}"></product-cloumn-comp>
    </block>
  </view>
```

> title-bar-comp 的 `title="最新上线"` 中 title 对应组件 properties 中的属性名
>
> product-cloumn-comp 中的`product="{{item}}"`, product 对应 behaviors 中的属性名

`index.wxss`

```
/* 分割线 */
.dividing-line{
  width:750rpx;
  height:22rpx;
  background:rgba(249,249,249,1);
}
/* 最新上线 */
.products-latest-container{
  width: 750rpx;
}
```

运行效果如下：

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/039.png)

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)