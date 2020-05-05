---
layout: post
title: 【坚果商城实战系列学习】第 2-7 课：前端之订单实现
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 2-7 课：前端之订单实现
---

# 第 2-7 课：前端之订单实现

> 所在路径 `client/pages/order/order`

## 1 订单基本实现

`order.js`  data 数据如下，目前我们是没有调后台的数据，默认给出默认值方便页面的展示

```
data: {
    orderStatus: 0,
    account:0.1,
    address:[],
    products: [
    {
      order_id: '7217ba20a53811e9825141fe9cfd302d',
      product_count: 1,
      product_id: '5cf526aaa87a1a18b6624ae6',
      product_img: 'cloud://release-prod.7265-release-prod/product/product-nux@1.png',
      product_name: '花生 300g',
      product_price: 0.1,
    }]
   
  }
```

`order.wxml`

```
<!--pages/order/order.wxml-->
<wxs module="filter" src="../../common/wxs/filters.wxs"></wxs>
<view class='container'>
  <!-- 地址 -->
  <view wx:if='{{address.length!=0}}'>
    <view wx:if="{{status!=0}}">
      <address-comp addressInfo="{{address}}" slot="address"></address-comp>
    </view>
    <view wx:else>
      <img-btn-comp open-type="primary" bind:addressInfo="address">
        <address-comp addressInfo="{{address}}" slot="address"></address-comp>
      </img-btn-comp>
    </view>
  </view>

  <img-btn-comp open-type="primary" bind:addressInfo="addressInfo" wx:else>
    <view class='redirect-address' slot="address">
      <text class='redirect-text'>新增收货地址</text>
      <icon class='iconfont icon-webicon213'></icon>
    </view>
  </img-btn-comp>
  <view class='product-container'>
    <block wx:for="{{products}}" wx:key="">
      <view class="product-item">
        <view class="item-left">
          <image src="{{item.product_img}}"></image>
        </view>
        <view class="item-middle">
          <view>{{item.product_name}}</view>
          <view>￥{{item.product_price}}</view>
        </view>
        <view class="item-right">
          ×{{item.product_count}}
        </view>
      </view>
    </block>
  </view>
  <view class='order-container'>
    <view class='order-remarks'>
      <view class='details' wx:if="{{orderId!=null}}">
        <text>订单编号:</text>
        <text>{{orderId}}</text>
      </view>
      <view class='details'>
        <text>配送方式:</text>
        <text>快递 免邮</text>
      </view>
      <view class='details'>
        <text>优惠券:</text>
        <text>暂无可用</text>
      </view>
      <view class='details'>
        <text>配送费:</text>
        <text>¥0</text>
      </view>
    </view>
  </view>
  <view class='confirm-container'>
    <view class='total'>
      <text>合计:</text>
      <text>¥{{account}}</text>
    </view>
    <view class='confirm'>
      <text>立即购买</text>
    </view>

  </view>

</view>
```

`order.wxss`

```
/* pages/order/order.wxss */
.container{
  width: 750rpx;
  min-height: 100%;
  background-color: #FFFFFF;
}

.redirect-address{
  width: 696rpx;
  height: 60rpx;
  display: inline-flex;
  justify-content: space-between;
  align-items: center;
  margin: 28rpx 0 0 28rpx;
  border-bottom: 4rpx solid rgba(242,242,242,1);
}
.redirect-address icon{
  font-size: 43rpx;
  color: #464646;
}
.redirect-text{
  height:50rpx;
  font-size:36rpx;
  color:rgba(41,41,41,1);
  line-height:50rpx;
}
.dealer{
  display: inline-flex;
  height: 60rpx;
  margin:  40rpx 0 0 28rpx;
  align-items: center;
}
.dealer image{
  width: 56rpx;
  height: 56rpx;
  border-radius: 50%;
}
.dealer text{
  font-size:30rpx;
  color:#000000;
  line-height:56rpx;
  margin-left: 12rpx;
}
.sku-container{
  width: 696rpx;
  margin: 32rpx 28rpx 0 28rpx;
  border-bottom:8rpx solid rgba(242,242,242,1);
  padding-bottom: 46rpx;
}
.sku-comp{
  width: 696rpx; 
  margin-top: 10rpx;
  
}
.order-status,.order-remarks{
  width: 696rpx;
  margin-left: 28rpx;
  border-bottom:8rpx solid rgba(242,242,242,1);
  padding-bottom: 40rpx;
}
.details{
  width: 696rpx;
  display: inline-flex;
  justify-content: space-between;
  margin: 30rpx 0 0 0;
}
.details text{
  font-size:30rpx;
  color:rgba(41,41,41,1);
  line-height:42rpx;
}
.order-remarks{
  border: 0;
  padding-bottom: 80rpx;
  
}
.remarks-input{
  display: inline-flex;
} 
.remarks-input label{
  color:rgba(41,41,41,1);
  margin-right: 30rpx;
}
.remarks-input input{
  margin-top: -3rpx;
}
.remarks-placeholder{
  font-size:30rpx;
  color:#B0B0B0;
  line-height:42rpx;
  
}
.total text:first-child{
  font-size:32rpx;
  color:#242424;
  line-height:44rpx;
  margin-right: 5rpx;
}
.total text:last-child{
  font-size:32rpx;
  color:#FC2C1D;
  line-height:44rpx;
}
.confirm-container{
  width: 750rpx;
  height: 80rpx;
  background-color: #FFFFFF;
  display: inline-flex;
  align-items: center;
  justify-content: flex-end;
  position: fixed;
  bottom: 0rpx;
 
}
.total{
  margin: 0 18rpx 0 0;
}
.confirm{
  width:168rpx;
  height:58rpx;
  background:#FD5E53;
  border-radius:29rpx;
  text-align: center;
  margin: 0 18rpx 0 0;
}
.confirm text{
  font-size:28rpx;
  color:#FFFFFF;
  line-height:58rpx;
}

.product-container{
  width: 750rpx;
}

.product-item{
    display: flex;
    justify-content: space-around;
    height:180rpx;
    color: #6D6D6D;
    border-bottom: 1rpx solid #E9E9E9;
    padding:20rpx;
}
.product-item .item-left{
    flex-basis: 180rpx;
    height: 100%;
    background-color: #F5F6F5;
    border-radius: 4rpx;
}
.product-item .item-left image{
    height: 100%;
    width: 100%;
}
.product-item .item-middle{
    flex: 1;
    margin-left: 20rpx;
}
.product-item .item-middle view{
    margin: 15rpx 0;
}
.product-item .item-right{
    flex-basis: 80rpx;
    text-align: center;
}

.order-accounts{
    background-color: #fff;
}
```

