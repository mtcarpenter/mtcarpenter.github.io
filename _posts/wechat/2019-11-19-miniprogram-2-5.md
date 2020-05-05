---
layout: post
title: 【坚果商城实战系列学习】第 2-5 课：前端之购物车实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 2-5 课：前端之购物车实现
---

# 第 2-5 课：前端之购物车实现

> 所在路径 `client/pages/cart/cart`

## 1 购物车基本实现

`cart.js` data 数据如下，目前我们是没有调后台的数据，默认给出默认值方便页面的展示

```
data: {
    cartData: [{
        _id: "5cf526aaa87a1a18b6624ae6",
        product_description: "",
        product_img: "cloud://release-prod.7265-release-prod/product/product-nux@1.png",
        product_name: "花生 300g",
        product_price: 0.1,
        product_sell_price: 0.1,
        product_stock: 100,
        counts: 2,
        selectStatus: true
      },
      {
        _id: "5cf526aaa87a1a18b6624ae8",
        product_description: "",
        product_img: "cloud://release-prod.7265-release-prod/product/product-nux@2.png",
        product_name: "夏威夷果 120g",
        product_price: 0.1,
        product_sell_price: 0.1,
        product_stock: 100,
        counts: 2,
        selectStatus: true
      }
    ],
    selectedCounts: 0, //总的商品数
    selectedTypeCounts: 0, //总的商品类型数 
    account: 0.7
  },
```

`cart.wxml`

```
<!--pages/cart/cart.wxml-->
<view class='container'>
  <view class='cart-container' wx:for="{{cartData}}" wx:key="index">
    <!-- 商品左边 -->
    <view class='cart-left-container' bindtap="toggleSelect" data-id="{{item._id}}" data-status="{{item.selectStatus}}" >
        <view class="cart-select {{item.selectStatus?'selectActive':''}}"   >
            <icon  class='iconfont iconiconfontcheck'></icon>
        </view>   
    </view>
     <!-- 商品图片 -->
    <view class='cart-middle-container'>
        <image src="{{item.product_img}}"></image>
    </view>
    <view class='cart-right-container'>
        <view class='product-basic'> 
          <view class='product-title'>
            <text>{{item.product_name}}</text>
          </view>
          <view class='product-price'>
            <text>￥{{item.product_sell_price}}</text>
          </view>
        </view>  
        <view class='edit-contianer'>
           <view class='edit-num'>
              <icon class="iconfont {{item.counts==1?'disabled':''}} iconjian  "  bindtap="{{item.counts==1?'':'changeCounts'}}" data-id="{{item._id}}" data-type="cut"></icon>
              <view class='num'> <text>{{item.counts}}</text></view>
              <icon class='iconfont allow iconjia1' bindtap="changeCounts" data-id="{{item._id}}" data-type="add"></icon>
           </view>
           <view class='delete'>
            <icon class='iconfont iconshanchu'></icon>
           </view>
        </view>
    </view>
  </view>

 <!-- 全选  wx:if="{{checkall?'selectActive':''}}" -->
    <view class='bottom-container'> 
      <view class='all-select'>
        <view class="all-select-icon {{selectedTypeCounts==cartData.length?'selectActive':''}}" data-dealerIndex='{{dealerIndex}}' bind:tap='checkall'   data-status="{{selectedTypeCounts==cartData.length?'true':'false'}}" >
          <icon  class='iconfont iconiconfontcheck'></icon>
        </view>
        <view class='all-select-text'>
            <text >全选</text>
        </view>
      </view>
      <view class='total-container' >
        <view class='total-price'>
          <text>合计:</text>        
          <text class='price-symbol' >￥</text>
          <text class='price' >{{account}}</text>
        </view>
        <view class='accounts' bind:tap="confirm">
          <text>结算</text>
        </view>      
      </view>
    </view>
</view>
```

`cart.wxss`

