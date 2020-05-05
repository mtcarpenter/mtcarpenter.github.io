---
layout: post
title: 【坚果商城实战系列学习】第 2-4 课：前端之主题实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 2-4 课：前端之主题实现
---

# 第 2-4 课：前端之主题实现

> 所在路径 `client/pages/theme/theme`

##  1 主题基本实现

`theme.js` data 数据如下，目前我们是没有调后台的数据，默认给出默认值方便页面的展示

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
  },
```

`theme.wxml`

```
<!--pages/theme/theme.wxml-->
<view class='container'>
  <block wx:for="{{products}}" wx:key="key">
    <product-row-comp product="{{item}}" bind:productDetails="productDetails"></product-row-comp>
  </block>
</view>


```

`theme.wxss`

```
/* pages/theme/theme.wxss */
.container{
  background-color: #f2f2f2;
  width: 750rpx;
  min-height: 100%;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-content: flex-start;
}
```

`theme.json`

```
{
  "usingComponents": {
    "product-row-comp": "/components/product-row/index"
  }
}
```

## 2 主题组件实现

商品信息我们任然封装成组件 `product-row/index`：

index.js 在前端之首页实现的章节我们把商品组件公共的数据封装成 `Behavior` ，在这里就可以直接复用

```
// components/theme/index.js
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

`index.wxml`

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

`index.wxss`

```
/* components/product-row/index.wxss */
@import "/common/css/iconfont.wxss";
.container{
  margin: 32rpx 10rpx 0 26rpx;
  background-color: #fff;
  border-radius: 20rpx;
}
.product-top {
  width:332rpx;
  height:290rpx;
}
.product-top image{
   width:332rpx;
   height:290rpx;
   border-radius: 20rpx; 
}
.product-title{
  padding-left: 10rpx;
  font-size:24rpx;
  font-weight:500;
  color:#242322;
  line-height:34rpx;
}
.market-price{
  margin-top:12rpx;
  padding-left: 10rpx;
}
.market-price text{
  font-size:24rpx;
  font-weight:400;
  text-decoration:line-through;
  color:#797979;
}
.product-container{
  width: 100%;
  display: inline-flex;
  justify-content: space-between;
  padding-left: 10rpx;
  margin-bottom: 10rpx;
}
.product-price text{
  width:80rpx;
  height:44rpx;
  font-weight:500;
  color:rgba(252,44,29,1);
  line-height:26rpx;
}
.product-price text:first-child{
  font-size:28rpx;
}
.product-price text:last-child{
  font-size:32rpx;
  margin-left: -1px;
}
.iconfont{
  font-size: 52rpx;
  color: #FC2C1D;
  padding-right: 20rpx;
}
```

完成效果如下：

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/042.png)



## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)