`order/order.json`

```
{
  "navigationBarTitleText":"确认订单",
  "usingComponents":{
    "address-comp":"/components/address/index",
    "img-btn-comp": "/components/image-button/index"
  }

}
```

## 2 订单组件实现

### 2.1 授权组件 `image-button `

```
// components/image-button/index.js
Component({
  /**
   * 组件的属性列表
   */
  options: {
    multipleSlots: true // 在组件定义时的选项中启用多slot支持
  },
  properties: {
    openType: {
      type: String
    },
    imageSrc: {
      type: String
    },
    bindgetuserinfo: {
      type: String
    }
  },
  /**
   * 组件的初始数据
   */
  data: {
    addressInfo:""
  },
  /**
   * 组件的方法列表
   */
  methods: {
    onGetUserInfo(event) {   
      if (this.data.openType == "getUserInfo"){             
       // if (event.detail.errMsg) {
        this.triggerEvent('getuserinfo', event.detail, {})
       // }
      }           
    }, 
    address(event){    
      if (this.data.openType == "primary") {
        this.addressInfo(event) 
      }      
    },
    addressInfo(event){      
      wx.chooseAddress({
        success: (res) => {
          this.setData({
            addressInfo: res
          })
        },
        fail:(err) =>{}, 
        complete:(e)=> {
          if (e.errMsg == "chooseAddress:ok") {
            this.triggerEvent('addressInfo', {
              addressInfo: this.data.addressInfo,
              status: "ok"
            }, {})
          } else {
            this.triggerEvent('addressInfo', {
              status: "error"
            }, {})
          }

        }
      })
    }
  }
})

```

`index.wxml`

```
<!--components/image-button/index.wxml-->
<button bind:getuserinfo="onGetUserInfo" bind:tap='address' open-type='{{openType}}'  plain='{{true}}' class="container" lang="zh_CN">
  <slot name="share"></slot>
  <slot name="address"></slot>
  <slot name="info"></slot>
</button>

```

`index.wxss`

```
/* components/image-button/index.wxss */
.container{
  padding: 0 !important;
  border:none !important;
}
```

> image-button 在微信小程序目前授权信息、地址、录音等只能通过 button 中固定的参数值获取（  [`官方介绍`](https://developers.weixin.qq.com/miniprogram/dev/component/button.html )）,上面封装的组件主要是为了实现微信开放能力，授权获取信息。

### 2.3 组件 `address`

`index.js`

```
// components/address/address.js
Component({
  /**
   * 组件的属性列表
   */
  properties: {
    addressInfo:Object
  },

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
<!--components/address/address.wxml-->
<view class='container'>
  <view class='address-title'>
    <text class='name'>收货人：{{addressInfo.userName}}</text>
    <text class='phone'>{{addressInfo.phone}}</text>
  </view>
  <view class='address-content'>
    <text class='address'>收货地址：{{addressInfo.detailAdress}}</text>
    <icon class='iconfont icon-webicon213'></icon>
  </view>

</view>
```

`index.wxss`

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

![1563015205084](https://nux.oss-cn-beijing.aliyuncs.com/doc/044.png)



## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)