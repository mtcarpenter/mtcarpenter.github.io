---
layout: post
title: 【坚果商城实战系列学习】第 2-6 课：前端之商品详情实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 2-6 课：前端之商品详情实现
---

# 第 2-6 课：前端之商品详情实现 

> 所在路径 `client/pages/product/product`

##  1 商品详情基本实现	

`product.js` data 数据如下，目前我们是没有调后台的数据，默认给出默认值方便页面的展示

```
 data: {
    currentTab: 0, // tab选项卡
    product: {
      _id: '5cf526aaa87a1a18b6624ae6',
      product_description: '',
      product_img: 'cloud://release-prod.7265-release-prod/product/product-nux@1.png',
      product_name: '花生 300g',
      product_price: 0.1,
      product_sell_price: 0.1,
      product_stock: 100
    }
  }
```

`product.wxml`

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
      <view class='icon-container home'  >
        <icon class='iconfont iconziyuan'></icon>
        <text>首页</text>
      </view>
      <view class="icon-container customer" >
        <icon class="iconfont  iconshoucang1 "></icon>
        <text>关注</text>
      </view>
      <view class='icon-container cart' >
        <icon class='iconfont iconicon01'></icon>
        <text>购物车</text>
      </view>
      <view class='go-container'>
        <view class='joinCart'><text>加入购物车</text></view>
        <view class='immediately'><text>立即购买</text></view>
      </view>
  </view>
</view>
```

`product.wxss`

```
/* pages/product/product.wxss */
@import '/common/css/iconfont.wxss';
.container{
  width: 750rpx;
  height: 100%;
  background-color: #FFFFFF;
}
.swiper-container{
  width:750rpx;
  display: flex;
  justify-content: center;
}
.slide-image{
  width:700rpx;
  height:460rpx;
}

.title-container{
  width: 750rpx;
  display: inline-flex;
  justify-content: space-between;
  align-items: center;
}
.title{
  width:602rpx;
  max-height:80rpx;
  font-size:28rpx;
  color:#141414;
  line-height:40rpx;
  margin-left: 28rpx;
}

.price-container{
  width: 750rpx;
  font-size:40rpx;
  color:#FC2C1D;
  line-height:56rpx;
  margin: 10rpx 0 0 28rpx;
}
.symbol{
  font-size: 34rpx;
}
.price{
  margin-left: -5rpx;
}
.market-price{
  font-size:28rpx;
  font-weight:400;
  text-decoration:line-through;
  color:#797979;
}

.parameter-container{
  width: 700rpx;
  height: 80rpx;
  display: inline-flex;
  justify-content: space-around;
  align-items: center;
}
.parameter-container text{
  font-size:24rpx;
  color:#B7B7B7;
  line-height:34rpx;
}
.parameter-container text:nth-child(2){
   color:#FC2C1D;
}

.swiper-tab{
  width: 600rpx;
  height: 50rpx;
  display: inline-flex;
  justify-content: space-around;
  margin-left: 75rpx;
}
.swiper-tab-item{
  width:152rpx;
  height:50rpx;
  text-align: center;
  margin-right: 12rpx;
  color: #000000;
  font-size:28rpx;
  line-height:50rpx;
}
.active{
  color:#FC2C1D;
  border-bottom:3rpx solid #FC2C1D;
}

.tab-container swiper {
  height: auto；
}
.swiper-tab-show{
  width: 750rpx;
  text-align: center;
  margin-top: 30rpx;
}
.swiper-tab-show text{
  font-size: 28rpx;
  color: #797979;
}
.bottom-container{
  width:750rpx;
  height:88rpx;
  position: fixed;
  bottom: 0;
  background:rgba(255,255,255,1);
  box-shadow:0 10rpx 12rpx 12rpx rgba(138,80,80,0.13);
  display: inline-flex;
  justify-content: space-around;
  align-items: center;
}
.icon-container{
  display: inline-flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
}
.cart icon{
  color: #6A6A6A;
}
.customer icon{
  color: #6A6A6A;
}
.home icon{
  font-size: 52rpx;
  color: #FC2C1D;
}
.customer icon{
  font-size: 46rpx;
}
.focusActive icon{
  color:rgba(255,140,79,1)
}
.cart icon{
  font-size: 46rpx;
}
.icon-container text{
  font-size:18rpx;
  color:#6A6A6A;
  line-height:26rpx;
}
.go-container{
 
  display: inline-flex;
 
}
.go-container>view{
  width: 206rpx;
  height:68rpx;
}
.joinCart{
  background:rgba(255,140,79,1);
  border-top-left-radius: 34rpx;
  border-bottom-left-radius: 34rpx;
  text-align: center;
}
.immediately{
  background:rgba(252,44,29,1);
  border-top-right-radius: 34rpx;
  border-bottom-right-radius: 34rpx;
  text-align: center;
}
.joinCart text,.immediately text{
  font-size:28rpx;
  color:#FFFFFF;
  line-height:68rpx;
}
```

完成效果如下：

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/043.png)





## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)