```
/* pages/cart/cart.wxss */
@import '/common/css/iconfont.wxss';
.container{
  display: flex;
  flex-direction: column;
  align-items:flex-start;
  background-color: #fff;
  min-height: 100%;

}

.cart-container{
  display: inline-flex;
  border-bottom: 1rpx solid #bbb;
  width: 100%;

}
.cart-left-container{
  width: 100rpx;
  height: 180rpx;
  display: inline-flex;
  justify-content: center;
  align-items: center;
}
.cart-select{
  width:37rpx;
  height:37rpx;
  position: relative;
  border: 1rpx solid #bbb;
  border-radius: 50%;
}
.cart-select icon{
  position: absolute;
  top: -14rpx;
  font-size: 38rpx;
  color: #fff;
}
.selectActive{
  background:#FD5E53;
  border: 1rpx solid #FD5E53;
}

.cart-middle-container{
  width: 180rpx;
  height: 180rpx;
}
.cart-middle-container image{
  width: 100%;
  height: 100%;
}
.cart-right-container{
  display: inline-flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 20rpx 0;
}
.product-basic{
  display: inline-flex;
  justify-content: space-between;
  width: 420rpx;
  color: #454553;
  font-size: 28rpx;
}
.edit-contianer{
  width: 420rpx;
  display: inline-flex;
  justify-content: space-between;
  align-items: flex-end;
}
.edit-num{
  display: inline-flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  align-content: center;
}
.edit-num .iconfont{
  color: #454553;
  font-size: 50rpx;
  padding: 0 10rpx;
}
.delete .iconfont{
  color: #D5D5DB;
  font-size: 40rpx;
}
.edit-contianer .num{
  font-size: 36rpx;
  padding: 0 10rpx;
  color: #454553;
}
.edit-num .disabled{
    color: #D5D5DB;
}
/* 结算 */
.bottom-container{
  position: fixed;
  display: inline-flex;
  justify-content: space-between;
  background-color: #FFFFFF;
  width: 750rpx;
  height: 100rpx;
  bottom: 10rpx;
  border-top: 1rpx solid #bbb;
}
.all-select{
  display: inline-flex;
  align-content: center;
}
.all-select-text{
  padding-top: 28rpx;
  margin-left: 20rpx;
  font-size:28rpx;
  font-weight:400;
  color:rgba(36,36,36,1);
}
.all-select-icon{
  width:27rpx;
  height:27rpx;
  border-radius: 50%;
  margin: 35rpx 0 0 34rpx;
  position: relative;
  border: 1rpx solid #bbb;
}

.all-select-icon icon{
  position: absolute;
  top: -20rpx;
  left: -1rpx;
  color: #FFFFFF;
}
.total-container{
 display: inline-flex;
 align-items: center;
}
.total-price{
  margin-right: 12rpx;
}
.total-price >text{
  font-size:28rpx;
  color:rgba(252,44,29,1);
}
.total-price >text:first-child{
  color: #242424;
}
.price-symbol{
  font-weight:500;
 
  line-height:26rpx;
}
.price{
  font-weight:600;
  color:rgba(252,44,29,1);
  line-height:40rpx;
  margin-left: -5rpx;
}
.total-remarks{
  color: #848484;
  font-size:20rpx;
  text-align: right;
}
.accounts{
  width:140rpx;
  height:58rpx;
  background:rgba(253,94,83,1);
  border-radius:29rpx;
  text-align: center;
  margin-right: 16rpx;
}
.accounts text{
  font-size:28rpx;
  color:rgba(255,255,255,1);
  line-height:58rpx;
}
.selectActive{
  background:#FD5E53;
  border: 1rpx solid #FD5E53;
}

.delete-container{
  margin: 24rpx 25rpx 0 0 ;
  width:140rpx;
  height:58rpx;
  border-radius:29rpx;
  border:2rpx solid #FD5E53;
  text-align: center;
}
.delete-container text{
  width:56rpx;
  height:40rpx;
  font-size:28rpx;
  font-weight:400;
  color:#FD5E53;
  line-height:58rpx;
}
.null-container{
  text-align: center;
  margin-top: 45%;
}

/* 价格增加 */
.num-container{
  display: inline-flex;
  justify-content: center;
  align-items: center;
}
.num-container view{
  width:37rpx;
  height:37rpx;
}

.count{
  text-align: center;
  border:2rpx solid #D9D9D9; 
  border-radius: 50%;
}
.num{
  margin: 0 10rpx;
  text-align: center;
}
.num text{
  width:14rpx;
  height:44rpx;
  font-size:32rpx;
  font-weight:400;
  color:#000000;
  line-height:44rpx;
}
.count text{
  font-size: 18rpx;
}
.num-container .iconfont{
  font-size: 38rpx;
 
}
.noarrow{
  color: #D9D9D9;
}
.allow{
  color: #FD5E53;
}
```

运行效果如下：

![1562420680094](https://nux.oss-cn-beijing.aliyuncs.com/doc/041.png)

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)