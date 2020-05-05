---
layout: post
title: 【坚果商城实战系列学习】第 2-3 课：前端之分类实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 2-3 课：前端之分类实现
---

# 第 2-3 课：前端之分类实现

> 所在路径 `client/pages/category/category`

## 1 分类基本实现

`category.js`  data数据如下，目前我们是没有调后台的数据，默认给出默认值方便页面的展示

```
 data: {
    menuCategories: [{
        category_name: '坚果炒货',
        category_type: 1
      },
      {
        category_name: '休闲零食',
        category_type: 2
      },
      {
        category_name: '饼干蛋糕',
        category_type: 3
      },
      {
        category_name: '蜜饯果干',
        category_type: 4
      },
      {
        category_name: '肉干肉脯',
        category_type: 5
      },
    ],
    menuSelect: 1,
    menuNmae: '',
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

`category.wxml`

```
<!--pages/category/category.wxml-->
<view class='container'>
  <!--  分类左边选择区域 -->
  <scroll-view class='left-container' scroll-y="true">
    <block wx:for="{{menuCategories}}" wx:key="key">
      <view class="categoryBar {{ menuSelect==item.category_type?'active':''}}" data-id='{{item.category_type}}' data-index='{{index}}' bind:tap="menu">
        <text >{{item.category_name}}</text>
      </view>
    </block>  
  </scroll-view>
  <!-- 分类右边选择区域 -->
   <scroll-view class='right-container' scroll-y="true">
    <!--主题宣传图  -->
    <view class='introduce-image'>
      <image src='../../images/temp/category.png'></image>
    </view>
    <view class='category-name'>
      <text>{{menuNmae}}</text>
    </view>
    <view class='product-container'>
      <block wx:for="{{products}}" wx:key="key">
        <category-comp  product="{{item}}" bind:productDetails="productDetails"  >      </category-comp>
      </block>   
    </view>
  </scroll-view>
</view>
```

`category.wxss`

```
/* pages/category/category.wxss */
.container{
  display: inline-flex;
  width: 100%;
  height: 100%;
  background-color: white;
  align-items:flex-start;
  flex-direction:row;
}
.left-container{
  width:146rpx;
  height: 100%;
  background:rgba(248,248,248,1);
}
.categoryBar{
  display: inline-flex;
  width:100%;
  height:100rpx;
  justify-content: center;
  align-items: center;
  color:#7A7A7A;
}
.categoryBar text{
  font-size:24rpx;
  font-weight:500;
  line-height:40rpx;
}
.active{
  color:#000000;
  background:#fff;
  border-left:6rpx solid #FF6200;
}
.active text{
  color: #FF6200;
}
/* 隐藏滚动条 */
::-webkit-scrollbar{
  width: 0;
  height: 0;
  color: transparent;
}
/* 右边 */
.right-containerr{
  width: 100rpx;
  min-height: 100%;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  background-color: #fff;
}
.introduce-image{
  margin: 20rpx 16rpx;
}
.introduce-image image{
  width:590rpx;
  height:270rpx;
  background:rgba(255,255,255,1);
  border-radius:10rpx;
}
.category-name{
  font-size:30rpx;
  font-weight:400;
  color:#686868;
  text-align: center;
}
oduct-container{
  display: flex;
  flex-wrap:wrap;
  margin: 10rpx 16rpx;
}
```

`category.json`

```
{
  "navigationBarTitleText": "分类",
  "usingComponents": {
    "category-comp": "/components/product-category/index"
  }
}
```

## 2 分类组件实现

商品信息我们任然封装成组件 `product-category/index`：

index.js 在上章我们把商品组件公共的数据封装成 `Behavior` ，在这里就可以直接复用

```
// components/category/index.js
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
<!--components/category/index.wxml-->
<view class='container' >
  <view class='product-img' >
    <image src='{{product.product_img}}'></image>
  </view>
  <view class='product-name'>
    <text>{{product.product_name}}</text>
  </view>
</view>
```

`index.wxss`

```
/* components/category/index.wxss */
.container{
  display: inline-flex;
  flex-flow: column;
  align-items: center;
  width:180rpx;
  margin-top: 20rpx;
  margin-left: 18rpx;
}
.product-img image{
  width:120rpx;
  height:120rpx;
}
.product-name{
  width: 150rpx;
  font-size:22rpx;
  font-weight:400;
  color:#838383;
  text-align: center;
}
```

完成效果如下：

![image](https://nux.oss-cn-beijing.aliyuncs.com/doc/040.png)

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)